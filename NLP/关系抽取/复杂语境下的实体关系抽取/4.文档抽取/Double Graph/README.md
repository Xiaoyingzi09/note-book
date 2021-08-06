# Double Graph Based Reasoning for Document-level Relation Extraction

# 基于双图的文档级关系抽取推理

## 摘要

文档级关系提取旨在提取文档内实体之间的关系。 与句子级关系提取不同，它需要对文档中的多个句子进行推理。在本文中，我们提出了具有双图特征的图聚合和推理网络（GAIN）。  GAIN 首先构建一个异构的提及级别图 (hMG) 来模拟文档中不同提及之间的复杂交互。它还构建了一个实体级图（EG），在此基础上我们提出了一种新的路径推理机制来推断实体之间的关系。 在公共数据集 DocRED 上的实验表明，与之前的最新技术相比，GAIN 实现了显着的性能提升（F1 上的 2.85）。 我们的代码可从 https://github.com/DreamInvoker/GAIN 获得。

## 1 前言

从文本中识别实体之间的语义关系的任务，即关系提取 (RE)，在各种基于知识的应用中起着至关重要的作用，例如问答 (Yu et al., 2017) 和大规模知识图谱构建 . 以前的方法（Zeng et al., 2014; Zeng et al., 2015; Xiao and Liu, 2016; Zhang et al., 2017; Zhang et al., 2018; Baldini Soares et al., 2019）侧重于句子级别 RE，在单个句子中预测实体之间的关系。 然而，句子级 RE 模型有一个不可避免的局限性——它们无法识别句子间实体之间的关系。 因此，在文档级别提取关系对于全面理解文本知识是必要的。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/figure_1.png)

图 1：来自 DocRED 的示例文档及其所需的关系（Yao 等，2019）。 这些关系实例中涉及的实体提及和关系是彩色的。 为清楚起见，其他提及的内容都标有下划线。

在文档级别进行有效的关系提取有几个主要挑战。 首先，关系中涉及的主宾实体可能出现在不同的句子中。 因此，不能仅基于单个句子来识别关系。 其次，同一实体可能在不同的句子中被多次提及。 必须聚合跨句上下文信息以更好地表示实体。 第三，许多关系的识别需要逻辑推理技术。 这意味着只有当其他实体和关系（通常分布在句子中）被隐式或显式识别时，才能成功提取这些关系。 如图1所示，很容易识别句内关系（马里兰州，国家，美国），（巴尔的摩，位于行政领土实体，马里兰州），和（埃尔德斯堡，位于行政领土实体，马里兰州）， 因为主语和宾语出现在同一个句子中。 然而，预测巴尔的摩和美国之间以及埃尔德斯堡和美国之间的句间关系并非易事，它们的提及不会出现在同一个句子中并且具有长距离依赖关系。 此外，这两个关系实例的识别也需要逻辑推理。 例如，埃尔德斯堡属于美国，因为埃尔德斯堡位于属于美国的马里兰州。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/table_1.png)

表 1：从用于 BiLSTM 的 DocRED 开发集（Yao et al., 2019）随机抽样的 100 个文档中的坏案例统计，总共有 1150 个坏案例。

最近，姚等人。  (2019) 提出了一个大规模人工注释的文档级 RE 数据集 DocRED，将句子级 RE 推进到文档级，它包含大量的关系事实。 图 1 显示了来自 DocRED 的示例。我们从 DocRED 开发集中随机抽取 100 个文档，并手动分析由 Yao 等人提出的基于 BiLSTM 的模型预测的坏案例。  (2019)。 如表 1 所示，句间错误类型和逻辑推理错误类型在所有坏案例中占很大比例，分别为 53.5% 和 21.0%。 因此，在本文中，我们旨在解决这些问题，以更好地从文档中提取关系。最近，姚等人。  (2019) 提出了一个大规模人工注释的文档级 RE 数据集 DocRED，将句子级 RE 推进到文档级，它包含大量的关系事实。 图 1 显示了来自 DocRED 的示例。我们从 DocRED 开发集中随机抽取 100 个文档，并手动分析由 Yao 等人提出的基于 BiLSTM 的模型预测的坏案例。  (2019)。 如表 1 所示，句间错误类型和逻辑推理错误类型在所有坏案例中占很大比例，分别为 53.5% 和 21.0%。 因此，在本文中，我们旨在解决这些问题，以更好地从文档中提取关系。

