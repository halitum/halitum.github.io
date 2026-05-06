+++
title = "[读论文] Agent设计模式目录"
date = "2025-04-15"
description = "对于agent软件设计中常见问题的可重用解决方案，《Agent design pattern catalogue》对其中的常用组件进行了收录和整理"
tags = [
 “LLM”,
 “Agent”
]
categories = [
 "大模型技术",
]
image = "https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250415001825379.png"

+++




> 本文中对agent的定义是“具有目标寻求和计划生成能力，以实现用户目标的AI”

# 该文章提出的 Agent 架构整体视图

![image-20250414120457772](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414120457772.png)

（省略了agent组件之间的详细交互）

- 上下文工程：获取用户具体意图与目标
- 提示词/响应工程：细化工具指令与提示词
- 计划生成：构建计划，将最终目标分解为可操作的任务
- 计划审查-人工干预模块：确保生成的计划准确、完整、可用

特别的，对于同一个任务，可以由多个不同的agent同时工作，并通过基于投票、基于角色或基于辩论的合作来最终确定结果（例如，agent分别扮演协调者coordinator和工作者worker，进行多轮交互后得出最终结论）

# 架构中的具体模块

1. ## 目标设定器

   执行上下文工程获取必要信息，并生成agent系统的最终目标。在上下文工程阶段，agent动态检索与用户对话的上下文取得足够的信息，在该阶段确认系统的最终执行目标。

   - 必要时可以调用传感器工具以取得额外信息
   - 可以引入人工干预，在与用户的多轮对话中追问和确认以敲定最终目标，确保准确性

   ![image-20250414150248367](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414150248367.png)

   ### 实现示例：

