# 最小化联合实体和关系提取中 Seq2Seq 模型的暴露偏差

# Minimize Exposure Bias of Seq2Seq Models in Joint Entity and Relation
Extraction

## 摘要

联合实体和关系提取旨在直接从纯文本中提取关系三元组。先前的工作利用序列到序列 (Seq2Seq) 模型来生成三元组序列。 然而，Seq2Seq 对无序三元组强制执行不必要的顺序，并且涉及与错误累积相关的大解码长度。 这些方法引入了曝光偏差，这可能会导致模型过度拟合频繁的标签组合，从而限制了泛化能力。我们提出了一种新的 Sequence-to-UnorderedMulti-Tree (Seq2UMTree) 模型，通过将三元组中的解码长度限制为三个并去除三元组之间的顺序来最小化曝光偏差的影响。 我们在两个数据集 DuIE 和 NYT 上评估我们的模型，并系统地研究曝光偏差如何改变 Seq2Seq 模型的性能。 实验表明，最先进的 Seq2Seq 模型对两个数据集都过拟合，而 Seq2UMTree 显示出明显更好的泛化。 我们的代码可在 https://github.com/WindChimeRan/OpenJERE 获得。

## 1 介绍

关系提取旨在从纯文本中提取实体-关系三元组 (h, r, t)。 例如，在三元组（Obama，毕业于哥伦比亚大学）中，Obama 和 Columbia University 是出现在文本中的头尾实体，而 graduate from 是这两个实体之间的关系。

对于有监督的关系抽取，早期的研究集中在管道方法上，它使用实体抽取器来抽取实体，然后对实体对的关系进行分类。 这些方法忽略了这两个子任务之间的内在交互，并通过任务传播分类错误。联合实体和关系提取（JERE）考虑子任务交互（Roth 和 Yih，2004 年；Ji 和 Grishman，2005 年； 姬等人，2005； 于和林，2010 年； 里德尔等人，2010 年；  Sil 和 Yates，2013 年； 李等人，2014 年； 李和姬，2014；  Durrett 和 Klein，2014 年；  Miwa 和 Sasaki，2014 年； 卢和罗斯，2015； 杨和米切尔，2016 年；  Kirschnick 等人，2016 年； 米瓦和班萨尔，2016 年； 古普塔等人，2016 年；  Katiyar 和 Cardie，2017），但他们主要利用基于特征的系统或多任务神经网络，无法捕获三元组间的依赖关系。NovelTagging (Zheng et al., 2017) 将这两个子任务集成到一个序列标记过程中，该过程为每个标记分配一个实体关系标签； 当一个token属于多个关系时，预测结果是不完整的。  Sequence-toSequence (Seq2Seq) 模型（Cho 等人，2014 年）能够多次提取一个实体而不是序列标记，因此可以将多个关系分配给一个实体，这自然地解决了这个问题（Zeng 等人，  2018, 2019a,b；Nayak 和 Ng，2019)。具体来说，所有现有的 Seq2Seq 模型都预先定义了目标三元组的顺序，例如 三元组字母顺序，然后根据顺序自回归解码三元组序列，这意味着当前的三元组预测依赖于之前的输出。 例如，在图 1 中，三元组列表被扁平化为[Obama]-[graduate from]-[Columbia University]-[Obama]-[graduate from]-[Harvard Law School]...

然而，Seq2Seq 模型的自回归解码引入了暴露偏差问题，这可能会严重降低性能。 曝光偏差是指解码过程的训练和测试阶段之间的差异（Ranzato 等，2015）。 在训练阶段，当前的三元组预测依赖于之前三元组的黄金标准标签，而在测试阶段，当前的三元组预测依赖于之前三元组的模型预测，这可以与黄金标准不同 标签。 因此，在测试阶段，偏斜的预测会进一步偏离后续三元组的预测； 如果解码长度很大，与金标准标签的差异将进一步累积。 这种累积的差异可能会降低性能，尤其是在预测更长的序列时，即多三元组预测.

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/figure_1.png)

图 1：针对不同三元组顺序的 Seq2Seq 和 Seq2UMTree 的训练和测试。