以前在文档级 RE 中的工作不考虑推理（Gupta 等人，2019 年；Jia 等人，2019 年；Yao 等人，2019 年），或仅使用基于图的或分层神经网络在隐式中进行推理 (Peng et al., 2017; Sahu et al., 2019; Nan et al., 2020)。 在本文中，我们提出了一种用于文档级关系提取的图聚合和推理网络（GAIN）。 它旨在直接解决上述挑战。  GAIN 构建了一个异构的提及级图（hMG），具有两种类型的节点，即提及节点和文档节点，以及三种不同类型的边，即实体内边、实体间边和文档边，以捕获上下文 文档中实体的信息。 然后，我们在 hMG 上应用 Graph Convolutional Network（Kipf 和 Welling，2017）以获得每个提及的文档感知表示。然后通过合并引用 hMG 中相同实体的提及来构建实体级图（EG），在此基础上我们提出了一种新的路径推理机制。 这种推理机制允许我们的模型推断实体之间的多跳关系。

总之，我们的主要贡献如下：

- 我们提出了一种新方法，图聚合和推理网络（GAIN），它具有双图设计，以更好地应对文档级 RE 任务。

- 我们引入了一个异构的提及级图 (hMG) 和基于图的神经网络，以模拟文档中不同提及之间的交互并提供文档感知提及表示。

- 我们引入了实体级图 (EG) 并提出了一种新的路径推理机制，用于实体之间的关系推理。

我们在公共 DocRED 数据集上评估 GAIN。 它以 2.85 F1 分数显着优于之前最先进的模型。 进一步的分析证明了 GAIN 能够聚合文档感知上下文信息并推断文档的逻辑关系。

## 2 任务制定

我们将文档级关系提取任务制定如下。给定一个由 N 个句子组成的文档 D = {si}N i=1 和各种实体 E = {ei}P i=1，其中 si = {wj}M j=1 指的是第 i 个句子组成的 M 个词，ei = {mj}Q j=1 并且 mj 指的是属于第 i 个实体的第 j 个提及的词的跨度，任务旨在提取 E 中不同实体之间的关系，即 {(  ei, rij, ej)|ei, ej ∈ E, rij ∈ R}，其中 R 是预定义的关系类型集。

在我们的论文中，实体 ei 和 ej 之间的关系 rij 被定义为句间关系，当且仅当 Sei ∩ Sej = ∅，其中 Sei 表示那些包含 ei 的句子。 相反，关系 rij 被定义为句内，当且仅当 Sei∩Sej ̸= ∅。 我们还将 K-hop 关系推理定义为基于现有关系的 K 长度链预测关系 rij，其中 ei 和 ej 是推理链的头和尾，即 ei r1 −→ em r2 −→ 。  .  .  en rK −→ ej ⇒ ei rij −→ ej。

## 3 图聚合和推理网络 (GAIN)

GAIN主要由4个模块组成：编码模块（3.1节）、提及级图聚合模块（3.2节）、实体级图推理模块（3.3节）、分类模块（3.4节），如图 在图 2 中。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/figure_2.png)

图 2：GAIN 的整体架构。 首先，上下文编码器使用输入文档来获取每个单词的上下文化表示。 然后，使用提及节点和文档节点构建提及级图。 应用 GCN 后，图被转换为实体级图，其中实体之间的路径被识别以进行推理。 最后，分类模块根据上述信息预测目标关系。 不同的实体有不同的颜色。 提及节点中的数字 i 表示它属于第 i 个句子。

#### 3.1 编码模型

在编码模块中，我们将包含 n 个单词的文档 D = {wi}n i=1 转换为向量序列 {gi}n i=1。 继姚等人。  (2019)，对于 D 中的每个词 wi，我们首先将其词嵌入与实体类型嵌入和共指嵌入连接起来：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_1.png)

其中Ew(·)、Et(·)和Ec(·)分别表示词嵌入层、实体类型嵌入层和共指嵌入层。  ti 和 ci 被命名为实体类型和实体 id。 我们为那些不属于任何实体的词引入 None 实体类型和 id。

然后将向量化的词表示输入编码器以获得每个词的上下文敏感表示：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_2.png)

其中编码器可以是 LSTM 或其他模型。

