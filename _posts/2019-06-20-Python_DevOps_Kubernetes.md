---
layout: post
title: "Python DevOps-Kubernetes部署详解"
date: 2019-06-20 
description: "K8s基础、Pod、Service、Deployment、ConfigMap、StatefulSet、Ingress、HPA"
tag: Python

---

## Kubernetes部署的重要性

Kubernetes是容器编排的事实标准，能够自动化应用的部署、扩展和管理。Python应用通过Kubernetes可以实现高可用、自动扩缩容、滚动更新等企业级特性。本文将从基础的Pod和Service到高级的StatefulSet和HPA，全面介绍Python应用在Kubernetes上的部署最佳实践。

## 基础资源

### 1. Deployment配置

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
  labels:
    app: python-app
    version: v1.0.0
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
        version: v1.0.0
    spec:
      containers:
      - name: python-app
        image: myregistry/python-app:1.0.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8000
          name: http
          protocol: TCP
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: redis-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
          readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

### 2. Service配置

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: python-app-service
  labels:
    app: python-app
spec:
  type: LoadBalancer
  selector:
    app: python-app
  ports:
  - name: http
    port: 80
    targetPort: 8000
    protocol: TCP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
---
# ClusterIP Service（内部访问）
apiVersion: v1
kind: Service
metadata:
  name: python-app-internal
spec:
  type: ClusterIP
  selector:
    app: python-app
  ports:
  - port: 8000
    targetPort: 8000
---
# NodePort Service（节点端口访问）
apiVersion: v1
kind: Service
metadata:
  name: python-app-nodeport
spec:
  type: NodePort
  selector:
    app: python-app
  ports:
  - port: 8000
    targetPort: 8000
    nodePort: 30080
```

### 3. ConfigMap和Secret

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    debug=false
    log_level=INFO
    max_workers=10
  redis-url: "redis://redis-service:6379"
  cache-ttl: "3600"
---
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  database-url: "postgresql://user:password@db-service:5432/mydb"
  api-key: "your-secret-api-key"
  jwt-secret: "your-jwt-secret-key"
```

## 高级配置

### 1. StatefulSet（有状态应用）

```yaml
# k8s/statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: python-worker
spec:
  serviceName: "python-worker"
  replicas: 3
  selector:
    matchLabels:
      app: python-worker
  template:
    metadata:
      labels:
        app: python-worker
    spec:
      containers:
      - name: worker
        image: myregistry/python-worker:1.0.0
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast-ssd"
      resources:
        requests:
          storage: 10Gi
```

### 2. Ingress配置

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: python-app-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: python-app-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: python-app-service
            port:
              number: 80
      - path: /api/v1
        pathType: Prefix
        backend:
          service:
            name: python-app-service
            port:
              number: 80
```

### 3. Horizontal Pod Autoscaler（HPA）

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: python-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: python-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 2
        periodSeconds: 15
      selectPolicy: Max
```

## Python应用配置

### 1. 健康检查端点

```python
# app/health.py
from flask import Flask, jsonify
import psutil
import time

app = Flask(__name__)

@app.route('/health')
def health_check():
    """健康检查端点"""
    return jsonify({
        'status': 'healthy',
        'timestamp': time.time()
    }), 200

@app.route('/ready')
def readiness_check():
    """就绪检查端点"""
    # 检查数据库连接
    try:
        # db.ping()
        db_ready = True
    except:
        db_ready = False
    
    # 检查Redis连接
    try:
        # redis.ping()
        redis_ready = True
    except:
        redis_ready = False
    
    if db_ready and redis_ready:
        return jsonify({
            'status': 'ready',
            'database': 'connected',
            'redis': 'connected'
        }), 200
    else:
        return jsonify({
            'status': 'not ready',
            'database': 'connected' if db_ready else 'disconnected',
            'redis': 'connected' if redis_ready else 'disconnected'
        }), 503

@app.route('/metrics')
def metrics():
    """Prometheus指标端点"""
    cpu_percent = psutil.cpu_percent(interval=1)
    memory = psutil.virtual_memory()
    
    metrics_text = f"""# HELP cpu_usage CPU使用率
# TYPE cpu_usage gauge
cpu_usage {cpu_percent}

# HELP memory_usage 内存使用率
# TYPE memory_usage gauge
memory_usage {memory.percent}
"""
    return metrics_text, 200, {'Content-Type': 'text/plain'}
```