此外，由于 Seq2Seq 模型顺序预测三元组，因此它对无序标签强制执行了不必要的顺序，而其他三元组顺序也是正确的。 因此，分配的顺序使模型容易记住和过度拟合训练集中的频繁标签组合，并且对看不见的顺序的泛化能力很差。 过度拟合也是曝光偏差的副作用（Tsai 和 Lee，2019），这可能导致 Seq2Seq 预测中缺少三元组。 例如，在图1中，在训练阶段，Seq2Seq模型以这样的顺序学习triplet1-triplet2-triplet3，而triplet2-triplet1-triplet3的顺序也是正确的。 在测试阶段，Seq2Seq 模型首先根据指定的顺序预测triplet2，但是因为triplet2-triplet3 是模型的频繁顺序，所以它忽略了triplet1 并直接以triplet3 结尾（即triplet2-triplet3）。 当对模型强制执行命令时，模型会继续进行更多的学习约束。

为了在保持 Seq2Seq 简单性的同时减轻曝光偏差问题，我们将一维三元组序列重新转换为二维无序多树 (UMTree)，并提出了一种新的模型 Seq2UMTree。
   Seq2UMTree 模型基于 Encoder-Decoder 框架，由传统编码器和 UMTree 解码器组成。  UMTree 解码器使用具有无序多标签分类的复制机制作为输出层，对实体和关系进行联合和结构化建模。 这种多标签分类模型确保同一层中的节点是无序的，并丢弃预定义的三元组顺序，这样预测偏差就不会聚合并影响其他三元组。 与标准 Seq2Tree（Dong 和 Lapata，2016；Liu 等，2019）不同，解码长度限制为三个（一个三元组），这是 JERE 任务的最短可行长度。 通过这种方式，在三重态 F1 指标下，曝光偏差被最小化。

总之，我们的贡献如下：

- 我们指出了 Seq2Seq 模型的预定义三元组顺序的冗余，并提出了一种新颖的 Seq2UMTree 模型，通过将有序三元组序列重新转换为无序多树格式来最小化曝光偏差。 
-  我们系统地分析了暴露偏差如何降低标准 Seq2Seq 模型性能分数的可靠性。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/figure_2.png)

图 2：模型概述。 三元组内的解码顺序是 r, t, h。 关系是从预定义的关系字典中预测出来的，实体是从句子中复制出来的。

## 2 方法

Seq2UMTree 模型由一个传统的 Seq2Seq 编码器和一个 UMTree 解码器组成。  UMTree 解码器与标准解码器的不同之处在于它生成无序多标签输出并使用 UMTree 解码策略。 该模型的概述如图所示2.我们在以下小节中说明模型细节。

#### 2.1 模型

形式上，输入句子 x = [x0, x1, .  .  .  , xn] 首先通过词嵌入和双向循环神经网络 (Bi-RNN) (Schuster and Paliwal, 1997) 和长短期记忆 (LSTM) (Hochreiter and Schmidhuber, 1997) 转换为一系列上下文感知表示 编码器：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_1.png)

然后我们将输出的序列传递给 Conven：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_2.png)

其中 Conven 是编码器卷积层。Conven 将 sE 映射到 o0，它也是一个序列，与 s 序列具有相同的维度。输出表示为 o0 ∈ Rn×h，其中 h 是隐藏大小，n 是输入句子的长度。o0 是句子的辅助表示，用于使用 Scratchpad 注意力机制解码（Benmalek et al., 2019）：on−1 用于计算注意力得分，on−1 将在每个解码步骤更新为 on  .

在解码过程中，我们使用不同的输入嵌入和输出层进行关系和实体提取，并且它们共享相同的解码器参数。 对于输入嵌入 wt，我们使用： (a) “startof-the-sentence”嵌入：wsos 0 ∈ Rh，它总是解码的开始，被认为是深度 0，(b) 关系嵌入：wr t ∈  Rh, (c) 实体嵌入：we t = oe1 t−1 + oe2 t−1 ∈ Rh，其中 e1 和 e2 分别是预测实体的开始位置和结束位置。t ∈ {1, 2, 3}，即解码时间步长。 解码顺序可以任意预定义，例如h-r-t或t-r-h。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/figure_3.png)

图 3：Seq2UMTree 通过将树与序列对齐以教师强制方式进行训练。 在测试阶段，模型自回归解码整个树。 图中HLR、HLS、CU分别是Harvard Law Review、Harvard Law School和Columbia University的缩写。 该示例使用 h-r-t 作为三元组中的顺序。

