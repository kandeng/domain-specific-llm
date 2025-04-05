# Introduction

According to ["Injecting Domain-Specific Knowledge into Large Language Models: A Comprehensive Survey"](https://arxiv.org/abs/2502.10708), 
so far there are 4 paradigms for the domain specific LLMs, illustrated as the following diagram. 

![4 paradigms for the domain specific LLMs](https://github.com/kandeng/domain-specific-llm/blob/main/assets/domain_specific_llm_4_paradigms.png)

The boundaries of the 4 paradigms are not fixed. In practise, it is more likely to integrate multiple techniques, rather than use only one single technique. 

1. For the private data and the recently updated data, "dynamic knowledge injection", especially [Retrieval-Augmented Generation (RAG)](https://arxiv.org/abs/2005.11401)
   is very useful.

2. For static domain knowledge, Supervised Fine Tuning (SFT) with [Low Rank Adaptation (LoRA)](https://arxiv.org/abs/2106.09685) is very popular,
   to improve the quality of the answer, and to reduce the response time.

3. To reduce the computational cost of SFT, in addition to LoRA, many other techniques are also used, including
   
   * modular adapter, e.g. [llama adapter](https://arxiv.org/abs/2303.16199),
     
     The adapters are plugins to the LLM with domain knowledge.
     And the SFT training only occurs to those adapters, rather than the LLM.
     
   * mixture of experts, e.g. [mistral](https://arxiv.org/abs/2401.04088),
  
     MoE aims at reducing the training cost of the feed-forward modules inside the transformers,
     by leveraging the sparseness of the matrics of feed-forward module's MLP weights. 
  
   * GPU IO-awareness, e.g. [flash attention](https://arxiv.org/abs/2205.14135)
  
     Flash attention directly uses Nvidia's low level C/C++ APIs, so as to optimize the IO efficiency between the GPU SRAM and HBM.

     Flash attention applies this approach to enhance the training efficiency of the attention modules inside the transformers.
     But in addition to the attention modules, the same approach can also applied to other modules of the transformers. 
  