### 2. Kubernetes客户端

```python
# app/k8s_client.py
from kubernetes import client, config
from kubernetes.client.rest import ApiException

class KubernetesManager:
    """Kubernetes管理器"""
    
    def __init__(self):
        # 在集群内运行时自动加载配置
        try:
            config.load_incluster_config()
        except:
            # 本地开发时使用kubeconfig
            config.load_kube_config()
        
        self.v1 = client.CoreV1Api()
        self.apps_v1 = client.AppsV1Api()
    
    def get_pods(self, namespace='default', label_selector=None):
        """获取Pod列表"""
        try:
            pods = self.v1.list_namespaced_pod(
                namespace=namespace,
                label_selector=label_selector
            )
            return [pod.metadata.name for pod in pods.items]
        except ApiException as e:
            print(f"获取Pod列表失败: {e}")
            return []
    
    def scale_deployment(self, name, replicas, namespace='default'):
        """扩缩容Deployment"""
        try:
            deployment = self.apps_v1.read_namespaced_deployment(
                name=name,
                namespace=namespace
            )
            deployment.spec.replicas = replicas
            
            self.apps_v1.patch_namespaced_deployment(
                name=name,
                namespace=namespace,
                body=deployment
            )
            return True
        except ApiException as e:
            print(f"扩缩容失败: {e}")
            return False
    
    def get_service_endpoints(self, service_name, namespace='default'):
        """获取Service端点"""
        try:
            endpoints = self.v1.read_namespaced_endpoints(
                name=service_name,
                namespace=namespace
            )
            return [
                f"{addr.ip}:{port.port}"
                for subset in endpoints.subsets
                for addr in subset.addresses
                for port in subset.ports
            ]
        except ApiException as e:
            print(f"获取端点失败: {e}")
            return []

# 使用示例
if __name__ == "__main__":
    k8s = KubernetesManager()
    
    # 获取Pod列表
    pods = k8s.get_pods(label_selector='app=python-app')
    print(f"Python应用Pod: {pods}")
    
    # 扩缩容
    k8s.scale_deployment('python-app', replicas=5)
    
    # 获取Service端点
    endpoints = k8s.get_service_endpoints('python-app-service')
    print(f"Service端点: {endpoints}")
```

## 部署脚本

### 1. 部署脚本

```bash
#!/bin/bash
# deploy.sh

set -e

NAMESPACE="production"
APP_NAME="python-app"
IMAGE_TAG="${1:-latest}"

echo "开始部署 $APP_NAME:$IMAGE_TAG 到 $NAMESPACE 命名空间..."

# 创建命名空间（如果不存在）
kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

# 应用ConfigMap和Secret
kubectl apply -f k8s/configmap.yaml -n $NAMESPACE
kubectl apply -f k8s/secret.yaml -n $NAMESPACE

# 更新Deployment镜像
kubectl set image deployment/$APP_NAME \
    python-app=myregistry/$APP_NAME:$IMAGE_TAG \
    -n $NAMESPACE

# 等待部署完成
kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=5m

# 验证部署
kubectl get pods -l app=$APP_NAME -n $NAMESPACE

echo "部署完成！"
```

### 2. 回滚脚本

```bash
#!/bin/bash
# rollback.sh

set -e

NAMESPACE="production"
APP_NAME="python-app"

echo "回滚 $APP_NAME..."

# 查看部署历史
kubectl rollout history deployment/$APP_NAME -n $NAMESPACE

# 回滚到上一个版本
kubectl rollout undo deployment/$APP_NAME -n $NAMESPACE

# 等待回滚完成
kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=5m

echo "回滚完成！"
```

## 总结

Kubernetes部署的关键要点：

1. **Deployment**：无状态应用的部署和滚动更新
2. **Service**：服务发现和负载均衡
3. **ConfigMap和Secret**：配置和密钥管理
4. **StatefulSet**：有状态应用的部署
5. **Ingress**：外部访问和路由管理
6. **HPA**：自动扩缩容
7. **健康检查**：Liveness和Readiness探针

掌握这些Kubernetes技能，可以实现Python应用的高可用部署、自动扩缩容和滚动更新，为生产环境提供强大的容器编排支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-Kubernetes部署详解](http://zhouzhiyang.cn/2019/06/Python_DevOps_Kubernetes/)