#### 3.2 提及级图聚合模块

为了对提及和实体之间的文档级信息和交互进行建模，构建了一个异构的提及级图 (hMG)。

hMG 有两种不同的节点：提及节点和文档节点。 每个提及节点表示一个实体的一个特定提及。 并且 hMG 还有一个文档节点，旨在对整体文档信息进行建模。 我们认为该节点可以作为与不同提及项交互的枢轴，从而减少文档中它们之间的长距离。

hMG 中存在三种类型的边：

- 实体内边缘：提及同一实体的提及与实体内边缘完全连接。通过这种方式，可以对同一实体的不同提及项之间的交互进行建模。
- 实体间边缘：如果它们同时出现在单个句子中，则对不同实体的两次提及与实体间边缘相关联。 通过这种方式，实体之间的交互可以通过它们的提及的同时出现来建模。
- 文档边缘：所有提及都通过文档边缘连接到文档节点。通过这种连接，文档节点可以处理所有提及并启用文档和提及之间的交互。此外，两个提及节点之间的距离最多为两个，以文档节点为枢轴。 因此可以更好地建模长距离依赖。

接下来，我们在 hMG 上应用图卷积网络（Kipf 和 Welling，2017 年）来聚合来自邻居的特征。 给定第 l 层的节点 u，图卷积操作可以定义为：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_3.png)

其中 K 是不同类型的边，W (l) k ∈ Rd×d 和 b(l) k ∈ Rd 是可训练的参数。  Nk(u) 表示在第 k 条边上连接的节点 u 的邻居。  σ 是一个激活函数（例如，ReLU）.

GCN的不同层表达不同抽象层次的特征，因此为了覆盖所有层次的特征，我们将每一层的隐藏状态串联起来，形成节点u的最终表示：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_4.png)

其中 h(0) u 是节点 u 的初始表示。对于文档中从第 s 个单词到第 t 个单词的提及，h(0) u = 1 t−s+1 �tj=s gj 并且对于文档节点，它使用文档表示输出进行初始化 来自编码模块。

#### 3.3 实体级图推理模块

在本小节中，我们将介绍实体级图 (EG) 和路径推理机制。 首先，将引用相同实体的提及合并到实体节点，以获得EG中的节点。 请注意，我们不考虑 EG 中的文档节点。 对于第 i 个实体节点 ei 被提及 N 次，它由其 N 次提及表示的平均值表示：

成节点u的最终表示：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_5.png)

然后，我们合并所有连接相同两个实体的提及项的实体间边，以获得 EG 中的边。  EG中从ei到ej的有向边的表示定义为：

成节点u的最终表示：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_6.png)

其中 Wq 和 bq 是可训练参数，σ 是激活函数（例如 ReLU）。

基于向量化的边表示，头实体eh和尾实体et之间经过实体eo的第i条路径表示为：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_7.png)

请注意，我们这里只考虑两跳路径，而它可以很容易地扩展到多跳路径。

我们还引入了注意力机制 (Bahdanau et al., 2015)，使用实体对 (eh, et) 作为查询，融合 eh 和 et 之间不同路径的信息。

基于向量化的边表示，头实体eh和尾实体et之间经过实体eo的第i条路径表示为：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_8.png)

其中 αi 是第 i 条路径的归一化注意力权重。 因此，该模型将更加关注有用的路径。  σ 是一个激活函数。

有了这个模块，一个实体可以通过从它的提及中融合信息来表示，这些信息通常分布在多个句子中。 此外，潜在的推理线索由实体之间的不同路径建模。 然后它们可以与注意力机制集成，以便我们将潜在的逻辑推理链考虑在内来预测关系。

#### 3.4 分类模块

对于每个实体对（eh，et），我们连接以下表示：（1）在实体级图中派生的头尾实体表示eh和et，通过比较操作（Mou et al., 2016）来加强 特征，即两个实体的表示之间减法的绝对值，|eh − et|，和元素相乘，eh ⊙ et；  (2) 文档节点在 Mention-level Graph 中的表示，mdoc，因为它可以帮助聚合跨句信息并提供文档感知表示；  (3)综合推理路径信息ph,t。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_9.png)

最后，我们将任务制定为多标签分类任务并预测实体之间的关系：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_10.png)

