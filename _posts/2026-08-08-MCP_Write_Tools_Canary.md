---
layout: post
title: "MCP里加了切金丝雀，必须带确认串"
date: 2026-08-08 
description: "只读稳了以后，写操作限制在0到20%"
tags: ["大模型"]

---

只读用了一段时间，加了两个会改流量的工具：`set_canary_percent`、`rollback_alias_to_stable`。都调网关已有接口，不 SSH。

`percent` 只允许 0～20。确认串要长成 `CANARY_{alias}_{percent}`，对不上直接拒。同一别名带幂等，防止对话重试连打两次。

实际还是会遇到模型先改再问人。宿主侧把写工具设成需批准。审计里记下变更前后快照，光看「成功」不够。

全量切换继续走变更单，MCP 只覆盖小步金丝雀。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [MCP里加了切金丝雀，必须带确认串](http://zhouzhiyang.cn/2026/08/MCP_Write_Tools_Canary/)