> [长短期-图+向量结合记忆力的动态Memory模块实现](https://github.com/mem0ai/mem0)
>
> ![image-20250414123127357](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414123127357.png)
>
> ![image-20250414123136517](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414123136517.png)
>
> [编程agent中引入屏幕截图信息判断用户的编程步骤](https://arxiv.org/abs/2304.14042)
>
> ![image-20250414124547816](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414124547816.png)

2. ## 提示/响应优化器

   执行提示词/响应工程，优化用户的提示词和agent系统的响应，或者使用特定提示词模板来构建最终的prompt输入

   目的：

   - 确保用户的提示词有效、便于用户理解模型的输出；
   - 确保用户的输入/模型的输出**符合json/MCP等上下文协议**，便于执行后续工作流。

   ![image-20250414125649571](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414125649571.png)

3. ## 计划生成器

   编排实现最终目标的中间步骤。具体流程是，先对任务进行拆解，而后按照优先级排序，这个过程中需要判断中间步骤是否可操作或最优。过程中的每一步都只有一个后续动作（CoT）。同时也可引入自一致性流程进行多次确认（CoT-SC）。

   ![image-20250414134102640](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414134102640.png)

   单路径计划生成实际上是CoT的实现。但计划的具体执行结果不一定同计划时的预期一致，因此需要引入多路径计划生成（ToT）。创建计划时在一些中间步骤会包含多个选择，以实现更好的鲁棒性。

   ![image-20250414135627791](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414135627791.png)

   *CoT——CoT-SC——ToT对比*：

   ![image-20250414141615532](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414141615532.png)

   ### 实现示例：

   > [NeurIPS 2023实现的tree-of-thought](https://github.com/princeton-nlp/tree-of-thought-llm)
   >
   > ```python
   > standard_prompt = '''
   > Write a coherent passage of 4 short paragraphs. The end sentence of each paragraph must be: {input}
   > '''
   > 
   > cot_prompt = '''
   > Write a coherent passage of 4 short paragraphs. The end sentence of each paragraph must be: {input}
   > 
   > Make a plan then write. Your output should be of the following format:
   > 
   > Plan:
   > Your plan here.
   > 
   > Passage:
   > Your passage here.
   > '''
   > 
   > 
   > vote_prompt = '''Given an instruction and several choices, decide which choice is most promising. Analyze each choice in detail, then conclude in the last line "The best choice is {s}", where s the integer id of the choice.
   > '''
   > 
   > compare_prompt = '''Briefly analyze the coherency of the following two passages. Conclude in the last line "The more coherent passage is 1", "The more coherent passage is 2", or "The two passages are similarly coherent".
   > '''
   > 
   > score_prompt = '''Analyze the following passage, then at the last line conclude "Thus the coherency score is {s}", where s is an integer from 1 to 10.
   > '''
   > ```
   >
   > [AutoGPT中实现tree-of-thought进行计划生成](https://github.com/Significant-Gravitas/AutoGPT)

4. ## 计划审查-人工干预模块

   目的：防止生成的计划可能会受到基础模型幻觉的影响，缺乏准确性。因此需要对生成的计划进行优化

   ![image-20250414142942373](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414142942373.png)

   具体实现上，可以通过agent自反思、agent交叉反思或人工干预来提供额外信息，辅助路径决策

   - agent自反思是一种形式化优化的过程，并在这一过程中不断对生成内容进行迭代审查和细化。具体来说，agent会在这一过程中回溯推理步骤，逐步进行验证，并将反思过程和结果可以保存在智能体的记忆中，用于持续学习。该模块的设计受到大模型的自我一致性能力的启发 [[archive\] Large Language Models Can Self-Improve](https://arxiv.org/abs/2210.11610)。

     相比较于其他反思方式，自反思没有额外的通信开销，有助于节省整体推理时间

     ### 使用示例：

     > [joonspk-research/generative_agents: Generative Agents: Interactive Simulacra of Human Behavior](https://github.com/joonspk-research/generative_agents)
     >
     > 每个智能体在一天结束时执行行两到三次自反思，并将结果存储在记忆流中

   - 交叉反思通过引入不同的agent或基础模型提供反馈并细化生成的计划和相应的推理程序

     ### 使用示例：

     > [XAgent](https://github.com/OpenBMB/XAgent)中，工具worker-agent持续向计划coordinate-agent发送反馈和反映，以指示任务是否完成，以对计划进行及时调整
     >
     > ![image-20250414144915296](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414144915296.png)

   - 计划执行完毕后可以向用户进行展示，用户给予反馈后基于该输入对计划进行持续迭代

5. ### Agent合作模式

   - 基于投票：利用多个agent在拥有各自视角的同时协作完成相同的任务，通过投票合作得出最终结果。具体来说，当一个agent需要生成响应时，该agent将生成多个候选响应，由其他拥有不同Memory或system-prompt的agent进行投票，由此选出最终输出的响应

     ![image-20250414151103994](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414151103994.png)

   - 基于角色：代理可以被分配各种角色，并以此完成决策。该设计主要考虑到不同模块的agent因为具有不同的Memory而会出现不同的表现，因此具体task执行的步骤将由特定agent来实现。特别的，当执行该步骤的agent不存在时，可以由assigner-agent来创建用于该功能的agent
     ![image-20250414151439538](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414151439538.png)

     ### 使用示例：

     > [一种分层的混合代理架构MoA](https://arxiv.org/pdf/2406.04692)：其中提议者agent为聚合器agent提供有用的参考响应，并且聚合器代理被分层组成以合成和细化响应。
     >
     > ![image-20250414151834742](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414151834742.png)

   - 基于辩论：不同agent在同一问题上同时工作，并同时生成自己的初始响应，之后提供给其他agent进行验证，同时提供指令和潜在优化方向。这个辩论过程可以迭代，以提高所有参与agent的性能。该模块的提出目的是解决agent独立完成任务时可能出现的“思维退化”风险 [[archive\] EncouragingDivergentThinkinginLargeLanguageModels throughMulti-AgentDebate](https://arxiv.org/pdf/2305.19118)

     ![image-20250414152827581](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414152827581.png)

     ### 使用示例：

     > [crewAI](https://github.com/crewAIInc/crewAI)：提供一个多代理编排框架，可以对多个代理进行分组，以共同讨论特定主题
     >
     > ![image-20250414152459750](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414152459750.png)
     >
     > 

6. ### 护栏

     提出目的：处理模型供应商返回的对抗性输出（对抗性输出例如“抱歉，这个问题我不能回答”）。防止该错误信息被发送到其他模型，或影响模型记忆即后续推理。同时满足道德标准和法律等特定要求。最后，护栏能够处理文本、音频、视频等多模态数据，在需要时进行模态转换。

     护栏可以作为复合AI系统中基础模型和所有其他组件之间的中间层。当基础模型接收任何消息时，护栏可以首先验证该信息是否符合特定的预定义要求。只有有效的信息才会传递给基础模型，而有风险或敏感的数据将在传输之前进行处理。同样，当基础模型响应时，护栏需要确保响应满足输入其他系统组件的基本要求。

     ![image-20250414153825925](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414153825925.png)

     ### 使用示例：

     > [guardrails-ai/guardrails: Adding guardrails to large language models.](https://github.com/guardrails-ai/guardrails)：列出了各种验证器，用于处理基础模型输入和输出中的不同风险
     >
     > ![image-20250414154216779](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414154216779.png)

7. ## Agent/Tools注册表

     目的：方便主动型代理（如ReAct）选择不同的外部代理和工具。同时，当agent-assinger生成出新的agent后，也将被登记在注册表中。

     ![image-20250414154526724](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250414154526724.png)

     注册表可以以不同的方式实现，例如，具有特定知识库的协调代理，基于区块链的智能合约等

8. ## Agent适配器

     用来连接Agent和外部工具：如MCP Cilent-Server，function call组件等

     ![image-20250415001045326](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250415001045326.png)

2. ## Agent评估器（AutoGen评估模块）

     根据不同的需求和指标执行测试来评估Agent

     ![image-20250415001100791](https://typora-picturelib.oss-cn-beijing.aliyuncs.com/image-20250415001100791.png)

     ### 相关工具：

     > [Inspect: A framework for large language model evaluations](https://github.com/UKGovernmentBEIS/inspect_ai)
     >
     > [confident-ai/deepeval: The LLM Evaluation Framework](https://github.com/confident-ai/deepeval)
     >
     > [promptfoo: Test your prompts, agents, and RAGs.](https://github.com/promptfoo/promptfoo)
     >
     > [ragas: Supercharge Your LLM Application Evaluations 🚀](https://github.com/explodinggradients/ragas)
