# Longformer: The Long-Document Transformer 

# Longformer：处理长文本的Transformer

## 摘要

基于 Transformer 的模型由于其自注意力操作而**无法处理长序列**，该操作与序列长度成二次方缩放。为了解决这个限制，我们引入了带有注意力机制的 Longformer，该机制随序列长度线性扩展，从而可以轻松处理包含数千个或更长标记的文档。  Longformer 的注意力机制是标准自注意力的替代品，将**局部窗口注意力**与任务驱动的**全局注意力**相结合。 继之前在长序列转换器上的工作之后，我们在字符级语言建模上评估了 Longformer，并在 text8 和 enwik8 上取得了最先进的结果。与大多数先前的工作相比，我们还预训练 Longformer 并在各种下游任务上对其进行微调。我们预训练的 Longformer 在长文档任务上始终优于 RoBERTa，并在 WikiHop 和 TriviaQA 上设置了最新的最新结果。 我们最终介绍了 Longformer-Encoder-Decoder (LED)，这是一种支持长文档生成序列到序列任务的 Longformer 变体，并在 arXiv 摘要数据集上证明了其有效性。

## 1 前言

Transformers (Vaswani et al., 2017) 在包括生成语言建模 (Dai et al., 2019; Radford et al., 2019) 和判别语言理解在内的广泛自然语言任务中取得了最先进的结果 （德夫林等人，2019 年）。 这一成功部分归功于自注意力组件，它使网络能够从整个序列中捕获上下文信息。 虽然功能强大，但自注意力的内存和计算要求与序列长度成二次方增长，这使得处理长序列变得不可行（或非常昂贵）。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/figure_1.png)

图 1：full self-attention 的运行时和内存以及 Longformer 的 self-attention 的不同实现；  Longformer-loop 是非矢量化的，Longformer-chunk 是矢量化的，而 Longformer-cuda 是自定义的 cuda 内核实现。Longformer 的内存使用与序列长度呈线性关系，这与当前 GPU 上长序列内存不足的完全自注意力机制不同。不同的实现速度不同，向量化的 Longformer-chunk 是最快的。 更多细节在第 3.2 节。

为了解决这个限制，我们提出了 Longformer，这是一种改进的 Transformer 架构，具有自注意力操作，可随序列长度线性扩展，使其在处理长文档时具有通用性（图 1）。 这对于自然语言任务（例如长文档分类、问答 (QA) 和共指解析）来说是一个优势，其中现有方法将长上下文划分或缩短为更小的序列，这些序列属于 BERT 式预训练模型的典型 512 个标记限制 . 这种分区可能会导致重要的跨分区信息的丢失，为了缓解这个问题，现有的方法通常依赖于复杂的架构来解决这种交互。 另一方面，我们提出的 Longformer 能够使用多层注意力构建整个上下文的上下文表示，从而减少对特定任务架构的需求。

最近的工作解决了 Transformer 在长序列上的计算效率低下的问题（见表 1）。 然而，他们主要关注自回归语言建模 (LM)，而将长文档转换器应用于迁移学习环境中的文档级 NLP 任务（Dai 和 Le，2015 年；Peters 等人，2018 年；Howard 和 Ruder，2018 年；Devlin 等人，2019 年）在很大程度上仍未得到探索。 我们解决了这个差距，并表明 Longformer 的注意力机制可以作为预训练 Transformer 中自我注意力机制的替代品，并在一系列文档 NLP 任务中获得收益。

Longformer 的注意力机制是窗口局部上下文自注意力和最终任务激发的全局注意力的组合，该全局注意力对任务进行归纳偏差编码。 通过**消融和对照试验**，我们表明两种注意力类型都是必不可少的——局部注意力主要用于构建上下文表示，而全局注意力允许 Longformer 构建用于预测的完整序列表示。

我们首先使用窗口化和新的扩张注意模式的组合在自回归字符级语言建模上评估 Longformer，允许模型在现代 GPU 上处理多达 32K 个字符的序列。 我们在 **text8 和 enwik8 基准数据集**上取得了最先进的结果，证明了 Longformer 在长文档建模中的有效性。

