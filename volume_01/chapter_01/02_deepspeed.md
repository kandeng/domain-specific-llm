# 02. Distributed LLM training with DeepSpeed 

## 1. Objectives

The objectives of this article is to use [DeepSpeed](https://github.com/deepspeedai/DeepSpeed)
to train a large scale model, DeepSeek-R1-Distill-Qwen-32B, across multiple GPU servers. 

[Unsloth](https://unsloth.ai/introducing) is a great tool that can make LLM training 30x faster. 
However, in case your model's scale is too large to fit into one GPU server, and you don't want to pay 
for [the advanced version of unsloth](https://unsloth.ai/pricing), 
you need an alternative tool to replace unsloth for the distributed model training. 

In our practice, we use DeepSpeed for distributed model training. 

