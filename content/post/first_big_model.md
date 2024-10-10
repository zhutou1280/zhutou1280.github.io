+++
title = '开发一个本地大模型应用'
date = 2024-10-09T14:45:23+08:00
draft = false
+++

[TOC]

## 一、大模型简介
### 1.1 什么是大模型
大语言模型（LLM，Large Language Model），也称大型语言模型，是一种旨在理解和生成人类语言的人工智能模型。
LLM 通常指包含数百亿（或更多）参数的语言模型，它们在海量的文本数据上进行训练，从而获得对语言深层次的理解。目前，国外的知名 LLM 有 GPT-3.5、GPT-4、PaLM、Claude 和 LLaMA 等，国内的有文心一言、讯飞星火、通义千问、ChatGLM、百川等。
大模型实际上是个语言模型，用来处理复杂的语言任务，如文本生成、语言翻译、问题回答等。
### 1.2 为什么叫大模型
大模型也是属于机器学习的一种，是深度学习的思想融入语言模型中的强大神经网络模型。
模型通俗的理解，就是数学模型，用公式来说就是y=fn(x1,x2)
大模型主要体现在这个大字。大的概念是一个参数多，能达到上亿个参数。对应公式中可能存在上亿个参数。另一个是数据量大，用于训练的数据可以达到PB级别。再者就是计算资源密集，训练和推理需要大量的计算资源。

## 二、检索增强生成RAG简介
### 2.1 什么是RAG
大模型不是万能的，在某些情况下，无法提供准确的答案。为了提高模型的性能和输出质量，产生了一种新的模型架构方式：检索增强生成（RAG, Retrieval-Augmented Generation）
#### 2.1.1 LLM面临的问题
- 知识更新滞后：LLM训练时用的静态数据集训练，比如说，训练的基础模型是24年前爬取的各类数据。那么原则上说，模型不可能知道2024年之后的信息内容。你问2024年发生了哪些大事件，模型肯定给不出对应的答案。所以通过RAG实时检索当前搜索引擎最新数据，保证内容的时效性，确保信息的持续更新和准确性。
- 专业领域知识欠缺：LLM在处理特定领域的专业知识时，效果不太理想。因为训练LLM的数据集都是公开的互联网数据以及各类公开发行的书籍之类的资源。但是如果是某个企业的私有业务功能及数据，LLM是不认识的，RAG可以检索特定领域的相关文档，为模型提供丰富上下文信息，从而提升专业领域内的问题回答质量和深度。
### 2.2 RAG工作流程
数据处理：
- 对原始数据进行清洗和处理
- 将处理后的数据转化为检索模型可以使用
- 将处理后的数据存储在特定数据库中
信息检索：检索器会根据用户的问题，从一个预先构建的知识库中找到最相关的文档或段落。
信息生成：生成器利用检索器提供的相关信息，再结合用户的输入，生成回答。
输出：返回基于外部知识增强后的生成结果给用户。

### 2.3 提高大模型准确性的其他方式
目前在提升大语言模型效果中，RAG和微调(Finetune)是两种主流的方法。
微调：在特定数据集上进一步训练大语言模型，来提升模型在特定任务上的表现。
RAG和微调的对比：
暂时无法在飞书文档外展示此内容
微调这块比较吃优质的数据集，并且还伴随大量的计算资源。成本比较高。优质的数据集，目前大部分是人工标注，都是人力堆出来的。而且一般个人开发者或者小公司，很少有资源能够去做这个微调。更关键的是，如果微调训练后的模型裂化了，那么这个模型就没用了，无法恢复到之前的某个状态。
LangChain简介
LangChain 是一个用于构建基于大语言模型（LLM）应用的框架，它使得开发者能够更容易地将语言模型与外部工具、数据源或环境集成起来，从而实现更复杂的应用。LangChain 的设计目标是帮助开发者搭建能够进行复杂推理、增强记忆、检索外部信息等功能的应用。
LangChain 的核心功能是将大语言模型（如 GPT、BERT 等）与多个外部系统结合起来，比如数据库、API、文件系统等，使得模型不仅能基于自身训练的知识进行推理，还能实时获取外部信息进行更准确或实时的回答。
[langchain官网](LangChain)