然后，为了评估 Longformer 替换现有预训练模型的完全自注意力操作的能力，我们使用掩码语言建模 (MLM) 目标对其进行预训练，从 RoBERTa (Liu et al., 2019) 发布的检查点继续。预训练后，我们通过微调将其应用于下游语言任务，并证明 Longformer 在广泛的文档级自然语言任务（包括文本分类、QA 和共指解析）上始终优于 RoBERTa，在以下两个方面取得了最先进的结果 这些数据集。

我们最后介绍了 Longformer 的一种变体，它不是仅编码器的 Transformer 架构，而是遵循类似于原始 Transformer 模型（Vaswani 等人，2017）的编码器-解码器架构，并且用于序列到序列（  seq2seq) 学习（Sutskever 等人，2014 年）。 我们将此模型称为 **Longformer-Encoder-Decoder (LED)**，它在编码器网络上使用 Longformer 的高效注意力模式，使其能够处理长文档 seq2seq 任务，例如摘要。 我们证明了 LED 在 arXiv 摘要数据集上的有效性（Cohan 等人，2018 年）。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_1.png)

表 1：先前关于为长文档调整 Transformer 的工作总结。  ltr：从左到右。

## 2 相关工作

**Long-Document Transformer** 图 1 总结了近期关于长文档的工作。 已经探索了两种类型的自注意力方法。 第一种是从左到右 (ltr) 方法，它以从左到右移动的块处理文档。 尽管此类模型在自回归语言建模中取得了成功，但它们不适用于具有受益于双向上下文的任务的迁移学习方法。

我们的工作属于另一种通用方法，该方法定义了某种形式的稀疏注意力模式并避免计算完整的二次注意力矩阵乘法。 与我们的注意力模式最相似的模型是 Sparse Transformer (Child et al., 2019)，它使用 BlockSparse (Gray et al., 2017) 提供的大小为 8x8 的块的扩张滑动窗口形式。 我们的实现（第 3 节）还包括一个自定义 CUDA 内核，但它比 BlockSparse 更灵活和可维护，后者是用 C++ 实现的，专为 TensorFlow 的特定版本而设计。 我们还介绍了适用于常见 NLP 任务（§3）的额外任务驱动全局注意力模式，并表明它们对于迁移学习环境中的良好表现至关重要。sq

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/figure_2.png)

图 2：比较 Longformer 中的完整自注意力模式和注意力模式的配置。

一些模型尝试了自回归语言建模以外的任务，这是向前迈出的一步，因为可以说专注于语言建模作为主要评估导致了适用性有限的模型的开发。  BPTransformer (Ye et al., 2019) 对机器翻译 (MT) 进行了评估，但没有探索 pretrainfinetune 设置。  Blockwise attention (Qiu et al., 2019) 预训练了他们的模型并评估了问答 (QA)。 然而，评估是有限的，因为它不包括语言建模，而且 QA 数据集的文档相对较短，2 因此该模型在长文档任务上的有效性仍有待探索。

**Task-specific Models for Long Documents** 已经开发了许多特定于任务的方法来解决 BERT 等预训练 Transformer 模型的 512 限制。 最简单的方法只是截断文档，通常用于分类（Xie 等，2019）。另一种方法将文档分成长度为 512 的块（可能重叠），分别处理每个块，然后将激活与特定于任务的模型相结合（Joshi 等，2019）。在多跳和开放域 QA 任务中流行的第三种方法使用两阶段模型，其中第一阶段检索传递到第二阶段进行答案提取的相关文档（克拉克和加德纳，2017 年；陈等人，2017 年）。由于两阶段方法的截断或级联错误，所有这些方法都会遭受信息丢失。 相比之下，Longformer 可以在不截断或分块的情况下处理长序列，这使我们能够采用更简单的方法，将可用上下文连接起来并在一次传递中对其进行处理。

一些同时代的作品 3 探索了与 Longformer 类似的想法，在 Transformer 中使用局部 + 全局注意力，并针对长文档自然语言任务对其进行预训练。 特别是，ETC（Ainslie 等人，2020 年）使用类似的局部 + 全局注意力而不是完全自我注意力来将 Transformer 扩展到长文档。 与 Longformer 不同，ETC 使用相对位置嵌入（我们仅用于自回归 LM 设置），为预训练引入了额外的训练目标（CPC 损失），并以略有不同的方式配置全局注意力。
   它在包括阅读理解和分类在内的多项任务上显示出强大的结果。  GMAT（Gupta 和 Berant，2020 年）使用了类似的想法，即输入中的几个全局位置用作全局记忆。  BigBird (Zaheer et al., 2020) 是对 ETC 的扩展，对附加任务进行评估，包括总结。 重要的是，通过理论分析，BigBird 表明稀疏 Transformer 是序列函数的通用逼近器，并保留了完全自注意力的这些属性。