给定输入嵌入 wt 和前一时间步长 sD t−1 的输出，使用一元 LSTM 解码器生成解码器隐藏状态：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_3.png)

其中 sD t 是解码器隐藏状态；  sD 0 由 sE n 初始化。

注意力机制（Luong et al., 2015）用于生成上下文感知嵌入：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_4.png)

其中 a ∈ Rh。 然后将上下文感知表示与原始 ot−1 连接起来，然后是卷积层：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_5.png)

其中 Convde 将维度 2h 映射到 h 并且 at 在串联之前复制 n 次。

关系预测的输出层是一个线性变换，然后是一个最大池化序列：其中 Convde 将维度 2h 映射到 h 并且 at 在串联之前复制 n 次。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_6.png)

其中σ是多关系分类的sigmoid函数，Wr∈Rh×r，br∈Rr，probr∈Rr是关系的预测概率向量。

实体预测的输出层是整个序列上的两个二元分类层，分别预测实体的开头和结尾的位置：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_7.png)

其中 We ∈ Rh×1，be 是标量，probe ∈ Rn×1 是实体的预测概率向量，eb 和 ee 指的是实体的开始和结束。 与 Nayak 和 Ng (2019) 不同，sigmoid 函数 σ 使模型能够一次预测多个实体。

#### 2.2 训练和测试

在训练阶段，对于每个句子，我们重新组织训练数据，UMTree 中每一对深度 1 和 2（例如 h-r）将形成一个训练示例，以便该策略遍历整棵树。 每个节点的训练过程对应于 Seq2Seq 模型中的一个时间步。 然后我们以教师强制（Williams and Zipser，1989）的方式训练模型：每个解码时间步的输入由黄金标准标签给出。 以阶 h-r-t 为例，在图 3a 中，总损失是以下三个解码步骤的损失之和：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/formula_8.png)

其中 h∗, r∗, t∗ 是三元组的基本事实，θ 是模型中的所有可训练参数。 在测试阶段，UMTree 使用自回归解码策略。 解码器逐层预测节点，其中前一层的预测结果分别作为下一个时间步的输入，如图3b所示。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/table_1.png)

表 1：NYT 和 DuIE 的主要结果。  #test 是测试集对模型的有效句子百分比。

## 3 实验

#### 3.1 设置

**数据集**

我们在两个数据集 NYT 和 DuIE1 上评估我们的模型。  NYT (Riedel et al., 2010) 是由远程监督生成的英文新闻数据集，无需人工标注，广泛应用于 JERE 研究中 (Zheng et al., 2017; Zeng et al., 2018; Takanobu et al., 2018)。  , 2018; Dai et al., 2019; Fu et al., 2019; Nayak and Ng, 2019; Zeng et al., 2019a,b; Chen et al., 2019; Wei et al., 2019)。 我们使用与 CopyRE 相同的数据拆分（Zeng 等，2018）。  DuIE (Li et al., 2019) 是一个大型中文 JERE 数据集，其中句子来自百度数据集=dureader News Feeds 和百度百科。 整个数据集由远程监督注释，然后手动检查。 我们随机取10%的训练集作为验证集，原始开发集作为测试集，因为原始测试集没有发布。 在预处理中，对于两个数据集，我们过滤掉不包含三元组的句子。这两个数据集的数据统计如表2所示。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/table_2.png)

**Baselines**

我们将提出的模型 Seq2UMTree 与相同超参数下的强基线进行比较，如下所示：1) CopyMTL (Zeng et al., 2019a) 是具有复制机制的 Seq2Seq 模型，实体通过多任务学习找到。  2) WDec (Nayak and Ng, 2019) 是一个标准的 Seq2Seq 模型，带有动态掩码，通过 token 对实体 token 进行解码。  3) MHS (Bekoulis et al., 2018) 是一个非 Seq2Seq 基线，它列举了所有可能的标记对。  4) Seq2UMTree 是提出的方法，它以简洁的树结构生成三元组。

