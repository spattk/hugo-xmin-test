---
title: "prefill vs decode : llm inference phases"
date: 2026-09-10T19:14:00.000-07:00
author: Sitesh Pattanaik
---
Every llm request runs in two distinct phases: prefill, where the model reads the whole prompt in one parallel burst, and decode, where it generates the response one token at a time, each one depending on the last.

* if the chatbot feels slow before the first word appears, then it's a prefill issue.
* if it crawls during the token generation, then it's a decode issue

**Prefill** - this processes your entire input prompt at once, including system instruction, retrived context, and the user message. It builds the internal state, called the KV cache, that the model needs for the generation phase.

**Decode** - this generates response one token at a time. using the cache state from prefill, it produces a token, feeds it back in, and repeats this until the response is complete.

due to the way the job is done, prefill is a compute bound process while decode is a memory-bandwidth-bound process considering how fast GPU can move data around.

### Prefill: drives the time to first token (ttft)

* attention is the catch
* the length of the input prompt is roughly increases the attention work by a square of the length
* prefill has to finish everything before the model can output anything
* hence, for RAG based workflows, which have huge input prompts, the intial response time feels much longer

#### optimization levers

* **efficient attention and flash attention**: there are a class of optimizations that helps prefill faster but the model produces same output.

  * FlashAttention doesn't do approximations
  * it fuses operations such as SoftMax, dot-product etc and minimizes GPU memory reads/writes by using tiling techniques and efficient memory access patterns. (will cover in a later blog)
* **semantic caching**: it works at the inference pipeline which reuses the LLM responses if the new prompt is sematically similar to the one earlier.

  * this is a vector search problem where the incoming vectors are compared against query vectors

### Decode: token generation drives the inter token latency (itl)

* every decode step depends on the prior token generated
* the memory is the kv cache
* itl = average per token time
  = (end_to_end_latency - ttft)/(out_token - 1)

#### optimization levers

* **speculative decoding**: this uses a small model to guess what tokens the main model would produce, then has the main model verify them in parallel.

  * when the guesses are right, the user get multiple tokens for the cost of a single decode step.
  * the catch is the accuracy of the small/draft model, otherwise it adds up the latency.
* **quantization**: it shrinks the numeric representation of the model's data, using fewer bits per number to store roughly the same information.

  * trade offs accuracy a bit.

#### Appendix

Reference: <https://redis.io/blog/prefill-vs-decode/>
