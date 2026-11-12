---
layout: post
title: "TensorRT-LLM部署"
date: 2025-10-15 
description: "TensorRT-LLM、NVIDIA优化、GPU加速、模型编译、性能优化、部署实践"
tag: 大模型

---

## 先泼冷水

TensorRT-LLM 的峰值性能经常很好看，尤其在固定模型、固定形状、NVIDIA 栈可控的时候。但代价也很实在：环境挑、要编译引擎、换模型或改 `max_batch_size` / 序列长度往往得重新 build。  

我把它定位成「延迟抠到极致、模型变更不频繁」的选项，而不是日常试模型的默认项。日常迭代还是 vLLM 省心。

## 环境比代码先对齐

大致需要：较新的 NVIDIA 驱动、匹配的 CUDA、TensorRT，以及项目声明的 Python 版本。版号对不上时，报错信息经常很绕。

```bash
nvidia-smi
nvcc --version
# TensorRT / TRT-LLM 版本以你选用的发行说明为准
```

很多人直接上官方容器，少踩本地编译依赖：

```bash
docker pull nvcr.io/nvidia/tensorrt-llm:latest
# 实际 tag 请换成你需要的固定版本，别长期漂在 latest
```

从源码装可以，但第一次建议容器里把流程跑通，再决定要不要固化到自己的镜像。

## 核心心智：先编译引擎，再推理

和「加载 HF 权重就能 serve」不同，TRT-LLM 典型路径是：权重 → 转换/构建 → `.engine` → runtime 加载。编译参数写进引擎里，所以：

- `max_batch_size`、`max_input_len`、`max_output_len` 要按生产峰值留余量，又不能无脑开最大（显存和编译时间都会炸）
- 改了这些上限，旧引擎往往不能直接复用
- 量化（INT8 / FP8 等）一般在构建阶段就定了

官方仓库的 example / convert 脚本随版本变，这里不贴容易过期的伪 API。实操时以当前文档的 `trtllm-build`（或对应版本命令）为准，把构建参数写进自己的脚本和镜像构建阶段，避免人工点。

示意（命令名随版本可能不同）：

```bash
# 伪流程：转换检查点 → build 引擎 → 放到固定目录供 runtime 加载
# trtllm-build --checkpoint_dir ... --output_dir ./engines/qwen7b \
#   --gemm_plugin float16 --max_batch_size 8 --max_input_len 4096 --max_seq_len 8192
```

构建尽量在流水线里做，产物当版本产物存起来（按模型名 + 精度 + 形状配置打 tag）。线上只读引擎，不在高峰现场编译。

## 运行时注意点

- **批处理：** 引擎按构建时的 max batch 工作；网关侧并发超过构建上限，行为取决于你怎么包服务，别假设「会自动变大」。
- **多卡：** 张量并行同样要在构建/启动配置里声明，卡拓扑差会直接反映到延迟。
- **和业务解耦：** 外面仍建议 OpenAI 兼容或自研网关，引擎升级时只切后端，不改调用方。
- **监控：** 编译成功 ≠ 服务健康。照样盯显存、TTFT、错误率；引擎加载失败要有明确就绪探针，避免半死不活接流量。

## 什么时候值得上

- 模型版本稳定，一周不会换三次权重
- 延迟 SLO 紧，vLLM 调参后仍差一截，且 profiling 证明瓶颈在引擎侧
- 团队能接受「构建产物版本化」这套发布方式

如果还在天天换基座、改上下文长度，先用 vLLM/SGLang 把产品和流量形状跑明白，再考虑把热点模型冻成 TRT 引擎。性能数字很好看，但运维节奏跟不上的话，只会变成更难回滚的包袱。

转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [TensorRT-LLM部署](http://zhouzhiyang.cn/2025/10/TensorRT_LLM_Deployment/)
