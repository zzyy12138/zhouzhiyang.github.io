---
layout: post
title: "Elasticsearch安全认证与授权"
date: 2025-05-02 
description: "安全概述、用户认证、角色权限、API密钥、TLS/SSL配置、安全最佳实践"
tag: ELK

---

## 安全概述

Elasticsearch 8.x版本默认启用了安全功能，包括认证、授权、加密通信等。在生产环境中，正确配置安全功能对于保护数据安全至关重要。

### 安全功能

```python
# Elasticsearch安全功能
security_features = {
    "认证": ["内置用户", "LDAP", "Active Directory", "PKI", "SAML", "OIDC"],
    "授权": ["基于角色的访问控制", "字段级安全", "文档级安全"],
    "加密": ["TLS/SSL", "传输加密", "静态加密"],
    "审计": ["操作审计", "访问日志"]
}
```

## 用户认证

### 1. 内置用户

```bash
# 创建内置用户
bin/elasticsearch-users useradd myuser -p mypassword -r superuser

# 修改用户密码
bin/elasticsearch-users passwd myuser

# 删除用户
bin/elasticsearch-users userdel myuser

# 列出所有用户
bin/elasticsearch-users list
```

### 2. 使用API管理用户

```bash
# 创建用户
curl -X POST "localhost:9200/_security/user/myuser?pretty" \
  -u elastic:password \
  -H 'Content-Type: application/json' \
  -d'
{
  "password": "mypassword",
  "roles": ["superuser"],
  "full_name": "My User",
  "email": "myuser@example.com"
}
'

# 更新用户
curl -X PUT "localhost:9200/_security/user/myuser?pretty" \
  -u elastic:password \
  -H 'Content-Type: application/json' \
  -d'
{
  "password": "newpassword",
  "roles": ["kibana_admin", "monitoring_user"]
}
'

# 删除用户
curl -X DELETE "localhost:9200/_security/user/myuser?pretty" \
  -u elastic:password
```

### 3. 内置用户角色

```yaml
# 内置用户角色
内置角色:
  superuser: "超级用户，拥有所有权限"
  kibana_admin: "Kibana管理员"
  kibana_system: "Kibana系统用户"
  monitoring_user: "监控用户"
  logstash_system: "Logstash系统用户"
  beats_system: "Beats系统用户"
  apm_system: "APM系统用户"
```

## 角色和权限

### 1. 创建角色

```bash
# 创建自定义角色
curl -X POST "localhost:9200/_security/role/my_role?pretty" \
  -u elastic:password \
  -H 'Content-Type: application/json' \
  -d'
{
  "cluster": ["monitor", "read_pipeline"],
  "indices": [
    {
      "names": ["my_index*"],
      "privileges": ["read", "write", "index"],
      "field_security": {
        "grant": ["*"],
        "except": ["password", "secret"]
      },
      "query": {
        "match": {
          "status": "active"
        }
      }
    }
  ],
  "applications": [
    {
      "application": "kibana-.kibana",
      "privileges": ["read"],
      "resources": ["*"]
    }
  ]
}
'
```

### 2. 角色权限类型

```yaml
# 角色权限类型
权限类型:
  集群权限:
    - monitor: "监控集群"
    - manage: "管理集群"
    - all: "所有集群权限"
  
  索引权限:
    - read: "读取索引"
    - write: "写入索引"
    - index: "索引文档"
    - delete: "删除文档"
    - manage: "管理索引"
    - all: "所有索引权限"
  
  应用权限:
    - 应用程序特定权限
    - Kibana权限
```

### 3. 字段级安全

```json
// 字段级安全配置
{
  "indices": [
    {
      "names": ["users"],
      "privileges": ["read"],
      "field_security": {
        "grant": ["name", "email", "created_at"],
        "except": ["password", "credit_card"]
      }
    }
  ]
}
```

### 4. 文档级安全

```json
// 文档级安全配置
{
  "indices": [
    {
      "names": ["orders"],
      "privileges": ["read"],
      "query": {
        "term": {
          "user_id": "${user.name}"
        }
      }
    }
  ]
}
```

## API密钥

### 1. 创建API密钥

```bash
# 创建API密钥
curl -X POST "localhost:9200/_security/api_key?pretty" \
  -u elastic:password \
  -H 'Content-Type: application/json' \
  -d'
{
  "name": "my-api-key",
  "expiration": "1d",
  "role_descriptors": {
    "my_role": {
      "cluster": ["monitor"],
      "indices": [
        {
          "names": ["my_index*"],
          "privileges": ["read"]
        }
      ]
    }
  }
}
'
```

