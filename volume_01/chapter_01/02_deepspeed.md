# 02. Distributed LLM training with DeepSpeed 

## 1. Objectives

The objectives of this article is to use [DeepSpeed](https://github.com/deepspeedai/DeepSpeed)
to train a large scale model, DeepSeek-R1-Distill-Qwen-32B, 
across multiple GPU servers (aka nodes) with multiple GPU cards. 

There are many tools for distributed model training. 

Among them, [unsloth](https://unsloth.ai/introducing) is a great tool that can make LLM training 30x faster. 
However, in case your model's scale is too large to fit into one GPU server, and you don't want to pay 
for [the advanced version of unsloth](https://unsloth.ai/pricing), 
you need an alternative tool to replace unsloth for the distributed model training. 

In our practice, we use DeepSpeed for distributed model training. 

There are many ways to use DeepSpeed, including
[HuggingFace Transformers](https://huggingface.co/docs/transformers/en/deepspeed), 
[HuggingFace Accelerate](https://huggingface.co/docs/accelerate/en/usage_guides/deepspeed) 
and [PyTorch Lightning](https://lightning.ai/docs/pytorch/stable/advanced/model_parallel/deepspeed.html).

Since our source codes deeply rely on Huggingface libraries, 
we use the combination of DeepSpeed with Huggingface transformers. 

We don't use the combination of DeepSpeed with Huggingface accelerate, 
because it seems that Huggingface accelerate is still in rapid upgrading. 
