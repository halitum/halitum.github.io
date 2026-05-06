+++
title = "[读论文] GraphRAG"
date = "2024-12-16"
description = "首个将图引入RAG的架构"
tags = [
 "RAG",

 "LLM"
]
categories = [
 "大模型技术",
]
image = "https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20241218162305869.png"

+++

> “在大部分指标上存在特定社区层次结果优于baseline，存在缺点1. 不能根据 Query 的期望的潜在搜索广度/深度动态适应不同层次的社区摘要进行回答，2. 耗费太高，3. 知识库难以实现增删改查。官网的项目配图很好看，评价为未来可期”

*原文地址：[From Local to Global: A Graph RAG Approach to Query-Focused Summarization](https://arxiv.org/pdf/2404.16130)*

## Highlight

通过两个阶段（索引和查询）构建基于图的文本索引

1. 先从源文档构建实体知识图，然后为密切相关的实体组预生成社区（community）摘要
2. 对于Query，将每个社区摘要作为生成的部分响应（partial response），在最终响应（final response）中总结所有部分响应



## Approach & Pipeline

![image-20241218172716864](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20241218172716864.png)

- ###  Source Documents →Text Chunks

  每个文本块被 LLM prompts 提取出图索引中的元素。值得注意的是，chunk越大，LLM调用就越少，但相应会导致召回率下降

  > **召回率（Recall）** 是信息检索中的指标，用于衡量被检索系统正确识别的重要信息占全部重要信息的比例。公式为：
  > $$
  > \text{召回率} = \frac{\text{正确检索出的重要信息数量}}{\text{全部重要信息数量}}
  > $$
  > 

  ![image-20241218174112881](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20241218174112881.png)

  随着 chunk size 上升，提取到的实体数目随之下降。解决方法是引入下文的“多轮收集”机制

- ### Text Chunks→Element Instances

  Element 指的是节点、边、或声明（声明就是节点或边的附加信息）

  这一步是从每个源文本块中识别和提取图节点（实体）和边（关系），使用多部分 LLM prompts （主要是 few-shot examples 方法）来完成此操作。提取出的 实体 和 关系 使用元组来表示。该元组属于一个 **实例组**

  > 例如：
  > （实体1，关系1，实体2）、（实体2，关系2，实体3）

  prompts 的具体要求是，首先识别文本中的所有实体，包括它们的名称、类型和描述，然后识别明确相关实体之间的所有关系，包括源和目标实体，以及它们之间关系的<u>描述</u>（作为声明）

  这一步使用多轮“收集”，直到指定的最大值，允许我们使用更大的 chunk size，而不会降低质量

- ### Element Instances → Element Summaries

  这一步的目的是从元素中提取出抽象摘要的描述。具体操作为，在匹配的实例组上进行进一步的LLM总结
  一个潜在的问题：同一实体会被提取出多种不同表示的元素实例。解决方法是在之后的步骤中将语义相近的实例组划分到一个社区中

- ### Element Summaries → Graph Communities

  使用聚类算法将语义相近的元素实例合并为一个社区（社区中的节点比其他节点具有更强的连接）

  > 具体来说，graphRAG中使用了 Leiden **社区检测**（Community Detection）的图论算法
  >
  > 这种算法可以构建出 **分层的社区结构**——社区不仅仅是独立的群体，而是具有层级关系的群体结构
  >
  > **Leiden算法中的分层社区结构**
  >
  > 它通过 **分层合并** （coarsing）过程，首先将图分成较大、较粗的社区，然后逐步细化这些社区，最终得到多个层级的社区划分。
  >
  > - **第一层**：图的最顶层会将节点分为较大的社区。
  > - **第二层及以后**：每个较大的社区会进一步细化为子社区，直到达到较小的细分。

- ### Graph Communities→Community Summaries

  这一步的目的是为 Leiden 层次结构中的每个社区创建类似报告的摘要，这些报告为每个子主题提供了更多细节。

  社区摘要生成方式：

  - 叶级社区：为每个元素摘要进行排序（按总体突出度降序），为其进行报告生成
  - 更高级的社区：如果所有子社区的元素摘要可以适配 LLM 上下文窗口大小，则直接生成报告。否则将长元素摘要替换成同社区内的短元素摘要

- ### Community Summaries→Community Answers→Global Answer

  给定一个用户查询，可以使用前一步生成的社区摘要在多阶段过程中生成最终答案

  对于给定的社区级别，答案生成流程为：

  1. 打乱社区摘要，然后分成多个预定大小的块（大小由**token数量限制**决定）
  2. 对每个摘要块并行生成 Community Answers，然后用 LLM 对生成的答案打分（0-100），过滤掉0分答案
  3. 将 Community Answers 按得分从高到低排序，选出几个帮助度最高的答案，添加到上下文窗口直到达到token限制。最终生成的全局答案会被返回给用户

  

## Evaluation

> • CO. Uses root-level community summaries (fewest in number) to answer user queries.
> • C1. Uses high-level community summaries to answer queries. These are sub-communities
> of C0, if present, otherwise C0 communities projected down.
> • C2. Uses intermediate-level community summaries to answer queries. These are sub-
> communities of C1, if present, otherwise C1 communities projected down.
> • C3. Uses low-level community summaries (greatest in number) to answer queries. These
> are sub-communities of C2, if present, otherwise C2 communities projected down.
> • TS. The same method as in subsection 2.6, except source texts (rather than community
> summaries) are shuffled and chunked for the map-reduce summarization stages.
> • SS. An implementation of na¨ıve RAG in which text chunks are retrieved and added to the
> available context window until the specified token limit is reached.

![image-20241218190402441](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20241218190402441.png)

> “消耗了巨量 token 但未指定社区层级的生成效果却很一般？”

结果表明在不同的任务上，不同层次社区的生成结果在某方面的表现与其他层次社区存在差异。值得注意的是，所有图方法的 Directness 指标都劣于 baseline