## 3 Longformer

原始的 Transformer 模型有一个自注意力组件，时间和内存复杂度为 O(n2)，其中 n 是输入序列长度。 为了应对这一挑战，我们根据“注意模式”来稀疏化完整的自注意矩阵，该“注意模式”指定了相互注意的输入位置对。与完全自注意力不同，我们提出的注意力模式与输入序列线性缩放，使其对更长的序列有效。 本节讨论这种注意力模式的设计和实现。

### 3.1 注意模式

**滑动窗口** 考虑到局部上下文的重要性（Kovaleva 等人，2019 年），我们的注意力模式在每个标记周围采用固定大小的窗口注意力。 使用这种窗口注意力的多个堆叠层会产生一个大的感受野，其中顶层可以访问所有输入位置，并有能力构建包含整个输入信息的表示，类似于 CNN（Wu 等人，2019）。给定一个固定的窗口大小 w，每个令牌在每侧处理 1 2w 个令牌（图 2b）。 该模式的计算复杂度为 O(n × w)，它与输入序列长度 n 线性缩放。在具有 ℓ 层的转换器中，顶层的感受野大小为 ℓ × w（假设 w 对于所有层都是固定的）。 根据应用程序，为每一层使用不同的 w 值来平衡效率和模型表示能力（第 4.1 节）可能会有所帮助。

**Dilated Sliding Window** 为了在不增加计算量的情况下进一步增加感受野，可以“扩张”滑动窗口。 这类似于扩张的 CNN（van den Oord 等人，2016 年），其中窗口具有大小为 d 的间隙（图 2c）。假设所有层的 d 和 w 固定，感受野为 ℓ × d × w，即使 d 的值很小，它也可以达到数万个标记。

在多头注意力中，每个注意力头计算不同的注意力分数。 我们发现每个头部具有不同扩张配置的设置通过允许一些没有扩张的头部专注于局部上下文，而其他具有扩张的头部专注于更长的上下文来提高性能。

**全局注意力 **在用于自然语言任务的最先进的 BERT 样式模型中，最佳输入表示与语言建模不同，并且因任务而异。 对于掩码语言建模 (MLM)，该模型使用本地上下文来预测掩码词，而对于分类，该模型将整个序列的表示聚合为一个特殊的标记（在 BERT 的情况下为 [CLS]）。 对于 QA，问题和文档是串联的，允许模型通过自注意力将问题与文档进行比较。

在我们的例子中，窗口化和扩张的注意力不够灵活，无法学习特定于任务的表示。 因此，我们在几个预先选择的输入位置上添加了“全局注意力”。 重要的是，我们使这个注意力操作对称：也就是说，具有全局注意力的令牌关注整个序列中的所有令牌，并且序列中的所有令牌都关注它。 图 2d 显示了一个滑动窗口注意力的例子，在自定义位置的几个标记上具有全局注意力。 例如，对于分类，全局注意力用于 [CLS] 标记，而在 QA 中，全局注意力用于所有问题标记。 由于此类标记的数量相对于 n 而言很小，并且与 n 无关，因此结合局部和全局注意力的复杂度仍然为 O(n)。 虽然指定全局注意力是特定于任务的，但它是一种向模型注意力添加归纳偏差的简单方法，并且比现有的任务特定方法简单得多，后者使用复杂的架构来组合较小输入块的信息。

全局注意力的线性投影回想一下，给定线性投影 Q、K、V，Transformer 模型（Vaswani 等人，2017 年）计算注意力分数如下：

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/formula_1.png)

我们使用两组投影 Qs、Ks、Vs 来计算滑动窗口注意力的注意力分数，以及 Qg、Kg、Vg 来计算全局注意力的注意力分数。 额外的投影为模拟不同类型的注意力提供了灵活性，我们表明这对于下游任务的最佳性能至关重要。  Qg、Kg、Vg 均使用与 Qs、Ks、Vs 匹配的值进行初始化。

