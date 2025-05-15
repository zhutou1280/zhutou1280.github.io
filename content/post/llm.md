+++
title = 'Llm'
date = 2025-01-08T11:17:50+08:00
draft = true
+++

## 一些问题

1. 什么是 embeddings
2. 神经网络模型是什么
3. embedding 的类型有哪些？
4. RNN 是什么，可以做什么？
5. Transformer 构型
6. BERT 构型
7. 什么是基础模型，pre-training
8. Fine-tuning 和 post-training 是什么意思
9. 如何让模型认识文本

## tokens and embeddings

1. llm 模型处理的一块文本称之为 token
2. 将 token 转成数字化的表达，叫 embeddings

## LLM Tokenization

我们目前接触的大模型用法，都是在一个聊天对话框进行交互，并且模型不是立即生成回答，而是一个 token，一个 token 的出来。

但是 token 不是仅仅只用于输出，模型的输入也是 token。

### 分词器如何为模型准备输入

在 prompt 给模型时，首先会被分词器分成多个部分，

每一个 token 都是一个数字，这个数字存在一个表中，这个分词器中的表，代表他所知的所有 token。

token 不完全是一个单次一个 token，有些 token 是代表文本开始，有些 token 是整个单词，有些 token 是单词的一部分。

### tokenizer 如何拆分文本

有三个因素影响拆分文本

1. 在模型设计阶段，模型创建者选择一个分词方法。
   常见的是 BEP（byte pair encoding），广泛被 GTP 模型使用。
   WordPiece 被 BERT 使用。
2. 选择了分词方法后，需要对参数做一些选择，比如词汇大小，如何表征特殊 token。
3. 分词器需要基于一个特殊的数据集被训练，从而达到最好的词汇来表征特征集。英文文本数据集和代码数据集或者多种语言数据集都是有区别的。

除了处理语言模型的输入，还会生成对应的输出，将 token ID 转换成对应的文字。

### 单词、部分单词、字符、字节 token

- Word tokens
  word2vec 现在在 NLP 中用的越来越少。但是在 NLP 领域之外，比如推荐系统，用的比较多。
  缺点是，tokenizer 可能无法处理分词器训练后进入数据集的新单词。也会产生包含大量差异最小的标记的词汇表。

- subWord tokens
  这个包含全部和部分单词，能解决 word token 中的问题。

- Character Tokens
  这个对于训练模型更难，比如说 play 这个单词，subWord 方法一个 token 就够，但是字符需要 4 个 token，并且需要对这 4 个 token 进行建模，才能表征这个含义。

- Byte Tokens
  这个是用字节

### Galactica

科学大模型

## 模型生成机制

模型一次生成一个 token，一个 token 的输入会追加到下一次 token 生成。这种方式是自回归模型（autoRegressive model）
BERT 文本表现模型不是自回归的。

## Transformer 构型

transformer 构型的 LLM 是由 tokennizer、a stack of transformer blocks、a language modeling head 组成。

## decode strategy

从概率分布中选择单个 token 的方法称为解码策略。

每次选择最高得分的 token，称为 greedy decoding。 当你设置 temperature parameter 为 0 时，就是这个策略。

## prompt 工程

1. prompt 优化方式

- 提问要具体
- 减少幻觉产生
- 顺序很重要。长对话内容，中间的部分往往可能被忽略。开头和结尾往往影响很大。

2. 更复杂的 prompt 手段

- 人员角色（指明 LLM 是什么角色的，比如你是一个宇宙学专家）
- 指令（即 llm 需要做的事情）
- 上下文。 对任务描述的额外补充信息
- 输出格式，如果不指定，llm 会按他自己的格式输出
- 受众
- 预期
- 数据
