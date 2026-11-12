---
layout: post
title: "ELK生产环境最佳实践"
date: 2025-05-15 
description: "生产环境部署、配置优化、安全加固、运维管理、故障处理、容量规划"
tag: ELK

---

生产上 ELK，我按「先算盘、再拆角色、再开安全、最后盯盘」这个顺序来，比一上来抄大而全配置靠谱。

## 容量先估一下

粗算：

```
主存容量 ≈ 日增原始日志 × 膨胀系数(1.1~1.5) × 保留天数 × (1 + 副本数)
分片数 ≈ 总存储 / 30GB（目标单分片大小）
```

预留 20%～30% 磁盘，别顶着水位跑。查询 QPS 高要加 data / coordinating，不是只加磁盘。

## 架构底线

- ES：≥3 节点；副本 ≥1；生产开安全  
- 采集：Filebeat 上业务机；复杂解析才 Logstash；高峰加 Kafka  
- Kibana：≥1，重要再双活 + 反代  
- 索引：模板 + ILM，禁止无限留存  

```yaml
# elasticsearch.yml 生产常见项
cluster.name: production-es
node.name: ${HOSTNAME}
network.host: 0.0.0.0
discovery.seed_hosts: ["es1", "es2", "es3"]
# bootstrap 完成后不要长期依赖 initial_master_nodes 乱改

xpack.security.enabled: true
```

```
# jvm.options
-Xms16g
-Xmx16g
```

堆 ≤ 物理一半且尽量 ≤32g；其余留给文件系统缓存。

## 配置上少踩的坑

- 分片别默认 5；日志小索引 1 主分片起步  
- mapping 显式化，动态字段收敛  
- 批量写调大 refresh，查多再调小  
- Logstash `pipeline.workers` / `batch.size` 按 CPU 调，开 persisted queue  
- Kibana 与 ES 版本对齐  

## 安全与备份

- 不用 elastic 做日常读写；API Key / 专用角色  
- TLS 至少 transport；HTTP 可放在 LB  
- 快照仓库（FS/S3/OSS）定期拍，**恢复演练做过才算数**  

```bash
PUT /_snapshot/backup_repo
{
  "type": "fs",
  "settings": { "location": "/mount/es-backup" }
}

PUT /_snapshot/backup_repo/snap-001?wait_for_completion=false
{
  "indices": "logs-*",
  "include_global_state": false
}
```

## 运维节奏

每日/值班：`_cluster/health`、磁盘、堆、ILM explain。  
变更：模板和 ILM 改动走评审，先在新索引验证。  
文档：集群拓扑、账号归属、告警人、回滚步骤写在内部 wiki，别只在个人笔记。

硬件参考（可按预算缩）：数据节点 16C+/64G+/SSD；Logstash 看解析复杂度；千万别和 ES 抢同一块已经很满的盘。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK生产环境最佳实践](http://zhouzhiyang.cn/2025/05/ELK_Production_Best_Practices/)