### 3.2 实现

在常规转换器中，注意力分数的计算方式与公式相同。  1. 代价高昂的操作是矩阵乘法 QKT，因为 Q 和 K 都有 n（序列长度）投影。 对于 Longformer，扩张的滑动窗口注意力只计算 QKT 的固定数量的对角线。 如图 1 所示，与完全自注意力的二次增加相比，这导致内存使用量的线性增加。 但是，实现它需要一种带状矩阵乘法形式，而现有的深度学习库（如 PyTorch/Tensorflow）不支持这种形式。 图 1 比较了三种不同实现方式的性能：loop 是一种内存高效的 PyTorch 实现，支持扩张但速度慢得无法使用，仅用于测试；  chunks 只支持非扩张情况，用于预训练/微调设置；  cuda 是我们使用 TVM (Chen et al., 2018) 实现的功能齐全的高度优化的自定义 CUDA 内核，用于语言建模实验（有关更多详细信息，请参见附录 A）。

## 4 自回归语言建模

自回归或从左到右的语言建模被松散地定义为在给定输入序列中的先前标记/字符的情况下估计现有标记/字符的概率分布。此任务被认为是自然语言中的基本任务之一，最近使用转换器对长序列进行建模的先前工作依赖于此任务作为其主要评估（Dai 等人，2019 年；Rae 等人，2020 年；Sukhbaatar 等人，2019 年）。  , 2019)。同样，我们在自回归语言建模上开发和评估我们的模型。

### 4.1 注意力模式

对于自回归语言建模，我们使用我们的扩张滑动窗口注意力。继 Sukhbaatar 等人之后。  (2019) 我们在各层中使用不同的窗口大小。 特别是，我们对较低层使用较小的窗口大小，并在移动到较高层时增加窗口大小。这允许顶层学习整个序列的更高级别表示，同时让较低层捕获本地信息。 此外，它提供了效率（较小的窗口尺寸由于较少的非零值而计算成本较低）和性能（较大的窗口尺寸具有更丰富的表示能力并通常会导致性能改进）之间的平衡。

我们不对较低层使用扩张的滑动窗口，以最大限度地提高其学习和利用直接本地上下文的能力。 对于更高的层，我们仅在 2 个头上使用少量的递增膨胀。 这使模型能够在不牺牲本地上下文的情况下直接处理远处的标记。

### 4.2 实验设置

为了与之前的工作进行比较，我们专注于字符级 LM（text8 和 enwik8；Mahoney，2009）。

**训练** 理想情况下，我们希望在现代 GPU 内存中可以容纳的最大窗口大小和序列长度上训练我们的模型。 然而，我们发现模型需要大量的梯度更新来首先学习局部上下文，然后再学习利用更长的上下文。 为了适应这一点，我们采用了分阶段训练过程，在该过程中我们增加了多个训练阶段的注意力窗口大小和序列长度。 特别是，在第一阶段，我们从较短的序列长度和窗口大小开始，然后在每个后续阶段，我们将窗口大小和序列长度加倍，并将学习率减半。 这使得训练速度更快，同时将慢速部分（最长序列和窗口大小）保持到最后。
   我们在 5 个总阶段上训练模型，最后一个阶段的起始序列长度为 2,048，结束序列长度为 23,040（有关每个阶段的详细配置以及所有其他超参数，请参见附录 B）。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_2.png)

表 2：text8 和 enwik8 上的小型模型 BPC

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_3.png)

表 3：大型模型在 enwik8 上的性能

评估 我们使用长度为 32,256 的序列进行评估。 继戴等人。  (2019)，我们将数据集拆分为大小为 32,256 的重叠序列，步长为 512，并报告该序列上最后 512 个标记的性能。

#### 4.2.1 结果

选项卡。 图 2 和图 3 总结了 text8 和 enwik8 数据集的评估结果。 我们在 text8 和 enwik8 上分别使用 BPC 为 1.10 和 1.00 的小模型在 text8 和 enwik8 上实现了新的最新技术，证明了我们模型的有效性。