### 2. 使用API密钥

```bash
# 使用API密钥访问
curl -X GET "localhost:9200/my_index/_search?pretty" \
  -H "Authorization: ApiKey VnVhQ2ZHY0JDZGJrU..."
```

## TLS/SSL配置

### 1. 生成证书

```bash
# 使用elasticsearch-certutil生成证书
bin/elasticsearch-certutil ca
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
bin/elasticsearch-certutil http
```

### 2. 配置TLS

```yaml
# config/elasticsearch.yml
# HTTP层TLS
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.key: /path/to/http.key
xpack.security.http.ssl.certificate: /path/to/http.crt
xpack.security.http.ssl.certificate_authorities: ["/path/to/ca.crt"]

# 传输层TLS
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.key: /path/to/transport.key
xpack.security.transport.ssl.certificate: /path/to/transport.crt
xpack.security.transport.ssl.certificate_authorities: ["/path/to/ca.crt"]
xpack.security.transport.ssl.verification_mode: certificate
```

## 外部认证

### 1. LDAP认证

```yaml
# config/elasticsearch.yml
xpack.security.authc.realms.ldap.ldap1:
  order: 1
  url: "ldaps://ldap.example.com:636"
  bind_dn: "cn=admin,dc=example,dc=com"
  bind_password: "password"
  user_search.base_dn: "ou=users,dc=example,dc=com"
  user_search.filter: "(uid={0})"
  group_search.base_dn: "ou=groups,dc=example,dc=com"
  ssl.verification_mode: certificate
```

### 2. Active Directory认证

```yaml
# config/elasticsearch.yml
xpack.security.authc.realms.active_directory.ad1:
  order: 2
  domain_name: "example.com"
  url: "ldaps://ad.example.com:636"
  bind_dn: "admin@example.com"
  bind_password: "password"
  ssl.verification_mode: certificate
```

## 安全最佳实践

### 1. 密码策略

```yaml
# 密码策略配置
xpack.security.authc.password_hashing.algorithm: bcrypt
xpack.security.authc.password_hashing.bcrypt.cost: 10
```

### 2. 审计日志

```yaml
# 启用审计日志
xpack.security.audit.enabled: true
xpack.security.audit.logfile.events.include: ["access_denied", "authentication_failed"]
xpack.security.audit.logfile.events.exclude: ["access_granted"]
```

### 3. 网络安全

```yaml
# 网络安全配置
network.host: _site_  # 绑定到特定网络接口
http.port: 9200
transport.port: 9300
```

### 4. 最小权限原则

```yaml
# 最小权限原则
权限配置:
  - 为每个用户分配最小必要权限
  - 使用角色而不是直接分配权限
  - 定期审查和更新权限
  - 使用字段级和文档级安全
```

## 实际应用案例

### 1. 多租户安全配置

```yaml
# 多租户安全配置
场景: 多个租户共享Elasticsearch集群

配置:
  索引命名: "tenant1-logs-*", "tenant2-logs-*"
  角色配置:
    - tenant1_user: 只能访问tenant1-logs-*
    - tenant2_user: 只能访问tenant2-logs-*
  
  文档级安全:
    - 使用查询限制访问范围
```

### 2. 生产环境安全配置

```yaml
# 生产环境安全配置检查清单
安全配置:
  - [ ] 启用TLS/SSL
  - [ ] 配置用户认证
  - [ ] 设置角色和权限
  - [ ] 启用审计日志
  - [ ] 配置密码策略
  - [ ] 限制网络访问
  - [ ] 定期更新证书
  - [ ] 监控安全事件
```

## 总结

Elasticsearch安全认证与授权的关键要点：

1. **安全概述**：安全功能、认证方式、授权机制
2. **用户认证**：内置用户、API管理、内置角色
3. **角色和权限**：创建角色、权限类型、字段级安全、文档级安全
4. **API密钥**：创建和使用API密钥
5. **TLS/SSL配置**：证书生成、TLS配置
6. **外部认证**：LDAP、Active Directory认证
7. **安全最佳实践**：密码策略、审计日志、网络安全、最小权限原则
8. **实际应用**：多租户安全、生产环境配置

掌握这些安全功能，可以构建安全可靠的Elasticsearch集群，保护数据安全。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch安全认证与授权](http://zhouzhiyang.cn/2025/05/Elasticsearch_Security_Authentication/)

