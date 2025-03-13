命令

### rsync

- `rsync`（remote synchronization）远程同步，同步文件/目录，可只传输文件中发生变化的部分  
  选项：`-a`（--archive）归档模式进行同步（递归并保留符号连接、文件权限、时间戳、属组信息、属主信息、设备文件和特殊文件）  
   `-v`（--verbose）详细输出  
   `-P`（--partial --progress），保留部分传输的文件（中断后可继续）、显示传输进度  
  `rsync -avP ftp.ncbi.nlm.nih.gov::refseq/release/plasmid/ RefSeq/`  
  从服务器`ftp.ncbi.nlm.nih.gov`的`refseq/release/plasmid/`目录同步文件到本地的`RefSeq/`目录。`::`是 rsync`协议的标识（使用 rsync 协议传输）

### gzip

- `gzip`压缩、解压缩
  选项：`-d`decompress 解压  
   `-c`stdout 解压后内容输出到标准输出。结合`>`指定输出保存文件  
   `-f`force 强制，输出文件已存在则覆盖而不提示

### tsv-utilities

- `tsv-filter`处理 tsv（tab-separated values），属于 tsv-utils 一部分。对 tsv 文件中数据基于字段值进行过滤
  选项：`--le`less than or equal to 小于等于。  
   `--gt`（greater than）大于  
   `2:2000`，2 表示列，2000 为阈值（字段值满足的小于等于的条件）  
   `--ff-str-ne 1:2`ff 字段 filed/str 字符串比较/ne 不相等 not equal，字段间字符串不相等过滤，第一列、第二列值不同时该行保留  
   `-d`设置字段分隔符，默认制表符\t。`-d" "`设置分隔符为空格  
   `--regex`匹配正则表达式 regular expression。  

- `tsv-join`将输入与`filter`文件中数据进行匹配
  选项：`-e`exclude 排除匹配的数据  
   `-f`--filter-file（必须）过滤条件

- `<(...)`进程替换，括号内命令的结果作为**临时文件**传递给前一命令
- `$(...)`命令替换，括号内命令的结果作为**字符串**传递

- MinHash 含义
  用哈希函数将元素映射成数字，从哈希值里找最小值，并只保留最小值，即 MinHash 签名，重复多次得到多个签名，再比较计算签名的相似度。  
  质粒中：将质粒序列拆分为多个 k-mer 形成一个 k-mer 集合，再用多个哈希函数处理每个质粒的 k-mer 集合，生成 MinHash 签名，比较计算质粒间的相似度  
  过滤后保留不相似的质粒（非冗余质粒）

### mash

- `mash sketch`计算 MinHash sktech，用于快速计算序列相似性。`Mash`工具之一
  用法：`mash sketch [options] <input> -o <output.msh>`  
  选项：`-k`k-mer 长度 21。`-s`sketch 哈希片段数量（默认 1000）。`-i`输入为 fasta（默认 fastq）。`-p`线程数。 `-o`输出。  
  `-s`sketch：mash 采用 MinMash 技术，从 k-mer 的哈希值中选出最小的-s 个（即 sketch），每个 sketch 最多保留 1000 个非冗余的 MinHash，用 sketch 代表基因组。

- `mash dist`计算两个 MinHash Sketch 之间的距离
  用`mash sketch`代表序列片段后，`mash dist`计算两个 MinHash 之间的相似性及 Mash 距离，衡量序列相似性。Mash 距离越接近 0 则相似性越高。

### cut

- `cut`文本中提取特定字段
  选项：`-f 1`field 第 1 列

### gsplit

- `gsplit`将文件分割成多个较小文件
  选项：`-l`line 每个输出文件包含行数 1000。`-a`输出文件的后缀长度 3。`-d`数字作为后缀（默认字母）。`-`输入是标准输入

### find

- `find`查找文件
  选项：`-maxdepth 1`查找深度 1，只查找 job 目录下文件。`-type f`查找文件的类型为普通文件 file。  
   `-name`文件名匹配格式。`"[0-9]??"`：`[0-9]`匹配一个数字，`??`两个任意字符；文件名为两个数字开头的文件。

### sort

- `sort`排序，按照字母或数字顺序
  选项：`-n`numeric 按数字大小排序
  `-r`reverse 降序（从大到小）

### perl

- `perl`运行 perl
  选项：`-n`循环处理输出的每一行，每一行被 perl 代码处理一次。不会自动输出结果（与`-p`区别）  
   `-l`line-ending processing 自动去掉每行的换行符并在输出是重新添加换行符  
   `-a`自动分割字段，并放入数组`@F`中，每个字段一个@F  
   `-F`设置分隔符，默认空格或制表符，`-F"\t"`指定分隔符为制表符  
   `-M`加载 perl 模块，`-MGraph::Undirected`加载模块 Graph::Undirected，创建无向图  
   `-e`执行后续 perl 代码  
   `-p`print loop 循环打印，自动读取输入对每一行都执行后续 perl 代码并自动输出结果

### sed

- `sed`流编辑器
  选项：`-e`允许在一个 sed 中执行多个编辑操作，如`sed -e 'command1' -e 'command2' filename`  
  用法：`sed '1d'`删除第一行

### xargs

- `xargs`将标准输入的数据作为参数传递给命令
  选项：`-I`insert mode 指定占位符，让 xargs**逐行**替换输入数据，（xargs 默认是所有输入数据合并处理）。

### parallel

- `parallel`并行处理
  选项：`--colsep`每列按顺序依次传递给后续命令  
   `{}`占位符，文件的完整路径
  `{/}`文件的目录路径，不包含文件名
  `{/.}`去掉扩展名的文件名（无路径）
  `{#}`当前任务的序号（从 1 开始）  
   `{.}`去掉扩展名的文件名（含路径）

### rg(repgrep)

- `rg`即`repgrep`快速文本搜索
  选项：`-F`固定字符串搜索（--fixed-strings），直接按字符匹配，而不作为正则表达式处理  
   `-l`只列出匹配到的文件名（--files-with-matches），而不显示文件内具体匹配的行的内容  
   `"{1}"`第一字段

### grep

- `grep`文本搜索匹配
  选项：`-v`--invert-match 反向匹配，保留没匹配上的
  `-x`--line-match 全行匹配，整行完全匹配才满需要求

### tr(translate)

- `tr`（translate）替换、删除、压缩字符
  选项：

### uniq

- `uniq`去掉相临的重复行，需要 sort 先排序，uniq 只能去掉连续相同的行

### paste

- `paste`合并多行文本
  选项：`-s`single 输入行和并转换为单行，默认 tab 分隔符
  `-d+`指定分隔符为`+`

### bc(basic calculator)

- `bc`basic calculator 精确计算

### bsub

- `bsub`向 LSF 任务调度系统提交任务
  LSF：loading sharing facility 负载共享设施，高性能计算集群的任务调度系统，管理、分配计算资源
  MPI：message passing interface 消息传递接口，一种并行计算框架
  选项：`-q`制定队列 queue，指定为`mpi`
  `-n`分配运行，多核心
  `-J`job 任务名称
  `-w`wait

### pup