其中 Wa、Wb、ba、bb 是可训练参数，σ 是激活函数（例如 ReLU）。 我们使用二元交叉熵作为分类损失以端到端的方式训练我们的模型：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/formula_11.png)

其中S表示整个语料库，I(·)表示指示函数。

## 4 实验

#### 4.1 数据集

我们在 DocRED (Yao et al., 2019) 上评估我们的模型，DocRED 是从维基百科和维基数据构建的用于文档级 RE 的大规模人工注释数据集。  DocRED 共有 96 种关系类型，132、275 个实体和 56、354 个关系事实。  DocRED 中的文档平均包含约 8 个句子，40.7% 以上的关系事实只能从多个句子中提取。此外，61.1% 的关系实例需要各种推理技能，例如逻辑推理（Yao 等，2019）。 我们遵循数据集的标准拆分，3, 053 个文档用于训练，1, 000 个用于开发，1, 000 个用于测试。 有关 DocRED 的更详细统计数据，我们建议读者参考原始论文（Yao 等，2019）。

#### 4.2 实验设置

在我们的 GAIN 实现中，我们使用了 2 层 GCN，并将丢弃率设置为 0.6，学习率设置为 0.001。 我们使用 AdamW（Loshchilov 和 Hutter，2019）作为权重衰减为 0.0001 的优化器训练 GAIN，并在 PyTorch（Paszke 等，2017）和 DGL（Wang 等，2019b）下实现 GAIN。

我们为 GAIN 实现了三个设置。GAIN-GloVe 使用 GloVe (100d) 和 BiLSTM (256d) 作为词嵌入和编码器。  GAINBERTbase 和 GAIN-BERTlarge 分别使用 BERTbase 和 BERTlarge 作为编码器，学习率设置为 1e−5。

#### 4.3 Baseline和评估矩阵

我们使用以下模型作为基线。

姚等人。  (2019) 提出的模型使用 CNN（Fukushima，1980）、LSTM（Hochreiter 和 Schmidhuber，1997）和 BiLSTM（Schuster 和 Paliwal，1997）作为它们将文档编码为隐藏状态向量序列 {hi}ni=1 编码器，并用它们的表示预测实体之间的关系。 其他预训练模型，如 BERT (Devlin et al., 2019)、RoBERTa (Liu et al., 2019) 和 CorefBERT (Ye et al., 2020) 也用作编码器 (Wang et al., 2019a; Ye  et al., 2020) 到文档级 RE 任务。

上下文感知，也由 Yao 等人提出。 (2019) on DocRED 改编自 (Sorokin and Gurevych, 2017)，使用 LSTM 对文本进行编码，但进一步利用注意力机制吸收上下文关系信息进行预测。

BERT-Two-Stepbase，由 Wang 等人提出。(2019a) 在 DocRED 上。 虽然类似于BERTREbase，但它首先预测两个实体是否有关系，然后预测具体的目标关系。

Tang 等人提出的 HIN-GloVe/HIN-BERTbase。  (2020)。分层推理网络 (HIN) 聚合来自实体级、句子级和文档级的信息以预测目标关系，并使用 GloVe (Pennington et al., 2014) 或 BERTbase 进行词嵌入。

LSR-GloVe/LSR-BERTbase，由 Nan 等人提出。  (2020) 最近。 他们基于依赖树构建图，并通过潜在结构归纳和 GCN 预测关系。 南等人。  (2020) 还将四种基于图的最先进的 RE 模型应用于 DocRED，包括 GAT (Velickovic et al., 2017)、GCNN (Sahu et al., 2019)、EoG (Christopoulou et al., 2019)  ) 和 AGGCN (Guo et al., 2019)。 我们还包括他们的结果。

继姚等人。  (2019)，我们在实验中使用了广泛使用的指标 F1 和 AUC。 我们还使用 Ign F1 和 Ign AUC，它们计算 F1 和 AUC，不包括训练和开发/测试集中的共同关系事实。

#### 4.4 结果

与其他基线相比，我们在表 2 中的 DocRED 数据集上展示了 GAIN 的性能。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/table_2.png)

表 2：DocRED 上的性能。 第一双线以上的模型不使用预训练模型。 带 * 的结果报告在他们的原始论文中。  ‡ 的结果是 (Nan et al., 2020) 中实现的基于图的最先进 RE 模型的性能。 带有 † 的结果基于我们的实施。

