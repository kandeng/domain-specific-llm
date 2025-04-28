# 02. Distributed LLM training with DeepSpeed 

## 1. Objectives

The objectives of this article is to use [DeepSpeed](https://github.com/deepspeedai/DeepSpeed)
to train a large scale model, DeepSeek-R1-Distill-Qwen-32B, 
across multiple GPU servers (aka nodes) with multiple GPU cards. 

More specifically, we have two GPU servers (nodes), `172.16.80.33` and `172.16.80.31`, 
each one has eight H20 cards. And we want to use `172.16.80.33` as the master node. 

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


&nbsp;
## 2. SSH HostKeyAlgorithms

We want `172.16.80.33` and `172.16.80.31` to be able to mutually `ssh` to each other, 
without typing in password. 

### 2.1 Problem: The authenticity of host can't be established   

The first time when we `ssh` from `172.16.80.33` to `172.16.80.31`, we encountered the following error, 

~~~
(grpo) root@172.16.80.33:~/kdeng# ssh -p 22 root@172.168.80.31
The authenticity of host '172.16.80.31' can't be established. 
ED25519 key fingerprint is SHA256:gxYjzI+p8kBNPp5JcaHskQ+DgfUoriKsgJN0XOt5Ikw.
This key is not known by any other names. 
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Warning: Permanently added '172.16.80.31' (ED25519) to the list of known hosts. 
~~~

### 2.2 Solution: HostKeyAlgorithms=+ssh-rsa

The solution is to add `-o HostKeyAlgorithms=+ssh-rsa` to the command. 
This should be done only once, 
after then, we can `ssh` successfully. 

~~~
(grpo) root@172.16.80.33:~/kdeng# ssh -p 22 root@172.16.80.31 -o HostKeyAlgorithms=+ssh-rsa
root@172.16.80.31's password: ...
(base) root@172.16.80.31# exit

(grpo) root@172.16.80.33:~/kdeng#
~~~

### 2.3 Verification

The following is the correct result, when we `ssh` from `172.16.80.31` to `172.16.80.33` without password. 

You should double check that `172.16.80.33` can `ssh` to `172.16.80.31` without password, too. 

~~~
(grpo) root@172.16.80.33:~/kdeng# ssh root@172.16.80.31
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-18-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

扩展安全维护（ESM）Applications 未启用。

124 更新可以立即应用。
这些更新中有 9 个是标准安全更新。
要查看这些附加更新，请运行：apt list --upgradable

7 个额外的安全更新可以通过 ESM Apps 来获取安装。
可通过以下途径了解如何启用 ESM Apps：at https://ubuntu.com/esm

Last login: Mon Apr 28 15:39:37 2025 from 172.16.80.33
(base) root@yw01:~# exit
注销
Connection to 172.16.80.31 closed.

(grpo) root@172.16.80.33:~/kdeng# 
~~~


&nbsp;
## 3. SSH without password


