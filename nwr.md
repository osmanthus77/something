# nwr 用法记录

[nwr 网页](https://github.com/wang-q/nwr)  
nwr(NCBI taxonomy, NeWick files and assembly Reports)是一个由 Rust 编写的命令行工具，可以用来处理 NCBI 分类信息，Newick 文件和组装报告文件

## 1 安装及初始化

- 使用`cargo`安装，详见[nwr 主页](https://github.com/wang-q/nwr)

```shell
cargo install nwr

# 下载`taxdump.tar.gz`和组装文件
nwr download

# 初始化分类信息库
nwr txdb

# 初始化组装信息数据库
nwr ardb
nwr ardb --genbank
```

- 测试
```shell
nwr member "Homo"

# 输出
#tax_id	sci_name	rank	division
9605	Homo	genus	Primates
2813598	unclassified Homo	no rank	Primates
2813599	Homo sp.	species	Primates
1425170	Homo heidelbergensis	species	Primates
9606	Homo sapiens	species	Primates
741158	Homo sapiens subsp. 'Denisova'	subspecies	Primates
63221	Homo sapiens neanderthalensis	subspecies	Primates
```

## 2 用法

### 2.1 概览

#### command

`nwr`主要命令 commands：
| commands | 解释 |
|-------------|------|
| download |下载 taxdump 和组装报告的最新版本 |
| txdb |初始化分类数据库(taxonomy database) |
| ardb |初始化组装数据库(assembly database) |
| append |向 TSV 文件追加(append)更上一级的分类字段 |
| common |输出指定内容的共同(common)进化树 |
| info |查询分类 ID（Taxonomy ID）或学名的信息(info) |
| kb |打印文档（知识库）(knowledge bases) |
| lineage |输出指定内容的系统发育谱系 |
| member |列出某一祖先的特定分类等级下的成员 |
| pl-condense |管道处理——基于分类法合并子树 |
| restrict |限定分类到某个祖先的后代范围 |
| seqdb |初始化序列数据库 |
| template |为系统发育基因组研究创建目录、数据和脚本 |
| data |Newick 数据相关命令 |
| ops |Newick 操作相关命令 |
| viz |Newick 可视化相关命令 |
| help |信息帮助页面（包括全部或者特定子命令） |

#### subcommand

命令及子命令分组，如下：
| group | command(+subcommand) |
|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| Database | download、txdb、ardb |
| Taxonomy | info、linage、member、append、restrict、common |
| Assembly | template、kb、seqdb |
| Newick | data: label、stat、distance <br>ops(operation): order、rename、replace、topo、subtree、prune、reroot、pl-condense <br>viz(visualization): indent、comment、tex |

### 2.2 具体命令用法

#### nwr data

- 用法：

```shell
nwr data <command>
```

- `nwr data`包含子命令：
  | command | 解释 |
  |----------|-------------------------------------------------|
  | lable | 获取 Newick 文件中的标签信息 |
  | stat | 统计 Newick 文件的相关信息 |
  | distance | 输出所有已命名节点之间的距离（TSV/Phylip 格式） |
  | help | 输出帮助信息 |

#### nwr ops

- 用法：

```shell
nwr ops <command>
```

- `nwr ops`包含子命令：
  | command | 解释 |
  |---------|--------------------------------------------|
  | order | 对 Newick 文件中的节点进行排序 |
  | rename | 重命名 Newick 文件中的已命名或未命名节点 |
  | replace | 替换 Newick 文件中的节点名称或注释 |
  | subtree | 提取子树 |
  | topo | 获取 Newick 文件的拓扑信息 |
  | prune | 从 Newick 文件中移除节点 |
  | reroot | 将根节点重新放置在指定节点与其父节点的中间 |
  | help | 输出帮助信息 |

#### nwr viz

- 用法：

```shell
nwr viz <command>
```

- `nwr viz`包含子命令：
  | command | 解释 |
  |---------|----------------------------------------------|
  | indent | 格式化 Newick 文件，使其更易阅读（缩进排列） |
  | comment | 为 Newick 文件中的节点添加注释 |
  | tex | 使用 LaTeX 可视化 Newick 进化树 |
  | help | 输出帮助信息 |

#### nwr member

列出某一祖先的特定分类等级下的成员

- 用法：

```shell
nwr member [OPTIONS] <terms>...
```

ps：`terms`为需要进行分类的名称或者 taxonomy id，数量不限

- 选项：

  > `-d`--dir，切换工作目录
  >
  > `-r`--rank，指定分类等级，如 genus、family 等
  >
  > `-o`--outfile，输出文件名，默认 stdout

- 示例
列出`Bacillales芽孢杆菌目`下的`family科`分类水平
```shell
nwr member Bacillales -r family
```

输出：
```
#tax_id	sci_name	rank	division
3120669	Anoxybacillaceae	family	Bacteria
3076165	Salinicoccaceae	family	Bacteria
3076164	Abyssicoccaceae	family	Bacteria
539738	Gemellaceae	family	Bacteria
539003	Bacillales Family X. Incertae Sedis	family	Bacteria
538998	Pasteuriaceae	family	Bacteria
186824	Thermoactinomycetaceae	family	Bacteria
186823	Alicyclobacillaceae	family	Bacteria
186822	Paenibacillaceae	family	Bacteria
186821	Sporolactobacillaceae	family	Bacteria
186820	Listeriaceae	family	Bacteria
186818	Caryophanaceae	family	Bacteria
186817	Bacillaceae	family	Bacteria
90964	Staphylococcaceae	family	Bacteria
```

#### nwr template

为系统发育基因组研究创建目录、数据和脚本

- 用法

```shell
nwr template [OPTIONS] <infiles>...
```
输入文件要求：制表符分隔的.assembly.tsv文件，至少包含五条信息
> `name`：命名方式为species + infraspecific_name + assembly_accession
>
> `ftp_path`：下载的ftp链接
>
> `biosample`：Sequence Read Archive（SRA）数据库中用于标识和索引基因组数据、转录组数据或其他类型的测序数据的唯一编号，以SAMN开头
>
> `species`：物种名称
>
> `assembly_level`：组装水平

- 选项：

> `--count`：Prepare Count/ materials，根据输入文件生成对应的物种文件（1个）和统计数量的脚本（3个）
>
> `--rank`：分类水平等级，genus、family、order、class
>
> `--ass`：Prepare ASSEMBLY/ materials，根据输入文件生成对应的下载链接文件（1个）和脚本（6个）