对于大型模型，鉴于这些实验的成本很高，并且根据最近的工作（Kitaev 等人，2020 年；Rae 等人，2020 年），我们仅对 enwik8 进行评估。 选项卡。 图 3 显示 Longformer 优于可比较的 TransformerXL 模型，与可比较的 Sparse Transformer（Child 等人，2019 年）的性能相匹配，并且与具有两倍以上参数数量的近期模型相匹配或略逊一筹。值得注意的是，Adaptive Span (Sukhbaatar et al., 2019) 和 Compressive Transformer (Rae et al., 2020) 并不适合第 2 节中讨论的预训练微调范式。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_4.png)

表 4：顶部：跨层更改窗口大小。 底部：有/无扩张（@阶段 1 上的 150K 步）

#### 4.3.2 消融研究

为了展示我们的注意力模式设计选择的重要性，我们尝试了不同的变体并报告了它们的受控实验结果。 为了使消融研究更易于管理，我们在 text8 上的小模型上使用阶段 1 配置训练每个配置 15 万步 4，然后报告开发集上的 BPC 性能。
选项卡的顶部。 图 4 展示了配置每层窗口大小的不同方式的影响。 我们观察到从底层到顶层增加窗口大小会导致最好的性能，以相反的方式排列它们会导致更差的性能，并且使用固定的窗口大小（其他配置的窗口大小的平均值）会导致 一种介于两者之间的表现。 选项卡的底部。 图 4 显示了添加膨胀的影响。 与根本没有扩张相比，向两个头部添加一些扩张会导致一些改进。

## 5 预训练和微调（Fine-tune）

当前用于许多 NLP 任务的最先进系统通过任务监督（例如 BERT）对预训练模型进行了微调。 我们的主要动机之一是开发适合长文档任务的模型。 为此，我们在文档语料库上对 Longformer 进行了预训练，并针对六项任务对其进行了微调，包括分类、QA 和共指解析。 生成的模型可以处理长达 4,096 个令牌的序列（比 BERT 长 8 倍）5。

我们使用掩码语言建模 (MLM) 预训练 Longformer，目标是恢复序列中随机掩码的标记。 由于 MLM 预训练很昂贵，我们继续从 RoBERTa (Liu et al., 2019) 发布的检查点进行预训练，同时只使最小模型库大 RoBERTa (seqlen: 512) 1.846 1.496 Longformer (seqlen: 4,096) + 10.299 复制位置 8.73 嵌入 1.957 1.597 + 2K 梯度更新 1.753 1.414 + 65K 梯度更新 1.705 1.358 Longformer（仅训练额外位置嵌入） 1.850 1.504 表 5：用于 RoBERTa 和各种预训练 Longformer 配置的 MLM BPC    支持 Longformer 注意力机制所需的更改。 请注意，我们的注意力模式可以插入任何预训练的 Transformer 模型，而无需更改模型架构。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_5.png)

**注意模式(Attention Pattern)**  我们使用窗口大小为 512 的滑动窗口注意，因此使用与 RoBERTa.6 相同的计算量

**位置嵌入(Position Embeddings)**  RoBERTa 使用学习的绝对位置嵌入，最大位置为 512。为了支持更长的文档，我们添加了额外的位置嵌入以支持最多 4,096 个位置。 为了利用 RoBERTa 的预训练权重，我们不是随机初始化新的位置嵌入，而是通过多次复制来自 RoBERTa 的 512 个位置嵌入来初始化它们，因为对 BERT 注意力头的分析显示了对关注本地上下文的强烈学习偏差，包括前一个或下一个 令牌（克拉克等人，2019 年）。 使用复制初始化可以在除分区边界之外的任何地方保留此本地结构。 尽管它很简单，但我们发现这是非常有效的（见表 5），允许 Longformer 预训练以少量梯度更新快速收敛。

**Continued MLM Pretraining** 预训练我们使用 fairseq（Ott 等人，2019 年）在我们编译的长文档语料库上预训练 Longformer（有关语料库的详细信息，请参见附录 C）。 我们训练两个模型尺寸，一个基本模型和一个大模型。 两种模型都经过 65K 梯度更新训练，序列长度为 4,096，批次大小为 64（218 个标记），最大学习率为 3e-5，线性预热为 500 步，然后是 3 次幂多项式衰减。 其余超参数与 RoBERTa 相同。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_6.png)

表 6：wordpieces 中数据集上下文长度的平均值和第 95 个百分位。  WH: WikiHop, TQA: TriviaQA, HQA: HotpotQA, ON: OntoNotes, HY: Hyperpartisan news

