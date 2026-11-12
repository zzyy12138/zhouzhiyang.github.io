---
layout: post
title: "Elasticsearch安全认证与授权"
date: 2025-05-02 
description: "安全概述、用户认证、角色权限、API密钥、TLS/SSL配置、安全最佳实践"
tags: ["ELK"]

---

8.x 默认开安全。开发图省事关掉可以，生产千万别。这篇记用户、角色、API Key 和 TLS 我实际用过的部分。

## 用户

内置文件用户（小环境够用）：

```bash
bin/elasticsearch-users useradd appuser -p '***' -r kibana_admin
bin/elasticsearch-users passwd appuser
bin/elasticsearch-users list
```

原生 realm 用 API（可进 `.security` 索引，方便集群共享）：

```bash
curl -u elastic:password -X POST "localhost:9200/_security/user/appuser?pretty" \
  -H 'Content-Type: application/json' -d'
{
  "password": "changeme",
  "roles": ["logs_reader"],
  "full_name": "App Reader"
}
'
```

`elastic` 只留给救急；Kibana 用 `kibana_system`，Logstash/Beats 各自系统用户或专用角色。别人手一把 superuser。

## 角色

```bash
curl -u elastic:password -X POST "localhost:9200/_security/role/logs_reader?pretty" \
  -H 'Content-Type: application/json' -d'
{
  "cluster": ["monitor"],
  "indices": [
    {
      "names": ["logs-*", "nginx-*"],
      "privileges": ["read", "view_index_metadata"],
      "field_security": {
        "grant": ["*"],
        "except": ["password", "token", "id_card"]
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

索引权限常用：`read` / `write` / `create_index` / `manage`。字段级、文档级安全（`query` 限制可见文档）适合多租户，规则别写太绕，难排查。

## API Key

给程序用，比在配置文件里塞用户密码好撤：

```bash
curl -u elastic:password -X POST "localhost:9200/_security/api_key?pretty" \
  -H 'Content-Type: application/json' -d'
{
  "name": "filebeat-writer",
  "expiration": "90d",
  "role_descriptors": {
    "writer": {
      "cluster": ["monitor"],
      "indices": [
        { "names": ["logs-*"], "privileges": ["create_doc", "create_index", "auto_configure"] }
      ]
    }
  }
}
'

# 请求头
# Authorization: ApiKey <base64(id:api_key)>
```

创建时返回的 key 只显示一次，存进密钥管理。不用了就 invalidate。

## TLS

节点间 transport SSL 生产建议开；HTTP SSL 看是否已有网关终结 TLS。

```bash
bin/elasticsearch-certutil ca
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

```yaml
# elasticsearch.yml 示例
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: http.p12

xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: transport.p12
xpack.security.transport.ssl.truststore.path: transport.p12
```

证书路径、权限、密码（keystore）配错时节点会起不来，仔细看日志里的 SSLHandshake。

## 其它

公司有 LDAP/AD 再配 realm，顺序用 `order` 控制。审计可开：

```yaml
xpack.security.audit.enabled: true
xpack.security.audit.logfile.events.include: [access_denied, authentication_failed]
```

9200/9300 别对公网；Kibana 前面加认证反向代理。最小权限、能 API Key 不用超级用户，这两条做到就不容易翻车。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch安全认证与授权](http://zhouzhiyang.cn/2025/05/Elasticsearch_Security_Authentication/)
