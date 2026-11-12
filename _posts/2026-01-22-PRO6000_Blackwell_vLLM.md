---
layout: post
title: "RTX PRO 6000 上起 vLLM：Blackwell 的坑比模型多"
date: 2026-01-22 
description: "sm_120驱动与CUDA、FP8默认策略、单卡试跑和与A100镜像隔离"
tags: ["大模型"]

---

## 为什么单独记一张卡

PRO 6000（Blackwell，约 96GB）理论上很适合单卡扛 30B 级 + 较长上下文，而且 FP8 吞吐比 BF16 好看一截。实际上第一周时间多半花在「环境能认出这张卡」上，而不是选模型。

## 环境问题

旧的 A100 推理镜像直接拿来用，典型失败：

- PyTorch 看不到 `sm_120`
- vLLM attention backend 选错，eager 模式慢得离谱
- 驱动和 CUDA 用户态不一致

后来给这张卡单独一套：较新驱动 + CUDA 12.8/13 路线 + 对应 vLLM 构建。不要和 A100 池共用一个「万能」wheel。

冒烟命令先确认：

```bash
nvidia-smi
python -c "import torch; print(torch.cuda.get_device_name(0), torch.cuda.get_device_capability(0))"
```

算力显示 (12, 0) 一类再继续装服务。

## 单卡试跑策略

这张卡我主要用来：

- 验证某量化档（尤其 FP8）能不能起、质量抽检过不过
- 试长上下文配置，再决定要不要迁到 A100 多卡
- 给内部工具/编码助手做「单机专用」推理，不进公共高峰池

公开材料里 FP8 相对 BF16 常见有 2～3 倍量级的吞吐提升，我们抽测同模型同长度，方向一致。默认优先 FP8，除非业务强要求全精度。

```bash
# 示意：具体模型名随当时仓库而定
vllm serve <model-fp8> \
  --host 127.0.0.1 --port 8001 \
  --max-model-len 32768 \
  --gpu-memory-utilization 0.90 \
  --kv-cache-dtype fp8_e4m3
```

绑定 `127.0.0.1` 再经网关反代，避免 Docker 把端口直接曝到公网。

## 和 A100 的差异（体感）

- **互联**：单卡没事；若将来两张 PRO 只有 PCIe，TP 时可能要关 P2P/自定义 all-reduce，和 NVLink A100 不是同一套手感
- **基线不同**：同一模型在 PRO 6000 与 A100 上的 TTFT 不能共用一个告警阈值
- **定位**：试验床 + 专人工作负载，不是 8 卡池的第九张「同构卡」

这张卡跑通之后，新模型我会先在这里冒烟，再决定占几张 A100。省下的是主池被试验配置打挂的次数。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [RTX PRO 6000 上起 vLLM：Blackwell 的坑比模型多](http://zhouzhiyang.cn/2026/01/PRO6000_Blackwell_vLLM/)
