----

> Cite Papers:
>
> - 基于序列标注
>   - **(Novel Tagging)** Joint Extraction of Entities and Relations Based on Novel Tagging Scheme (2017 ACL 基于序列标注)
>   - **(HTB)** A Novel Hierarchical Binary Tagging Framework for Joint Extraction of Entitles and Relations (ACL 2020)
> - 基于表填充
>   -  Modeling Joint Entity and Relation Extraction with Table Representation (EMNLP 2014 基于表填充) 
>   - Joint recognition and relation extraction as a multi-head selection problem（基于表填充工作的一项较好的改进为**多头选择**工作）
> - 基于序列到序列
>   - **(CopyRE)** Extracting Relational Facts by an End-to-End Neural Model with Copy Mechanism (2018 ACL 基于序列到序列)
>   - **(Seq2UMTree)** Minimize Exposure Bias of Seq2Seq Models in Joint Entity and Relation Extraction (EMNLP 2020 Findings)
>   - **(CopyMTL)** Joint recognition and relation extraction as a multi-head selection problem
> - 基于文档抽取
>   - **(GCNN)** Inter-sentence Relation Extraction with Document-level Graph Convolutional Neural Network （ACL 2019） (文档级抽取)
>   - **(EOG)** Connecting the dots: Document-level neural relation extraction with edge-oriented graphs （EMNLP 2019）
>   - **(LSR)** Reasoning with latent structure refinement for document-level relation extraction （ACL 2020）
>   - **(Double Graph)** Double Graph Based Reasoning for Document-level Relation Extraction （EMNLP 2020）

----





## 1 实体关系抽取任务介绍

> 实体关系抽取任务

- 关系定义为两个或多个实体的某种联系
- 实体关系抽取是自动识别出实体之间是否存在某种关系



> 复杂语境

传统的实体关系抽取我们称之为简单语境，其主要针对一个句子中的两个实体之间的语义关系特征，在这种语境下会忽略其他实体或者关系的影响。而复杂语境通常包括两种，如上图中所列出的：（1）同一个句子中多个三元组之间相互影响；如上图（左），Donald J. Trump和Queens之间有Born_in关系，Queens和New York City之间有Located_in关系，那么也可以推断出Donald J. Trump和New York City之间有Located_in关系；（2）大量的实体间关系是通过多个句子表达的，其主要涉及多个实体间的跨句关系抽取。上图（右）为从清华大学刘知远老师团队发布的DocRED数据集上所获得的截图，在发布该数据集时有进行简单的统计，统计结果表明：在维基百科数据集人工标注的结果中，至少有40%实体关系相关的事实只能从多个句子中进行联合抽取。

## 2 实体关系联合抽取

 对于实体关系联合抽取任务，目前有较多的框架/方法类别，如：

- 基于序列标注的方法

- 基于表填充的方法

- 序列到序列的方法

- 多任务学习的方法等

以下我们将就三类方法进行展开介绍。

#### 1 基于序列标注

> 联合抽取：序列标注（NovelTagging）

对于序列标注的方法，不得不提2017年在ACL上发表的一篇文章《Joint Extraction of Entities and Relations Based on Novel Tagging Scheme》，（当时这篇文章也获得了Outstanding paper）。在上述NovelTagging方法中，作者提出了一种新的关系标注模式（Relation tagging schema），对于每种关系，将其与（Begin, Inside，End，Single）以及头实体和尾实体的序号（1,2）组会起来进行关系抽取，并根据最后的标注结果进行解码，进而得到关系三元组。再者，该方法额外考虑了一个Other标签，主要表示不属于任何一种关系。如果总共有|R|种关系，那么一共就有2*4*|R|+1个标签。





