LangChain 的主要组件：
1. 语言模型（LLM）接口：LangChain 支持与多种大语言模型交互，比如 OpenAI 的 GPT 系列、HuggingFace 的模型、Cohere 等。开发者可以利用这些模型生成文本、回答问题、进行对话等。
2. 链（Chain）：链是 LangChain 中的一个核心概念，它允许将多个步骤或模型串联起来。例如，你可以先从数据库中检索信息，然后让大语言模型对这些信息进行总结，最后输出给用户。每个步骤都可以是一个独立的处理逻辑。
3. 工具（Tools）：LangChain 可以集成外部工具或 API，让大语言模型调用这些工具来扩展其功能。例如，模型可以通过查询数据库、访问网络 API 或者从文档库中检索信息来增强回答的质量。
4. 记忆（Memory）：LangChain 提供了一种“记忆”机制，使得语言模型可以在会话过程中记住之前的上下文。这对于实现对话机器人或长期交互系统非常有用，它可以让模型基于之前的对话内容生成连续、相关的回应。
5. 检索增强生成（RAG）：LangChain 也支持通过 RAG 技术来增强生成能力，结合外部的文档库或数据库，使模型生成更加准确和基于事实的内容。
6. 代理（Agents）：LangChain 支持创建智能代理系统，允许语言模型根据上下文和规则，动态选择要执行的动作。例如，一个代理系统可以根据用户的输入来决定是否需要从外部系统检索信息、执行计算，或与用户对话
一句话总结，这就是个大模型应用的开发框架，里面封装了各个组件，能够开箱即用，简化开发过程。
## 三、什么是大模型开发
以大语言模型为功能核心、通过大语言模型的强大理解能力和生成能力、结合特殊的数据或业务逻辑来提供独特功能的应用称为大模型开发。
开发大模型相关应用，其技术核心点虽然在大语言模型上，但一般通过调用 API 或开源模型来实现核心的理解与生成，通过 Prompt Enginnering 来实现大语言模型的控制，因此，虽然大模型是深度学习领域的集大成之作，大模型开发却更多是一个工程问题。
### 3.1 大模型开发与AI应用开发的差异
以调用、发挥大模型为核心的大模型开发与传统的 AI 开发在整体思路上有着较大的不同。大语言模型的两个核心能力：指令遵循与文本生成提供了复杂业务逻辑的简单平替方案。
- 传统的 AI 开发：首先需要将非常复杂的业务逻辑依次拆解，对于每一个子业务构造训练数据与验证数据，对于每一个子业务训练优化模型，最后形成完整的模型链路来解决整个业务逻辑。
- 大模型开发：用 Prompt Engineering 来替代子模型的训练调优，通过 Prompt 链路组合来实现业务逻辑，用一个通用大模型 + 若干业务 Prompt 来解决任务，从而将传统的模型训练调优转变成了更简单、轻松、低成本的 Prompt 设计调优。
同时，在评估思路上，大模型开发与传统 AI 开发也有质的差异。
- 传统 AI 开发：需要首先构造训练集、测试集、验证集，通过在训练集上训练模型、在测试集上调优模型、在验证集上最终验证模型效果来实现性能的评估。
- 大模型开发：流程更为灵活和敏捷。从实际业务需求出发构造小批量验证集，设计合理 Prompt 来满足验证集效果。然后，将不断从业务逻辑中收集当下 Prompt 的 Bad Case，并将 Bad Case 加入到验证集中，针对性优化 Prompt，最后实现较好的泛化效果。

### 3.2 大模型开发基础要素
1. 框架：LangChain
框架可以使用langchain，为了方便小白玩，langflow这个可视化编辑挺好用。
2. Embedding模型
embedding是什么，首先我们使用RAG来做增强检索，基于语言的检索不同于常规的关系数据库检索，而是向量检索，对于一段文本，用什么模型进行向量化，就是这个embedding模型需要做的。
3. 数据库
同样的，构建的知识库或者本地文档库，用什么来存储，就是这个数据库需要做的。一般这个数据库是个向量数据库。
4. 基础大模型
这个很容易理解，是openAI的chatgpt、还是文心一言、通义千问

## 四、开发一个本地大模型应用
上面介绍了很多，下面来实战。先最简单做一个应用。
### 4.1 依赖安装
#### 4.1.1 langflow安装
langflow是一个可视化编辑器。
安装命令（要求python3.10以上）
pip install langflow
#### 4.1.2 ollama下载
Ollama 是一种基于本地大语言模型的工具或框架，旨在让开发者能够在本地环境中运行和使用大语言模型，而无需依赖云端服务
去https://ollama.com/官网下载，安装。
有两个环境变量需要注意：
OLLAMA_MODELS：OLLAMA模型的下载路径。
OLLAMA_HOST: OLLAMA模型以服务方式运行的时候，即提供API，默认是只能被localhost访问的，设置这个为0.0.0.0，可以被网络访问。
#### 4.1.3 下载大模型
ollama安装完成后，命令行输入ollama，会有对应的命令帮助。
运行ollama pull llama3:8b，下载大模型。
下载完成后，运行ollama run llama3:8b ，可以进行语言聊天。
## 4.2 开始开发
### 4.2.1 运行langflow
在命令行中运行langflow，python -m langflow run
然后访问http://127.0.0.1:7860，会弹出一个可视化编辑界面。
![](/img/ai/langflow_overview.img)

### 4.2.2 运行大模型
ollama run llama3:8b
访问http://localhost:11434，会有输出Ollama is running

### 4.2.3 选择一个模板进行开发
点击new project，选择一个模板
![alt text](/img/ai/template.png)
我选择Vector Store RAG，跟我目前的需求比较匹配。
![](/img/ai/flow_demo.png)
具体修改点：
1. 将模型换成ollama，找到左边组件中model，替换成本地我们下载的模型。如果用其他模型，都需要api_key，大部分都是要收钱。
2. 向量数据库使用ChromaDB，chromadb是本地存储。
3. embedding选择ollama对应的值