**Hyperparameters（超参数）**
为了公平比较，我们使用相同的超参数设置自己重现了所有基线。 我们对英文使用 200 维词嵌入，对中文使用字符嵌入。 两者均从高斯分布 N(0, 1) 初始化，并且两者均使用 200 维 Bi-LSTM 编码器以减轻这两种语言的异质性。 这些模型由 Adam 优化器（Kingma 和 Ba，2014）训练 50 个 epoch，并使用具有最高验证 F1 分数的模型进行测试。 所有对比模型的训练可以在单个 NVIDIA V100 16GB GPU 中在 24 小时内完成。  Seq2UMTree 在两个数据集中的解码顺序都是 r-t-h。 我们将在 4.2 小节中讨论该命令的影响。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/figure_4&5.png)

图 4：具有不同三元组数的测试子集 NYT 和 DuIE 上模型的 F1 分数。 子集包含三元组数为 1、2、3、4 和 >4 的句子，在 NYT 中有 3080、1127、298、315 和 470，在 DuIE 中有 9853、7034、2366、1153 和 1233。

图 5：三元组频率小于阈值的模型的 F1 分数。三元组频率表示测试三元组在训练集中出现的频率。

#### 3.2 主要结果(Main Results)

实验结果如表1所示。由于 GPU 内存的限制，Seq2Seq 模型和 MHS 无法处理所有测试数据。 测试集的有效句子百分比显示在#test 列中。  WDec 将最大解码长度设置为 50，而 CopyMTL 最多只能解码 5 个三元组，导致它们在 DuIE 测试集上的覆盖不完整，其中 8.1% 和 3.8% 的测试句子在预处理阶段被删除。 此外，由于 DuIE 中的实体通常比 NYT 具有更多的标记，因此 WDec 的最大解码长度过滤掉 DuIE 中的示例（8.1%）比 NYT 中的（1.2%）更多。  MHS 通过穷举所有标记对来提取三元组，导致编码句子的 GPU 内存消耗为 O(l2r)，其中 l 是句子长度，r 是关系数。 在我们的复制中，我们删除了 NYT 中超过 100 个标记和 DuIE 中超过 150 个标记的句子，这覆盖了 NYT 测试集的 0.5% 和 DuIE 测试集的 1.6%。 在所有模型中，只有 Seq2UMTree 可以应用于两个数据集 2 中的所有句子，空间复杂度为 O(2l + r)。

从表 1 中我们可以看到，Seq2UMTree 在 DuIE 中的 F1 分数比之前最好的 Seq2Seq 模型 WDec 高 15.6%，但在 NYT 中比 WDec 低 3.1%。 两个数据集上表现的不一致促使我们在下一节中进行更深入的调查。

## 4 数据和模型偏差调查（Investigation on Data & Model Bias）

#### 4.1 曝光偏差和泛化（Exposure Bias and Generalization）

Seq2Seq 为三元组分配顺序，而 Seq2UMTree 以无序方式生成三元组，而不管三元组的数量如何。 为了验证 Seq2UMTree 在多个三元组上的有效性，我们将 NYT 和 DuIE 测试集分成五个子集，其中每个句子只包含特定数量的三元组（1、2、3、4、>4）。 子集中模型的性能如图4所示。在 DuIE 中，当三元组数增加时，WDec 的 F1 分数从 70% 急剧下降到大于 2 的三元组数的 40%。随着三元组数的增加，MHS 和 Seq2UMTree 表现更好。 相比之下，在 NYT 中，所有模型在使用不同数量的三元组时表现相似。 为了更好地解决性能差异背后的原因，我们对数据进行了定性分析，发现在 NYT 中，测试集中 90% 的三元组在训练集中重复出现，而在 DuIE 中，这个百分比只有 30%。 基于这一观察，我们假设 Seq2Seq 模型在 NYT 中获得高分是因为暴露偏差：由于测试集中的三元组与训练集中的三元组高度重叠，模型通过记住频繁出现的训练集获得高分 三元组，这会导致过度拟合，使模型很难泛化到看不见的三元组。

为了研究数据偏差对重复出现频率的影响，我们根据训练集中三元组的重复出现频率 (1-10) 将测试集分成 10 个子集。 结果如图所示。  5. 在 NYT 中，WDec 和 Seq2UMTree 的 F1 分数随着重复出现频率的增加而增加。 在 DuIE 中，尽管重复出现频率，但性能曲线几乎是平坦的。这意味着性能与 NYT 中的再次发生频率（90% 再次发生）高度相关，但与 DuIE 中的重复发生频率（30% 再次发生）的相关性极小。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/table_3.png)

