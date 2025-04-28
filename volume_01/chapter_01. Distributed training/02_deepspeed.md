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

We want `172.16.80.33` and `172.16.80.31` to be able to mutually `ssh` to each other. 

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

The following was a succussful example, when we `ssh` from `172.16.80.31` to `172.16.80.33`. Notice that we were asked for the password when logging in. 

You should double check that `172.16.80.33` can `ssh` to `172.16.80.31`, too. 

~~~
(grpo) root@172.16.80.33:~/kdeng# ssh -p 22 root@172.16.80.31
root@172.16.80.31's password: ...

Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-18-generic x86_64)
...
(base) root@yw01:~# exit
注销
Connection to 172.16.80.31 closed.
(grpo) root@172.16.80.33:~/kdeng# 
~~~


&nbsp;
## 3. SSH without password

Not only did we want `172.16.80.33` and `172.16.80.31` to be able to mutually `ssh` to each other, 
but also wanted them `ssh` to each other without typing in password. 

### 3.1 Problem: ssh needs password

~~~
(grpo) root@172.16.80.33:~/kdeng# ssh -o PasswordAuthentication=no 172.16.80.31
root@172.16.80.31: Permission denied (publickey,password).
~~~

### 3.2 Solution: ssh-copy-id user@host

**Step 1.** Set PasswordAuthentication yes

According to ["Error 'Permission denied (publickey,password)'"](https://superuser.com/questions/912531/error-permission-denied-publickey-password),
the first step is to set `PasswordAuthentication yes` in file `/etc/ssh/sshd_config` at the destination machine. 

Since we wanted `172.16.80.33` and `172.16.80.31` mutually `ssh` to each other without password, 
we modified `/etc/ssh/sshd_config` files on both machines. 

**Step 2.** ssh-copy-id from 172.16.80.33 to 172.16.80.31

According to ["ssh-copy-id no identities found error"](https://stackoverflow.com/questions/22530886/ssh-copy-id-no-identities-found-error),
the simplest way is to,

~~~
ssh-keygen
[enter]
[enter]
[enter]

cd ~/.ssh
ssh-copy-id -i id_rsa.pub USERNAME@SERVERTARGET
~~~

Therefore, on `172.16.80.33`, we did the following,

~~~
(grpo) root@172.16.80.33:~/kdeng# cd ~/.ssh/
(grpo) root@172.16.80.33::~/.ssh# pwd
/root/.ssh

(grpo) root@172.16.80.33::~/.ssh# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:BbGviX75OPi01kqy61BHX/T+oSBxPVy3kds9NUKZhvw root@yw-NF5688-M7-A0-R0-00
The key's randomart image is:
+---[RSA 3072]----+
|        o.. +.ooo|
|         o +o*.o=|
|        o...++o.*|
|       . +o. E.oo|
|      . S.o.  ...|
|     . o o. . ...|
|    . o.=o   .  .|
|     o.==o.      |
|     .===+.      |
+----[SHA256]-----+

(grpo) root@172.16.80.33::~/.ssh# ls
id_rsa  id_rsa.pub  known_hosts  known_hosts.old

(grpo) root@172.16.80.33::~/.ssh# ssh-copy-id -i id_rsa.pub root@172.16.80.31
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@172.16.80.31's password: ...

Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'root@172.16.80.31'"
and check to make sure that only the key(s) you wanted were added.

(grpo) root@172.16.80.33::~/.ssh#
~~~

**Step 3.** ssh-copy-id from 172.16.80.31 to 172.16.80.33

Similar to the above. 

**Step 4.** Set PasswordAuthentication no

According to ["Error 'Permission denied (publickey,password)'"](https://superuser.com/questions/912531/error-permission-denied-publickey-password),
the last step is to set `PasswordAuthentication no` in file `/etc/ssh/sshd_config` on both machines. 


### 3.3 Verification

The following is a successful example that `172.16.80.33` can successfully 
`ssh -o PasswordAuthentication=no` to `172.16.80.31`, without password.

Or, even simpler, `172.16.80.33` can successfully 
`ssh` to `172.16.80.31` without being asked for password.

You should double check that `172.16.80.31` can successfully 
`ssh` to `172.16.80.33` without password, too. 

~~~
(grpo) root@172.16.80.33:~/kdeng# ssh -p 22 -o PasswordAuthentication=no root@172.16.80.31
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-18-generic x86_64)
...
Last login: Mon Apr 28 22:46:26 2025 from 172.16.80.33
(base) root@yw01:~# exit
注销
Connection to 172.16.80.31 closed.
~~~
~~~
(grpo) root@172.16.80.33:~/kdeng# ssh -p 22 root@172.16.80.31
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.5.0-18-generic x86_64)
...
Last login: Mon Apr 28 22:46:26 2025 from 172.16.80.33
(base) root@yw01:~# exit
注销
Connection to 172.16.80.31 closed.
~~~

&nbsp;
## 4. pdsh: remote SHells

`pdsh` stands for Parallel Distributed SHells. 
The `pdsh` tool is arguably one of the most popular parallel remote shells. 
It allows you to run commands on multiple nodes using `SSH`. 

Installation,

~~~
(grpo) root@172.16.80.33:~/kdeng# sudo apt-get -y install pdsh
正在读取软件包列表... 完成
正在分析软件包的依赖关系树... 完成
正在读取状态信息... 完成                 
pdsh 已经是最新版 (2.31-3build2)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 140 个软件包未被升级。
(grpo) root@172.16.80.33:~/kdeng# 
~~~

Verification, 

~~~
(grpo) root@172.16.80.33:~/kdeng# pdsh -w ssh:172.16.80.33,ssh:172.16.80.31 date
172.16.80.33: 2025年 04月 28日 星期一 23:37:02 CST
172.16.80.31: 2025年 04月 28日 星期一 23:37:56 CST
~~~
