---
layout: post
title: "ELK故障排查指南"
date: 2025-05-20 
description: "常见问题、故障诊断、日志分析、性能问题、集群问题、恢复方案"
tag: ELK

---

排障时我固定顺序：健康色 → 节点还在不在 → 磁盘/堆 → 未分配分片原因 → 组件日志。先收集再改配置，避免瞎重启。

## 集群红 / 黄

```bash
curl -s "localhost:9200/_cluster/health?pretty"
curl -s "localhost:9200/_cat/nodes?v"
curl -s "localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason" | grep UNASSIGNED
```

- **red**：有主分片丢了。找挂掉的 data 节点、盘满、分配被关（`cluster.routing.allocation.enable`）。  
- **yellow**：副本没地方放。单节点必黄；多节点则查副本数是否大于 data 节点数、磁盘水位、awareness 是否过严。  

```bash
curl -s "localhost:9200/_cluster/allocation/explain?pretty"
```

这条经常直接告诉你为什么不分配。

## 节点起不来

```bash
journalctl -u elasticsearch -n 200 --no-pager
# 或
tail -200 /var/log/elasticsearch/*.log

free -h
df -h
sysctl vm.max_map_count
ulimit -n
ss -lntp | grep 9200
```

常见：bootstrap check 失败（mmap、文件句柄）、堆设超过可用内存、权限/属主不对、证书密码错。OOM 被杀看 `dmesg`。

## 写入失败 / 堆积

- ES：`_cat/thread_pool/write?v`、磁盘是否只读、circuit breaker  
- Logstash：9600 pipelines 统计、`_grokparsefailure` 比例、队列是否打满  
- Filebeat：registry、输出报错、磁盘  

```bash
curl -s "localhost:9600/_node/stats/pipelines?pretty" | head -c 4000
```

## 查询又慢又超时

```bash
# 看段、字段数据、断路器
curl -s "localhost:9200/_nodes/stats/indices/fielddata,query_cache,segments,breakers?pretty" | less
```

排查点：时间范围是否过大、是不是扫了 text fielddata、深分页、聚合 size 过大、热点分片。临时用 `profile: true` 看哪段慢（别在生产高频开）。

## 数据「丢了」

多数不是丢，是：

1. 查错索引名 / 别名  
2. 时间字段或时区不对  
3. ILM 删了  
4. 过滤条件太狠  

先 Discover 放大时间范围、去掉 filter。真要救数据，靠快照恢复，别指望 translog 万能。

## Logstash / Kibana 连通

```bash
bin/logstash -f pipeline.conf --config.test_and_exit
curl -s -o /dev/null -w "%{http_code}" http://localhost:5601/api/status
```

401/403 先核对用户角色；Kibana 连不上 ES 看 `kibana.yml` 的 hosts 和证书。版本大跨度混用不支持。

留一个习惯：改任何 cluster settings 前先 `GET _cluster/settings`，把旧值记下来，方便回滚。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK故障排查指南](http://zhouzhiyang.cn/2025/05/ELK_Troubleshooting_Guide/)