图 5 显示了我们训练语料库开发集上的 BPC。 第一行显示了使用 RoBERTa-base 的 1.846 BPC，这与 RoBERTa 论文在其语料库上报告的 1.880 BPC 相当。 这表明我们的训练语料库来自与用于训练 RoBERTa 的分布接近的分布。 以下两行显示了 Longformer 在使用随机初始化的位置嵌入和复制的位置嵌入进行预训练之前的性能。 显着差异表明复制初始化的重要性，RoBERTa BPC 和初始化 BPC 之间相对较小的差异表明我们的滑动窗口注意力与 RoBERTa 权重配合良好。 以下两行显示了继续预训练的影响。  2K 步的训练将 BPC 从 1.957 提高到 1.753，在 65K 步后进一步降低到 1.705，表明该模型正在学习更好地利用滑动窗口注意力和更长的上下文。 使用 RoBERTa-large 和 Longformer-large 观察到类似的模式。

**冻结 RoBERTa权重**  我们还在冻结所有 RoBERTa 权重的同时预训练了 Longformer，并且只训练了新的位置嵌入。这种配置的动机是完美地保留 RoBERTa 在短文档上的性能。 此配置的 BPC 为 1.850（低于初始化时的 1.957），但高于 1.705，其中所有权重都是可训练的。

## 6 任务

我们将 Longformer 应用于多个长文档任务，包括 QA、共指解析和分类。 选项卡。 图 6 显示评估数据集的上下文明显长于 512 个词条。 我们的主要目标是评估我们的注意力机制是否可以替代 BERT 风格模型中的标准自我注意力机制，并针对强基线进行对照试验。 我们也有兴趣评估是否可以用更简单的模型替换 BERT 有限上下文所需的复杂任务特定模型，这些模型只是将所有可用上下文连接成一个序列。

我们的基线是一个基于 RoBERTa 的模型，它将上下文分解成尽可能长的片段，每个片段单独通过 RoBERTa，并连接激活以进行进一步处理。 对于 QA 任务，我们还将问题连接到每个段，以便 RoBERTa 可以调节问题上下文的上下文表示。  Longformer 变体用我们在预训练期间使用的窗口注意力替换了 RoBERTa 自注意力机制，加上任务驱动的全局注意力。 全局注意力使用额外的线性投影（第 3.1 节）。

### 6.1 QA

我们使用了三个数据集：WikiHop（Welbl 等人，2018 年）、TriviaQA（Joshi 等人，2017 年，维基百科设置）和 HotpotQA（Yang 等人，2018 年，干扰设置）7。

对于 WikiHop 和 TriviaQA，我们遵循 BERT 的简单 QA 模型（Devlin 等人，2019 年），将问题和文档连接成一个长序列，通过 Longformer 运行它，然后有一个特定于数据集的预测层。  WikiHop 为候选词使用分类层，而 TriviaQA 使用 Clark 和 Gardner (2017) 的损失函数来预测答案跨度。 我们包括对 WikiHop 的问题标记和答案候选以及 TriviaQA 的问题标记的全球关注。

HotpotQA 是一个多跳 QA 数据集，涉及从 10 个维基百科段落中提取答案范围和证据句子，其中 2 个是相关的，其余是干扰项。 我们使用两阶段模型，首先选择最相关的段落，然后将它们传递到第二阶段进行答案提取。 两个阶段都将问题和上下文连接成一个序列，通过 Longformer 运行它，然后使用特定于任务的预测层。 我们以多任务方式训练模型，以联合预测相关段落、证据句子、答案跨度和问题类型（是/否/跨度）。 请注意，该模型比最近包含复杂任务特定架构的 SOTA 模型更简单（例如，（Tu 等人，2019 年；Chen 等人，2019 年；Tu 等人，2020 年；Groeneveld 等人，2020 年））  . 有关模型和超参数的更多详细信息，请参阅附录 D。

### 6.2 共指解析

我们使用 OntoNotes（Pradhan 等人，2012 年）和 Joshi 等人的模型。  （2019），Lee 等人对系统的修改。  (2018) 用 BERT 替换 ELMo。  Longformer 系统是对基线模型的直接改编，通过用 Longformer 替换 RoBERTa 并扩展序列长度。 我们没有在这个任务中使用全局注意力。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_7.png)