在不使用 BERT 或 BERT 变体的模型中，GAIN-GloVe 在测试集上始终优于所有基于序列和基于图的强基线 0.9 ∼ 12.82 F1 分数。 在使用 BERT 或 BERT 变体的模型中，与强基线 LSR-BERTbase 相比，GAINBERTbase 在开发和测试集上的 F1/Ign F1 分别提高了 2.22/6.71 和 2.19/2.03。 与之前最先进的方法 CorefRoBERTaRElarge 相比，GAIN-BERTlarge 还在测试集上提高了 2.85/2.63 F1/Ign F1。 这表明 GAIN 在文档级 RE 任务中更有效。 我们还可以观察到，LSR-BERTbase 在开发和测试集上将 F1 提高了 3.83 和 4.87，其中 GloVe 嵌入替换为 BERTbase。 相比之下，我们的 GAINBERTbase 提高了 5.93 和 6.16，这表明 GAIN 可以更好地利用 BERT 表示。

#### 4.5 消融研究

为了进一步分析 GAIN，我们还进行了消融研究，以说明 GAIN 中不同模块和机制的有效性。 我们在表 3 中显示了消融研究的结果。

首先，我们移除 GAIN 的异构 Mentionlevel Graph (hMG)。 详细地，我们使用方程初始化实体级图（EG）中的实体节点。  5 但将 mn 替换为 h(0) n ，并将 GCN 应用于 EG。 将 GCN 不同层中的特征连接起来以获得 ei。 在没有 hMG 的情况下，GAIN-GloVe/GAIN-BERTbase 的性能在开发集上急剧下降了 2.08/2.02 Ign F1 分数。 这一下降表明 hMG 在捕获属于相同和不同实体和文档感知特征的提及之间的交互方面起着至关重要的作用。

接下来，我们移除推理模块。 具体来说，该模型摒弃了Entitylevel Graph中得到的头尾实体ph,t之间的路径信息，仅基于实体表示eh和et和文档节点表示mdoc预测关系。 推理模块的移除导致所有指标的性能不佳，例如，GAIN-GloVe/GAIN-BERTbase 的开发集上的 Ign F1 分数降低了 2.21/2.17。这表明我们的路径推理机制有助于捕获潜在的 K 跳推理路径以推断关系，从而提高文档级 RE 性能。

此外，去掉 hMG 中的文档节点会导致 GAIN-GloVe/GAIN-BERTbase 的开发集上的 Ign F1 减少 2.19/1.88。 它帮助 GAIN 聚合文档信息，并作为一个枢轴来促进不同提及之间的信息交换，尤其是文档中相距较远的那些。

#### 4.6 分析和讨论

在本小节中，我们将进一步分析开发集上的句间和推理性能。 与 Nan 等人相同。  (2020)，我们在表 4 中报告了 Intra-F1/Inter-F1 分数，它们分别只考虑了句内或句间关系。 同样，为了评估模型的推理能力，表 5 报告了 Infer-F1 分数，该分数仅考虑参与关系推理过程的关系。 例如，我们在计算 Infer-F1 时考虑黄金关系事实 r1、r2 和 r3，如果存在 eh r1 −→ eo r2 −→ et 和 eh r3 −→ et。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/table_3.png)

表 3：具有不同嵌入和子模块的 GAIN 性能。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/table_4.png)

表 4：在 DocRED 开发集上的 Intra-F1 和 Inter-F1 结果。 带 * 的结果报告在 (Nan et al., 2020) 中。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/table_5.png)

表 5：在 DocRED 开发集上的 Infer-F1 结果。  P：精度，R：召回。

如表 4 所示，GAIN 不仅在 Intra-F1 中而且在 Inter-F1 中都优于其他基线，并且去除 hMG 导致 Inter-F1 比 Intra-F1 下降更显着，这表明我们的 hMG 确实有助于提及之间的交互 ，尤其是分布在不同句子中的远距离依赖。

此外，表 5 表明 GAIN 可以更好地处理关系推理。例如，与 RoBERTa-REbase 相比，GAINBERTbase 改进了 5.11 Infer-F1。推理模块在捕获实体之间的潜在推理链方面也发挥着重要作用，否则 GAINBERTbase 将下降 1.78 Infer-F1。

#### 4.7 案例分析