表 3：纽约时报的 AB 测试。 我们将 NYT 测试集拆分为两个子集。Test-A 集（2,625 个句子）中的三元组 100% 出现在过滤训练子集（33,963 个句子）中，而 Test-B 集（2,317 个句子）中的三元组从未出现在过滤训练集中。

为了进一步证明曝光偏差对可见和不可见数据的影响，我们对 NYT 数据集进行了 AB 测试。 我们从 NYT 训练集中取一个新的训练集，然后从 NYT 测试集中取两个新的测试集 Test-A 和 Test-B：Test-A 的三元组与新训练集中的这些三元组 100% 重叠，但是 Test-B 中的三元组从未出现在新的训练集中。 新训练集包含原始训练集的 60%。  Test-A 和 Test-B 分别包含原始句子的 53% 和 47%。 结果列于表3中.

尽管 Seq2UMTree 在 100% 重叠的集合中表现不如 WDec，但它在看不见的集合中表现优于 WDec。  Seq2UMTree 的性能下降（从可见到不可见）小于 WDec，这意味着 Seq2UMTree 更健壮和更可靠。 这验证了我们的假设，即 Seq2Seq 模型更容易受到曝光偏差的影响，从而导致更多的过度拟合，而具有最小曝光偏差的 Seq2UMtree 更泛化到看不见的三元组。

由于 NYT 数据集在其训练和测试集中有很大一部分重叠的三元组，并且已经被现有模型过度拟合，我们建议 NYT 不够公正，不能用作基线数据集，并且模型的 F1 分数 在 NYT 上不可靠。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/%E5%A4%8D%E6%9D%82%E8%AF%AD%E5%A2%83%E4%B8%8B%E7%9A%84%E5%AE%9E%E4%BD%93%E5%85%B3%E7%B3%BB%E6%8A%BD%E5%8F%96/3.%E5%BA%8F%E5%88%97%E5%88%B0%E5%BA%8F%E5%88%97/Seq2UMTree/figure/table_4.png)

#### 4.2 顺序三元组(Orders within Triplets)

在 Seq2UMTree 中，关系、头实体和尾实体仍按预定义的顺序（例如，h-r-t 或 r-t-h）解码。 我们枚举每个数据集中所有六种可能的解码顺序并比较性能。 结果示于表中。  4. 三元组中的表现因顺序而异，而两个数据集中对 t-h-r 和 h-t-r 顺序的召回率分别急剧下降。

然后我们假设三元组内的顺序在某种程度上很重要。 考虑到这一点，我们决定逐时间查看训练阶段的时间，发现这 2 个订单甚至不能很好地拟合训练集：htr 在训练集上的召回率仅为 13%（在测试集上为 12%  ）。 此外，大多数预测在第一个时间步 (h) 中丢失。 这意味着 r 的位置提供了对预测很重要的信息并证明了我们的假设。 通过彻底的误差分析，我们意识到对于阶htr（thr遵循相同的逻辑），模型必须在第二个时间步预测所有关于h的t，不受r的约束，这使得每个可能的实体 成为预测候选者。 然而，该模型无法在第三个时间步消除无关系实体对，因此该模型容易将低几率（低召回率）但高置信度（高精度）的实体对馈送到分类层。

相比之下，对于 h-r-t 阶，给定预测的 h，可以根据上下文轻松识别对应的 r。 随后，预测的 h-r 对给出了最后一个时间步预测的强烈暗示，因此模型不会从无关系中崩溃。 这也适用于前两个时间步中带有 r 的任何其他顺序。

## 5 相关工作