表 7：QA、共指解析和文档分类的微调结果总结。 结果是在比较我们的 Longformer-base 和 RoBERTa-base 的开发集上。  TriviaQA，Hyperpartisan 指标是F1，WikiHop 和IMDB 使用准确率，HotpotQA 是联合F1，OntoNotes 是平均F1。

### 6.3 文本分类

我们对 IMDB (Maas et al., 2011) 和 Hyperpartisan news detection (Kiesel et al., 2019) 数据集进行评估。8 IMDB 是由电影评论组成的标准情感分类数据集。 虽然该数据集中的大多数文档都很短，但其中约 13.6% 的文档大于 512 个词（表 6）。  Hyperpartisan 中的文档比较长，而且很小，只有 645 个文档，很好地测试了 Longformer 对有限数据的适应能力。 我们对 [CLS] 令牌使用全局关注。

### 6.4 结果

**主要结果** 图 7 总结了我们所有微调实验的结果。 我们观察到 Longformer 始终优于 RoBERTa 基线。 对于 WikiHop 和 Hyperpartisan 等需要长上下文的任务，其性能提升尤为明显。 对于 TriviaQA，由于当地环境通常足以回答问题，因此改进更为温和。 在 HotpotQA 的情况下，支持事实辅助监督允许模型轻松找到相关上下文，然后关注局部上下文，从而导致较小的收益。 这与 WikiHop 形成对比，后者仅包括对中间推理链的远程监督，我们的方法在整个上下文中进行推理。 在 IMDB 和 OntoNotes 数据集上，性能提升较小。 对于 IMDB，大部分数据集由短文档组成，因此预计会有较小的改进。 对于 OntoNotes，我们发现任意两个提及项之间的距离通常非常小，因此单独处理较小块的基线能够将提及项拼接成共指链，而无需考虑跨块交互。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_8.png)

表 8：Longformer-large 在提交时（2020 年 5 月）的排行榜结果。 所有数字均为 F1 分数。

**Longformer-large for QA** 我们还评估 Longformer-large 在长上下文 QA 任务上的性能。 选项卡。 图 8 表明，我们的 Longformer-large 在 WikiHop 和 TriviaQA 上取得了新的最先进结果 9（分别为 3.6 和 4 分），而对于 HotpotQA，它的表现低于当前最先进的结果（Fang et  al., 2020) 一点。 选项卡。 图 9 显示了 HotpotQA 与已发布和未发布的并发模型相比的详细结果。Longformer 在已发布的排行榜上排名第二，优于除 HGN 之外的所有其他已发布结果（Fang 等人，2020 年）。 此任务中所有已发布的表现最佳的模型（Tu 等人，2019 年；Fang 等人，2020 年；Shao 等人，2020 年）都使用 GNN（Kipf 和 Welling，2017 年）或实体图网络，它们似乎编码了一个 任务的重要归纳偏差，并可能进一步改善我们的结果。 尽管如此，Longformer 的性能明显优于所有其他方法，包括最近的非 GNN 方法（Glaß 等人，2019 年；Shao 等人，2020 年；Groeneveld 等人，2020 年）。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_9.png)

表 9：HotpotQA 在干扰设置测试集中的结果。夸克的测试结果不可用。 所有数字均为 F1 分数。  † 显示同期排行榜提交。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_10.png)

表 10：WikiHop 开发集消融

### 6.5 WikiHop 上的消融

图 10 展示了 WikiHop 在开发集上的消融研究。 所有结果都使用 Longformerbase，除特别注明外，均使用相同的超参数对五个 epoch 进行了微调。  Longformer 受益于更长的序列、全局注意力、全局注意力的单独投影矩阵、MLM 预训练和更长的训练。 此外，当配置为 RoBERTa-base（seqlen：512 和 n2 attention）时，Longformer 的性能比 RoBERTa-base 稍差，确认性能提升不是由于额外的预训练。 仅在解冻额外的位置嵌入时使用预训练的 RoBERTa 模型时，性能略有下降，这表明 Longformer 可以学习在大型训练数据集（如 WikiHop）的任务特定微调中使用远程上下文。

## 7 Longformer-Encoder-decoder（LED）

