### antismash用法
#### antismash运行流程

- 输入基因组序列
- 基因注释+BGC预测
> antismash对输入序列进行基因注释，识别蛋白基因。在寻找可能是次级代谢产物合成途径的BGC(NRPS、PKS等)
- BGC识别
> 预测识别出的BGC可能合成的代谢产物，并与已知数据库比对（MIBiG等）
- BGC结果
> 预测的BGC包括：已知BGC（与数据库比对相似）、新型BGC（数据库中不完全匹配）、可能的BGC（结构完整但功能未知）

#### 基本命令格式
```shell
conda activate antismash
antismash <input.fasta/genbank/EMBL> --output-dir <output_directory>
```
#### 参数
- 基础参数
> `-t {bacteria,fungi}`：输入序列的分类学层次，默认bacteria
>
> `-c 数字`：并行使用的cpu数量

- 附加分析
> `--fullhmmer`：使用Pfam（蛋白质数据库）进行全基因组级别的HMMer分析
>
> `--cassis`：基于motif（基序）的预测方法，识别次级代谢基因簇SMGCs
>
> `--clusterhmmer`：使用Pfam进行局部HMMer分析，只分析BGC相关
>
> `--tigrfam`：使用TIGRFAMs数据库进行基因簇功能注释
>
> `--asf`运行active  site finder活性位点预测，分析关键酶的活性位点
>
> `--cc-mibig`：将预测的基因簇和MIBiG数据库比较。识别与已知生物活性化合物合成相关的基因簇，帮助推测BGC的可能功能和代谢产物
>
> `--cb-general`：将识别出的BGC与antismash内置BGC数据库进行比对。内置数据库中包含大量其他预测得到的潜在BGC，不止已知的天然产物
>
> `--cb-subclusters`：将预测的BGC片段与已知的subcluster比对，识别前体合成模块
>
> `--cb-knownclusters`：将BGC只与MIBiG数据库中已知基因簇比对，发现相似的BGC
>
> `--pfam2go`：运行Pfam-Gene Ontology映射模块，将蛋白家族信息转换为GO术语

ps：预测的BGC是指从输入的全基因组序列中，antismash识别预测出的可能负责次级代谢产物合成的基因簇。   
cc：cluster compare。完整BGC与MIBiG中BGC进行全局对比，检查输入数据中是否包含MIBiG中的基因簇，考虑序列相似性、基因簇的布局结构和基因功能，综合评估相似性    
cb：cluster blast。BGC与MIBiG中BGC进行序列比对，比较基因间相似性和位置关系    

### 结果html
- overview-table
打开html，overview界面。在safiri中打开开发界面，`元素`窗口中，如下图
![antismash_overview](/pic/antismash_overview.png "antismash_overview")
解释：   
`<table class="region-table">`中所有内容对应上面的`Region	Type	From	To	Most similar known cluster	Similarity`整个表格    
点击左边灰色箭头可以查看每一个单元格的内容。    
`<table class="region-table">`从外到内结构层次为：
> `thead`：表格标题
> 
> `tbody`：表格主体部分，包含tr和td。与`thead`同一级别。
> 
> `tr`：table row表格的行，包含td。
> 
> `td`：table data表格数据单元格，存放具体内容
> 
> `a`：anchor超链接，包含链接的名称（纯文本），点击可跳转链接

- 每个region中的MIBiG表
在overview界面，任意选择一个region，同样打开开发界面元素窗口，如下图
![antismash_mibig](/pic/antismash_mibig.png "antismash_mibig")
解释：   
> `<div class="page" id="r1c1"`：整个1.1region的页面、页面id
>
> `<div class="body-details-section r1c1-MIBiG-cluster-compare"`：其中的MIBiG栏
>
> `<div id="comparison-MIBiG-r1c1-ProtoToRegion_RiQ" class="comparison-container comparison-container-active">…</div>`：MIBiG栏中表格部分
>
> `<table class="cc-heat-table>"`：表格部分。包括tr、td、a三部分内容

ps：得到的MIBiG-table中菌株名称和cluster很重要，分别在第一列和第二列


### 产物预测
#### overview预测
基于泛基因组数据库（Pfam/TIGRFAMs/SMART），对整个基因组分析，识别并对BGCs分类，再根据BGCs类型预测代谢产物类型；再利用（NRPS或PKS）中模块预测产物
similarity score计算：提取BGC的核心合成酶基因，再用HMM或blast在数据库查找相似BGC，并基于HMM评分
HMM隐马尔可夫概率模型：由状态states和转移概率transition probabilities组成
> 隐状态hidden states：代表基因/蛋白质的不同功能域
>
> 观察序列observed sequence：需要分析的BGC序列
>
> 状态转移概率transition probabilities：不同功能域之间的转换概率

#### 每个region中MIBiG预测
基于MIBiG数据库（仅收录实验验证的代谢产物的BGC），用比对工具将region与MIBiG数据库中已知BGC比对（包括基因序列、结构、核心酶功能相似性等），对基因组中单独的一个region进行分析预测代谢产物，比overview更加准确详细
similarity score计算：基于MIBiG数据库中已知BGC，通过diamond工具和clustercompare算法局部蛋白序列比对，


### 结果js文件
存储四个对象。`recordData`、`all_regions`、`details_data`、`resultsdata`


- `details_data`中：
> `r1c?`:每个region
> 
> `orfs`：开放阅读框，包含每个基因的信息，包括id、sequence、domains、modules
>
> `domains`：包含一个或多个domain信息，再细分为一个域一个对象，包含位置、序列等详细信息
>
> 一个domain对象中，包括了type、start、end、predictions、数据库link、sequence、dna_sequence、abbreviation、html_class等详细信息\

`details_data`部分内容如图：
![regions_js_picture](/pic/regions_js.png "regions_js")