图 3 还显示了我们提出的模型 GAIN 与其他基线相比的案例研究。如图所示，BiLSTM 只能识别第一句话中的两个关系。  BERT-REbase 和 GAIN-BERTbase 都可以成功预测《Without Me》是 The Eminem Show 的一部分。 但只有 GAIN-BERTbase 能够推断出《Without Me》的表演者和出版日期与 The Eminem Show 相同，即 Eminem 和 2002 年 5 月 26 日，需要跨句子进行逻辑推理。

## 5 相关工作

以前的方法侧重于句子级关系提取（Zeng et al., 2014; Zeng et al., 2015; Wang et al., 2016; Zhou et al., 2016; Xiao and Liu, 2016; Zhang et al., 2017  ；冯等人，2018 年；朱等人，2019 年）。 但是句子级 RE 模型在实践中面临不可避免的限制，其中许多现实世界的关系事实只能跨句子提取。 因此，许多研究人员逐渐将注意力转移到文档级关系提取上。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/b35d161a9a6df59dc34edc8b878522fbea77f5f2/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/4.%E6%96%87%E6%A1%A3%E6%8A%BD%E5%8F%96/Double%20Graph/figure/figure_3.png)

图 3：我们提出的 GAIN 和基线模型的案例研究。 模型以文档为输入，以不同颜色预测不同实体之间的关系。 由于篇幅限制，我们只展示了文档中的部分实体和相应的句子。

几种方法（Quirk 和 Poon，2017；Peng 等人，2017 年；Gupta 等人，2019 年；Song 等人，2018 年；Jia 等人，2019 年）利用依赖图来更好地捕获特定于文档的特征，但它们 忽略文档中无处不在的关系推理。 最近，提出了许多模型来解决这个问题。 唐等人。  (2020) 通过考虑来自实体级、句子级和文档级的信息，提出了一种分层推理网络。 然而，它基于层次网络隐式地进行关系推理，而我们采用路径推理机制，这是一种更显式的方式。

(Christopoulou et al., 2019) 是最近文档级 RE 任务中最强大的系统之一。 与 (Christopoulou et al., 2019) 和其他基于图的关系提取方法相比，我们的架构具有许多不同的设计，背后有着不同的动机。首先，图构建的方式不同。 我们创建了两个不同级别的独立图来分别捕获长距离文档感知交互和实体路径推理信息。 而 Christopoulou 等人。  (2019) 将提及和实体放在同一个图中。 此外，他们没有像 GCN 那样进行图节点表示学习来聚合构建图上的交互信息，仅使用 BiLSTM 的特征来表示节点。其次，路径推理的过程不同。 克里斯托普卢等人。  (2019) 使用基于步行的方法为每个实体对迭代生成路径，这需要额外的超参数调整开销来控制推理过程。 相反，我们使用一种注意力机制来选择性地融合实体对的所有可能路径信息，同时没有额外的开销。

当我们写这篇论文时，（Nan 等人，2020 年）将他们的工作作为预印本公开，它采用依赖树来捕获文档中的语义信息。 他们将提及节点和实体节点放在同一个图中，并通过使用 GCN 进行隐式推理。 与他们的工作不同，我们的 GAIN 在不同的图中呈现了提及节点和实体节点，以更好地进行句间信息聚合并更明确地推断关系。

其他一些尝试（Verga 等人，2018 年；Sahu 等人，2019 年；Christopoulou 等人，2019 年）研究特定领域中的文档级 RE，如生物医学 RE。 然而，他们使用的数据集通常包含非常有限的关系类型和实体类型。 例如，CDR (Li et al., 2016) 只有一种类型的关系和两种类型的实体，这可能不是关系推理的理想测试平台。

## 6 总结

提取句间关系和进行关系推理在文档级关系提取中具有挑战性。

在本文中，我们引入了图聚合和推理网络（GAIN）来更好地应对文档级关系提取，它具有不同粒度的双图。GAIN 利用异构的提及级图来模拟文档中不同提及之间的交互并捕获文档感知特征。 它还使用具有建议路径推理机制的实体级图来更明确地推断关系。

在大规模人工标注数据集 DocRED 上的实验结果表明，GAIN 优于以前的方法，尤其是在句子和推理关系场景中。 消融研究还证实了我们模型中不同模块的有效性。