最初的 Transformer（Vaswani 等人，2017 年）由编码器-解码器架构组成，用于序列到序列任务（Sutskever 等人，2014 年），例如摘要和翻译。 虽然仅编码器变换器在各种 NLP 任务上都很有效，但预训练的编码器解码器变换器模型（例如 BART（Lewis 等人，2020 年）和 T5（Raffel 等人，2020 年））在摘要等任务上取得了很好的成绩 . 然而，这样的模型不能有效地扩展到具有更长输入的 seq2seq 任务。

为了便于为 seq2seq 学习对长序列进行建模，我们提出了一种 Longformer 变体，它具有编码器和解码器 Transformer 堆栈，但它使用 Longformer 的高效局部+全局注意力模式，而不是编码器中的完全自注意力。 解码器对整个编码令牌和先前解码的位置使用完全自注意力。 我们称这种模型为 Longformer-Encoder-Decoder (LED)，它与输入成线性比例。 由于预训练 LED 很昂贵，我们从 BART 初始化 LED 参数，并在层数和隐藏大小方面遵循 BART 的确切架构。 唯一的区别是，为了处理更长的输入，我们将位置嵌入扩展到 16K 个标记（从 BART 的 1K 个标记开始），并且我们通过重复复制 BART 的 1K 个位置嵌入 16 次来初始化新的位置嵌入矩阵，如第 5 节中针对 RoBERTa 的那样。 在 BART 之后，我们发布了两种模型尺寸，LED-base 和 LED-large，它们在编码器和解码器堆栈中分别具有 6 层和 12 层。

我们使用 arXiv 摘要数据集（Cohan 等人，2018 年）在摘要任务上评估 LED，该数据集专注于科学领域的长文档摘要。 文档长度的第 90 个百分位数是 14.5K 令牌，使其成为评估 LED 的合适测试平台。  LED 的编码器读取文档，其解码器生成输出摘要。 编码器使用窗口大小为 1,024 个标记的局部注意力和第一个标记上的全局注意力。 解码器充分注意整个编码器和先前解码的位置。 作为 seq2seq 模型的标准，LED 使用教师强制进行黄金训练摘要进行训练，并在推理时使用波束搜索。

图 11 展示了 LED-large 16K 在 arXiv 摘要任务上的结果。 该模型仅从 BART 初始化，没有额外的预训练。 我们观察到 LED 在 arXiv 上取得了最先进的结果，略胜于 BigBird（Zaheer 等人，2020 年）。 请注意，BigBird 摘要模型支持 4K 令牌的序列长度，但从 Pegasus（Zhang 等人，2020）开始并继续预训练，这是一个专门为摘要设计和预训练的模型。 由于没有预训练或特定任务的初始化，但能够处理更长的输入，LED 的性能略胜于 BigBird。 通过对 LED 进行预训练，应该可以进一步改进。 图 3 进一步说明了序列长度的重要性，显示处理更长输入的能力显着改善了结果。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/table_11.png)

表 11：LongformerEncoder-Decoder (LED) 在 arXiv 数据集上的总结结果。从左到右的指标是 ROUGE-1、ROUGE-2 和 ROUGE-L。

![](https://gitee.com/Xiaoyingzi09/note-book/raw/master/NLP/Bert/Longformer/figure/figure_3.png)

图 3：改变输入大小时 LED 的 ROUGE-1 和 ROUGE-2（arXiv 验证集）。

## 8 总结与展望

我们提出了 Longformer，这是一种基于转换器的模型，可扩展用于处理长文档，并且可以轻松执行各种文档级 NLP 任务，无需分块/缩短长输入，也无需复杂的架构来组合这些块的信息。  Longformer 采用了一种注意力模式，该模式结合了局部和全局信息，同时也随序列长度线性缩放。Longformer 在 text8 和 enwik8 的字符级语言建模任务上取得了最先进的结果。 预训练后，Longformer 在长文档任务上始终优于 RoBERTa，并在 WikiHop 和 TriviaQA 上设置了最新的最新结果。 我们进一步介绍了 LED，这是 Longformer 的编码器-解码器变体，用于对序列到序列任务进行建模，并在 arXiv 长文档摘要任务上取得了最先进的结果。 对于未来的工作，我们想研究其他预训练目标，尤其是 LED，增加序列长度，并探索可能从我们的模型中受益的其他任务。
