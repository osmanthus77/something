## json 文件格式

JSON：JavaScript 对象表示（Object Notation），轻量级的文本数据交换格式，独立于语言  
对象是无序的键值对集合，`{"键"："值"，"键"："值"}`形式  
`值`可以是数值、字符串、数组、对象、对象数组、数组对象

- 数值
  {“key1”:100,“key2”:20}
- 字符串
  {“key1” : “张三”,“key2” : “大忽悠”}
- 数组
  {key" : [000, 111111],“key1” : [18874, 15157]}
- 对象
  { “key” : {“1”: “亚索”},“key1” : 」{“2”: “刘备”} }
- 对象数组
  {“我”:[{“key”: “好好学习”},{“key1”:“天天向上”}]}
- 数组对象
  { “我”:{“你” : [18874,15157]} }

## bootstrap

[详细解释 1](https://blog.csdn.net/weixin_39609670/article/details/111200181)  
[详细解释 2](https://www.jianshu.com/p/a556beb7ff37)  
bootstrap（自展法）：用小样本估计总体值的一种非参数方法。  
原理：生成一系列 bootstrap 伪样本，每个样本是初始数据有放回抽样。通过对伪样本的计算，获得统计量的分布。  
进化分析中：通过对数据集多次重复取样，构建多个进化树，用来检查给定树的分枝可信度。针对基因树，通过随机选择位点，以放回式抽样，从实际数据中构造出 1000 个或更多个不同的数据集。  
因此，如果基因树中的某种分支方式被序列中大多数的位点支持，则从大多数自举样本得来的基因树会包含同样的分支方式。
具体来讲，Bootstrap 值是指根据所选的统计计算模型，设定**初始值 1000 次**，就是把序列的位点都重排，重排后的序列再用相同的办法构树，如此让模型计算并**绘制 1000 株系统发育树**，这是命令阶段产生的。如果原来树的分枝在重排后构的树中也出现了，就给这个分枝打上 1 分，如果没出现就给 0 分，这样**给进化树打分**后，每个分枝就都得出分值。系统发育树中每个节点上的数字则代表在命令阶段要求的 1000 次进化树分析中，有多少次。重排的序列有很多组合，值越小说明分枝的可信度越低，最好根据数据的情况选用不同的构树方法和模型。
一般推荐用两种以上不同的方法构建进化树，如果所得到的进化树类似，且 bootstrap 值总体较高，则得到的结果较为可靠。

## muscle 比对

用法：

```shell
muscle -align seq.fasta -output aln.fasta
```

`-align`需要比对的序列  
`-output`比对结果输出文件名，默认 fasta 格式

## mafft 比对

多序列比对，比 muscle 速度稍慢，但准确性更高。  
输入格式：fasta
用法：

```shell
mafft --auto in > out
```

`--auto`自动选择高速或者高精度

## iqtree 建树

IQtree 基于最大似然法  
输入文件格式：所有序列一个 fasta 文件，每条序列以`>`开头切该行无空格和标点。其他格式也支持，自动转换为 Phy 格式。

### 简单构树

```shell
iqtree -s sample.phy
```

`-s`后跟多序列比对结果。生成两个文件，`.phy.iqtree`和`.phy.treefile`，前者记录构树信息，后者记录进化树的 newick 结果。  
iqtree 运行中保存每一步成功运行的结果，中断的话会从断点重新开始。需要从头开始时加上参数`-redo`

### 选择替代模型

```shell
iqtree -s sample.phy -m MFP
```

`-m MFP`自动测试选择最优替代模型，多输出一个文件`.phy.model`记录所有模型的似然信息。  
`-m 最优模型`已知最优模型，直接构树。  
`-m MF`参数，只看最优替代模型是什么，不构树。加上`-mtree`检查所有可用模型。

`-m`可以指定替代模型、状态频率、速率异质性类型。用法`-m MODEL+FreqType+RateType`  
MODEL：
![MODEL](/pic/iqtree_model.webp "iqtree_model")

FresType:  
![FreqType](/pic/iqtree_FreqType.webp "iqtree_FreqType")

RateType:
![RateType](/pic/iqtree_RateType.webp "iqtree_RateType")

### 超快 bootstrap 检验

```shell
iqtree -s sample.phy -m TIM2+I+G -bb 1000
```

`-bb 1000`快速 BS 法 1000 次，输出文件`.phy.iqtree`中增加一个部分的内容`MAXIMUM LIKELIHOOD TREE`，记录了具体 BS 结果。
另外多输出三个文件，`example.phy.contree`，`example.phy.splits`，`example.phy.splits.nex`
但是，快速 BS 法与常规 BS 不同，会高估 bootstrap 值，可以加上参数`-bnni`  
`-b`常规 bs 法

其他参数：

- `-alrt`启用 SH-aLRT 检验，≥1000，检验树的拓扑结构可信度，推荐大数据运算时使用
- `-o`指定外群 outgroup
- `redo`覆盖之前运行成功后已经生成的文件
- `pre`设置输出文件前缀和路径，/root/Phylogenetic_trees/iqtree1，在/root/Phylogenetic_trees/路径下生成结果文件，文件名称均为 iqtree1.XXX

对于结果，iqtree 建议`SH-aLRT >= 80%`和 `UFboot >= 95%`比较可信。

## fasttree 建树

FastTree 基于最大似然法，运行速度快，支持几百万条序列建树。  
核酸模型：JC、GTR，默认 JC  
蛋白模型：JTT、LG、WAG，默认 JTT  
输入文件格式：fasta 或 phy  
生成文件：newick 格式  
用法：

```shell
# 快速构树（核酸）
fasttree  -gtr -nt nucleotides_seq.fas > tree.nwk
```

ps：  
`nucleotides_seq.fas`是以比对且修剪好的 fasta 格式文件。  
fasta 输入文件前缀名称与 nwk 一致。  
`-gtr`使用 gtr 模型，默认是 JC  
`-nt`指定核酸对比，默认蛋白

```shell
# 比对后蛋白构树，默认JTT+CAT
# alignment_file为比对结果文件， tree_file为输出文件名
fasttree < alignment_file > tree_file
fasttree alignment_file > tree_file
```

ps：  
使用`-wag`、`-lg`参数指定建树模型。默认用蛋白序列，如果要用核酸，使用`-nt`参数，`-gtr`是核酸的模型。

## 链霉菌

分类拉丁文后缀：

> 门 Phylum：-bacteria / -mycota
>
> 纲 class：-ia
>
> 目 order：-ales
>
> 科 family：-aceae

链霉菌（Streptomycetaceae）属放线菌门/纲。放线菌（Actinobacteria）属于革兰氏阳性菌。  
[Kitasatosporales](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?mode=Undef&id=85011&lvl=3&lin=f&keep=1&srchmode=1&unlock)目包含三个科：Allostreptomycetaceae、Carbonactinosporaceae、[Streptomycetaceae](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?mode=Tree&id=2062&lvl=3&lin=f&keep=1&srchmode=1&unlock)。[参考](https://lpsn.dsmz.de/order/kitasatosporales)

## SQL 结构化查询语言

SQL(Structured Query Language, 结构化查询语言)：用于管理关系数据库管理系统(Relational Database Management System, RDBMS)，通过一系列的语句和命令来执行数据定义、数据查询、数据操作和数据控制等功能,包括数据插入、查询、更新和删除，数据库模式创建和修改，以及数据访问控制。   
RDBMS 中的数据存储在被称为表的数据库对象中。表是相关的数据项的集合，它由列和行组成。