以前的工作使用 PIPELINE 从文本中提取三元组（Nadeau 和 Sekine，2007 年；Chan 和 Roth，2011 年）。 他们首先识别输入句子中的所有实体，然后详尽地对每个实体对的关系进行分类。  Li and Ji (2014) 指出分类错误可能会跨子任务传播。 对于联合实体和关系提取（JERE），TABLE 方法不是分别处理这两个子任务，而是通过穷举计算所有标记对和关系的相似性得分，并通过表中输出的位置找到提取的三元组（Miwa 和 Bansal，2016 年；Gupta 等人，2016 年）。 然而，由于三元组可能包含不同长度的实体，表方法要么遭受指数计算负担（Adel 和 Schüutze，2017 年），要么回滚到流水线方法（Sun 等人，2018 年；Bekoulis 等人，2018 年）  ; Fu 等人，2019 年）。 此外，这种表格枚举二次稀释了正标签，从而加剧了类别不平衡问题。 为了以更简洁的方式对任务建模，Zheng 等人。  (2017) 提出了一种 NOVELTAGGING 方案，它在一个标签中表示关系和实体，以便联合提取可以通过经过充分研究的序列标记方法来解决。 然而，这种标记方案不能将多个标签分配给一个令牌，因此在重叠的三元组上失败。 后续方法修改了标记方案以启用多通道序列标记（Takanobu 等人，2018 年；Dai 等人，2019 年），但它们引入了与表格方法类似的稀疏问题。

另一种有前景的方法 SEQ2SEQ 由 Zeng 等人首先提出。  (2018)。  Seq2Seq 不仅可以直接解码三元组列表，还可以规避重叠三元组问题。 尽管本文引入了无法预测多令牌实体的问题，但该问题已被多篇后续论文解决（Zeng 等，2019a；Nayak 和 Ng，2019）。 然而，Seq2Seq 模型仍然存在一个弱点，即暴露偏差，这一点一直被忽视。

暴露偏差源于训练和测试之间的差异：Seq2Seq 模型使用数据分布进行训练，使用模型分布进行测试（Ranzato 等，2015）。 现有工作主要集中在如何减轻 arg max 采样的信息丢失（Yang et al., 2018, 2019; Zhang et al., 2019）。 南等人。  (2017) 注意到不同的阶数会影响 Seq2Seq 模型在多类分类 (MCC) 中的性能，并对频率阶次和拓扑阶次进行彻底的实验。 在 JERE 中，Zeng 等人。  (2019b) 研究了额外的基于规则的三元组预测顺序，包括字母顺序、混洗和固定未排序，然后提出了一个强化学习框架来动态生成自适应顺序的三元组。  Tsai 和 Lee (2019) 首先指出不必要的顺序会导致曝光偏差改变 MCC 的性能，他们发现 Seq2Seq 模型容易过度拟合频繁的标签组合，并且对看不见的目标序列的泛化能力较差。

我们的方法解决了曝光偏差问题。由于曝光偏差问题源于有序的从左到右的三元组解码，我们通过删除三元组生成的顺序来阻止它们彼此的解码，因此可能的预测误差不能从三元组传播到三元组。 此外，由于每个三元组都是由独立的解码过程生成的，因此解码长度大大缩短，从而最大限度地减少了曝光偏差的影响。 我们的方法与之前关于曝光偏差的解决方案不同，我们通过结构解码而不是随机采样来移除顺序（Tsai 和 Lee，2019）。

CASREL (Wei et al., 2020) 是最近提出的两步标注方法，它首先找到句子中的所有头部实体，然后为每个头部实体标记一个关系-尾部表，这也可以看作是一个 UMTree 解码器 解码长度为2。 然而，他们忽略了 NYT 中的数据偏差问题，这会导致模型不可靠和可能的模型偏差。

请注意，我们的任务与 ONEIE (Lin et al., 2020) 不同，ONEIE 以 Seq2Graph 方式对事件提取、实体跨度检测、实体类型识别和关系提取进行建模。 与 ONEIE 相比，JERE 旨在仅提取关系实体三元组，这可以自然地由我们的 UMTree 结构建模。 树的简单性使模型能够进行全局提取。

## 6 总结

在本文中，我们彻底分析了 Seq2Seq 模型的曝光偏差对联合实体和关系提取的影响。 曝光偏差会导致过度拟合，从而损害性能分数的可靠性。 为了解决曝光偏差的问题，我们指出目标三元组的顺序是多余的，并将目标三元组序列制定为无序多树。  Unordered-MultiTree 结构通过将三元组中的解码长度限制为三个，并去除三元组之间的顺序来最小化曝光偏差的影响。 我们进行了深入的实验，揭示了暴露偏差和数据偏差之间的关系。 结果表明我们的模型具有很好的泛化性。















