---
layout: post
title: "vLLM部署与优化"
date: 2025-10-05 
description: "vLLM框架、PagedAttention、连续批处理、部署配置、性能优化、最佳实践"
tags: ["大模型"]

---

## 为什么默认会选 vLLM

试过几家之后，生产推理我还是经常回到 vLLM：OpenAI 兼容接口省事，连续批处理吞吐靠谱，社区坑也相对好搜。PagedAttention 把 KV 分页管起来，显存碎片少很多——这点比「理论加速比」更打动我。

当然它不是万能的。配置乱拧一样 OOM；多模型硬塞一张卡也会互相拖死。下面按我实际起服务的顺序记。

## 安装

```bash
pip install vllm

# 版本和 CUDA 对不上时，宁可看官方 wheel 说明，别盲装最新
# 源码装适合要跟某个 commit：
# git clone https://github.com/vllm-project/vllm.git && cd vllm && pip install -e .
```

装完先 `python -c "import vllm; print(vllm.__version__)"`，再 `nvidia-smi` 确认驱动可见。容器里看不到 GPU，九成是 runtime / device plugin 问题，不是 vLLM 本身。

## 最小可跑

```python
from vllm import LLM, SamplingParams

llm = LLM(model="Qwen/Qwen2.5-7B-Instruct")
params = SamplingParams(temperature=0.7, top_p=0.9, max_tokens=256)

outputs = llm.generate(["用三句话说清楚什么是 KV cache"], params)
print(outputs[0].outputs[0].text)
```

本机验证用这个够了。对外服务别用同步 `LLM` 堵着，直接上 OpenAI 兼容入口。

## 我真正用来对外的启动方式

```bash
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-7B-Instruct \
  --host 0.0.0.0 \
  --port 8000 \
  --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.90 \
  --max-model-len 8192 \
  --max-num-seqs 64
```

调用侧跟 OpenAI SDK 几乎一样，只改 `base_url`：

```python
from openai import OpenAI

client = OpenAI(base_url="http://127.0.0.1:8000/v1", api_key="not-needed")
resp = client.chat.completions.create(
    model="Qwen/Qwen2.5-7B-Instruct",
    messages=[{"role": "user", "content": "你好"}],
    stream=False,
)
print(resp.choices[0].message.content)
```

业务请走网关，不要让前端/工作流直连 `8000`。换模型名、切金丝雀时，直连会变成全员改配置。

## 几个参数我怎么拧

**`gpu_memory_utilization`。** 文档常写 0.9。我的习惯是先 0.85 跑稳，看显存曲线再抬。贴满 0.95 短期吞吐好看，长上下文一来很容易炸。

**`max_model_len`。** 开太大是 OOM 重灾区。业务上下文实际只用 4k，就别为了「看起来能 32k」把 KV 池撑满。

**`max_num_seqs` / 批处理相关。** 并发上限。超了应该在网关排队或 429，而不是把引擎打到集体重启。单请求摸底显存后，往上加并发看曲线，留 10%～20% 余量再对外开放。

**`tensor_parallel_size`。** 单卡装不下再拆。多卡尽量同 NVLink 域；跨 NUMA 硬拆，延迟会变得很难解释。

量化模型直接指到 AWQ/GPTQ 权重路径即可，比先上全精度再后悔省事。精度掉多少，用你自己的评测集看，别只看官方 demo。

## 踩过的坑

- **OOM：** 先降 `max_num_seqs` 和 `max_model_len`，再考虑量化或加卡。碎片严重时重启能缓，但治本还是限流。
- **吞吐上去、首包变差：** 连续批把 GPU 喂饱了，队列也长了。对话场景要盯 TTFT，不能只看 tokens/s。
- **共卡：** 和别的训练/推理进程抢同一张卡，延迟会一阵一阵。核心业务我倾向独占。
- **流式被反代缓冲：** Nginx 之类没关 buffering，客户端感觉「卡住再一次性吐出」。SSE 相关超时和缓冲要单独查。

指标至少看：GPU 利用率、显存、请求成功率、p95 TTFT。vLLM 的 `/metrics` 能接 Prometheus，别等出事故再补。

对我来说 vLLM 是「默认能扛事」的那一档；把显存账和并发上限算清楚，比追新特性重要得多。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [vLLM部署与优化](http://zhouzhiyang.cn/2025/10/vLLM_Deployment/)