### 4.2.4 构建
从图中，可以看到2条路径，一条是构建向量数据库，一条是运行应用。
我们先构建向量数据库数据，点击Chroma DB的运行按钮，就能构建。
我选择了一个零一万物福利手册pdf作为文件录入数据库。点击运行
![](/img/ai/build_flow.png)
然后点击Retriever，可以看到建模的数据。
![](/img/ai/retriver.png)


### 4.2.5 对比效果
点击playground，可以在线测试效果。
![](/img/ai/test_flow.png)

我们看下不用RAG的大模型输出。
输入是零一万物有哪些福利？
```txt
零一万物 (Zhēng Yī Wàn Wù) is a Chinese term that literally means 
"Zero to Ten Thousand Things" or "All Under Heaven". It refers to the vast 
and diverse universe, encompassing all existence, phenomena, and beings. 
In this context, "福利" (fèi lì) means benefits or advantages.

So, when we talk about the welfare of zero to ten thousand things, it's a 
philosophical and cosmological concept that considers the well-being and 
flourishing of all existence within the vast expanse of the universe. Here 
are some potential benefits:

1. **Interconnectedness**: Recognizing the intricate web of relationships 
between all phenomena, fostering harmony and cooperation.
2. **Dynamic Balance**: Understanding the ever-changing nature of the 
universe, allowing for adaptation and resilience in the face of 
uncertainty.
3. **Compassion and Empathy**: Cultivating a sense of shared humanity (and 
non-humanity) across the vast expanse of existence, promoting kindness and 
understanding.
4. **Cyclical Renewal**: Embracing the cycles of birth, growth, decay, and 
rebirth, acknowledging the eternal nature of change and transformation.
5. **Holistic Awareness**: Integrating the wisdom of various disciplines 
(e.g., science, philosophy, spirituality) to grasp the complexity and 
beauty of existence.
6. **Non-dualism**: Recognizing that distinctions between subject-object, 
self-other, and being-non-being are ultimately illusory, promoting a 
deeper sense of unity and wholeness.
7. **Evolving Intelligence**: Honoring the evolutionary nature of 
intelligence, acknowledging the potential for growth and development 
within all beings.

These benefits can guide our understanding of the universe, inspiring us 
to live in harmony with the natural world, appreciate the intricate web of 
relationships, and cultivate a sense of compassion and connection with all 
existence. 🌊💖
```

让大模型用中文回答一遍，结果是这样的：
```txt
零一万物（Zhēng Yī Wàn Wù）的福利指的是宇宙中所有事物的福利，这是一个哲
学和宇宙学概念，考虑宇宙中所有存在的福祉和 процветание。以下是可
能的福利：

1. **互连关系**：认清宇宙中所有事物之间的 intricate 网络关系，培养和谐和合作
。
2. **动态平衡**：理解宇宙的变化性质，使其在不确定的情况下适应和恢复。
3. **同情心和同体共感**：培养对所有存在的同情心和同体共感，促进慷慨和理解。

4. **循环更新**：承认宇宙中的生、长、衰、更新的循环，表彰变化和更新的永恒性
质。
5. **holistic Awareness**：整合各种学科（例如科学、哲学、精神）来理解宇宙的
复杂和美丽。
6. **非二元论**：认清主题-客体、自我他人的区别和有-无之间的 illusion，促进更
深入的 unity 和整体感。
7. **演化智慧**：表彰智能的演化性质，使其在所有存在中增长和发展。

这些福利可以指导我们对宇宙的理解，鼓励我们与自然世界和谐，欣赏事物之间的 
intricate 网络关系，并培养同情心和连接所有存在。 🌊💖
```

然后我们用langflow的playground，运行我们的查询，得到的结果：
```
根据提供的文本内容，零一万物（可能指的是某种保险或医疗服务）的福利包括：
原发性血小板增多症
自身免疫性溶血性贫血
骨髓异常增生综合症
慢性胃炎
克罗恩病
重症肌无力
慢性乙型肝炎、慢性丙型肝炎、慢性丁型肝炎
肝硬化
慢性肾炎、慢性肾功能不全（非透析治疗）
系统性红斑狼疮
慢性再生障碍性贫血
颅内良性肿瘤
骨髓纤维化
这些福利可能与某种保险或医疗服务有关，具体内容需要根据上下文进行理解。
当然结果不算很正确，但是至少靠近了。
```
说明RAG的作用还是有的，剩下就是更好的构建向量数据库，挑选合适的embedding模型，并且搭配合理的Prompt。


### 4.2.5 发布api
langflow支持api调用，点击api可以看到对应的api调用方式模板。
![](/img/ai/flow_api.png)
当调试好应用后，就可以通过这个api发布出去，可以通过http请求调用。

## 五、参考资料

1. [动手学大模型应用开发](https://datawhalechina.github.io/llm-universe/#/./C1/1.%E5%A4%A7%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%20LLM%20%E7%90%86%E8%AE%BA%E7%AE%80%E4%BB%8B)
2. [langchain](https://www.langchain.com/)
3. [langflow](https://www.langflow.org/)
4. [langchain+ollama构建本地大模型应用](https://gitcode.csdn.net/66c6d3551016441633659a8f.html)