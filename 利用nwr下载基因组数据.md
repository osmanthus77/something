# 利用 nwr 下载基因组数据

[参考链接](https://github.com/wang-q/genomes/blob/main/groups/Bacillus.md)

## 1 统计分类信息

```shell
mkdir ~/project/nwr/Bacillus
cd ~/project/nwr/Bacillus

# Bacillales下的科
nwr member Bacillales -r family |
    keep-header -- tsv-sort -k2,2

# 科下的所有分类水平的数量统计
nwr member \
    Bacillaceae Paenibacillaceae \
    Sporolactobacillaceae Thermoactinomycetaceae Alicyclobacillaceae \
    Planococcaceae Pasteuriaceae \
    Desulfuribacillaceae |
    tsv-summarize -H -g 3 --count

# 科下species group 和 species subgroup
nwr member \
    Bacillaceae Paenibacillaceae \
    Sporolactobacillaceae Thermoactinomycetaceae Alicyclobacillaceae \
    Planococcaceae Pasteuriaceae \
    Desulfuribacillaceae \
    -r "species group" -r "species subgroup" |
    tsv-select -f 1-3 |
    keep-header -- tsv-sort -k3,3 -k2,2
```

结果：

```
#tax_id	sci_name	rank	division
3076164	Abyssicoccaceae	family	Bacteria
186823	Alicyclobacillaceae	family	Bacteria
3120669	Anoxybacillaceae	family	Bacteria
186817	Bacillaceae	family	Bacteria
539003	Bacillales Family X. Incertae Sedis	family	Bacteria
186818	Caryophanaceae	family	Bacteria
539738	Gemellaceae	family	Bacteria
186820	Listeriaceae	family	Bacteria
186822	Paenibacillaceae	family	Bacteria
538998	Pasteuriaceae	family	Bacteria
3076165	Salinicoccaceae	family	Bacteria
186821	Sporolactobacillaceae	family	Bacteria
90964	Staphylococcaceae	family	Bacteria
186824	Thermoactinomycetaceae	family	Bacteria
```

```
rank	count
family	8
genus	208
species	46229
subspecies	33
no rank	278
strain	736
species group	4
species subgroup	2
biotype	1
```

```
#tax_id	sci_name	rank
1792192	Bacillus altitudinis complex	species group
86661	Bacillus cereus group	species group
653685	Bacillus subtilis group	species group
2044880	Paenibacillus sonchi group	species group
1938374	Bacillus amyloliquefaciens group	species subgroup
653388	Bacillus mojavensis subgroup	species subgroup
```

## 2 统计组装信息

组装信息分为两种：`RefSeq`（参考基因组）、`Genbank`（所有测序数据）  
这一步需要从 refseq 和 genbank 中生成得到包含下载信息的两个文件

```shell
mkdir -p ~/project/nwr/Bacillus/summary
cd ~/project/nwr/Bacillus/summary

# 统计属名
nwr member \
    Bacillaceae Paenibacillaceae \
    Sporolactobacillaceae Thermoactinomycetaceae Alicyclobacillaceae \
    Planococcaceae Pasteuriaceae \
    Desulfuribacillaceae \
    -r genus |
    sed '1d' |
    sort -n -k1,1 \
    > genus.list.tsv
wc -l genus.list.tsv
#      208 genus.list.tsv

# 整理刚才得到属名的 refseq 信息   统计每个 genus 下的 每个species的id、名称和数量
cat genus.list.tsv | cut -f 1 |
while read RANK_ID; do
    echo "
        SELECT
            species_id,
            species,
            COUNT(*) AS count
        FROM ar
        WHERE 1=1
            AND genus_id = ${RANK_ID}
        GROUP BY species_id
        HAVING count >= 1
        " |
        sqlite3 -separator $'\t' ~/.nwr/ar_refseq.sqlite
done |
    tsv-sort -k2,2 \
    > RS1.tsv

wc -l RS1.tsv
#    4096 RS1.tsv

# 整理刚才得到属名的 genbank 信息   统计每个 genus 下的 每个species的id、名称和数量
cat genus.list.tsv | cut -f 1 |
while read RANK_ID; do
    echo "
        SELECT
            species_id,
            species,
            COUNT(*) AS count
        FROM ar
        WHERE 1=1
            AND genus_id = ${RANK_ID}
        GROUP BY species_id
        HAVING count >= 1
        " |
        sqlite3 -separator $'\t' ~/.nwr/ar_genbank.sqlite
done |
    tsv-sort -k2,2 \
    > GB1.tsv

wc -l GB1.tsv
#    4443 GB1.tsv

# 分别统计 refseq 和 genbank 中组装信息的总数量
for C in RS GB; do
    for N in $(seq 1 1 10); do
        if [ -e "${C}${N}.tsv" ]; then
            printf "${C}${N}\t"
            cat ${C}${N}.tsv |
                tsv-summarize --sum 3
        fi
    done
done
# RS1	15892
# GB1	21237
```

解释：

> SELECT：指定筛选的列
> 
> FROM ar：指定筛选的库 ar 库是 nwr 中从 NCBI 中下载的数据的存储的数据库。
> 
> WHERE 1=1：WHERE:指定筛选的条件 1=1:始终为真，所以可以避免一些条件组合时可能出现的逻辑错误，方便后续拼接多个`AND`条件
> 
> GROUP BY species_id：指定了对查询结果按照 species_id 字段进行分组

## 3 下载数据

### 3.1 生成下载的数据表格

在`refseq`中存在的数据，在`genbank`中不需要再次下载

- 从 refseq sqlite数据库中筛选指定属的参考基因组信息

```shell
cd ~/project/nwr/Bacillus/summary

# 
echo "
.headers ON
    SELECT
        *
    FROM ar
    WHERE 1=1
        AND genus IN ('Bacillus', 'Staphylococcus', 'Listeria')
        AND refseq_category IN ('reference genome')
    " |
    sqlite3 -separator $'\t' ~/.nwr/ar_refseq.sqlite |
    tsv-join -H -d assembly_accession -f ../Bacteria.reference.tsv -k assembly_accession |
    tsv-select -H -f organism_name,species,genus,ftp_path,biosample,assembly_level,assembly_accession \
    > raw.tsv
```

- refseq 中筛选上一步骤中得到的species信息

```shell
SPECIES=$(
    cat RS1.tsv |
        cut -f 1 |
        tr "\n" "," |
        sed 's/,$//'
)

# 排除未知种和杂交种
echo "
    SELECT
        species || ' ' || infraspecific_name || ' ' || assembly_accession AS name,
        species, genus, ftp_path, biosample, assembly_level,
        assembly_accession
    FROM ar
    WHERE 1=1
        AND species_id IN ($SPECIES)
        AND species NOT LIKE '% sp.%'    #sp.未知种
        AND species NOT LIKE '% x %'     #x杂交种
    " |
    sqlite3 -separator $'\t' ~/.nwr/ar_refseq.sqlite \
    >> raw.tsv

# 添加未知种
echo "
    SELECT
        genus || ' sp. ' || infraspecific_name || ' ' || assembly_accession AS name,
        genus || ' sp.', genus, ftp_path, biosample, assembly_level,
        assembly_accession
    FROM ar
    WHERE 1=1
        AND species_id IN ($SPECIES)
        AND species LIKE '% sp.%'
    " |
    sqlite3 -separator $'\t' ~/.nwr/ar_refseq.sqlite \
    >> raw.tsv

# 单独保存 refseq 中 assembly_accession
cat raw.tsv |
    tsv-select -H -f "assembly_accession" \
    > rs.acc.tsv
```

- genbank 中筛选上一步骤中得到的species信息

```shell
SPECIES=$(
    cat GB1.tsv |
        cut -f 1 |
        tr "\n" "," |
        sed 's/,$//'
)

# 排除未知种和杂交种
echo "
    SELECT
        species || ' ' || infraspecific_name || ' ' || assembly_accession AS name,
        species, genus, ftp_path, biosample, assembly_level,
        gbrs_paired_asm
    FROM ar
    WHERE 1=1
        AND species_id IN ($SPECIES)
        AND species NOT LIKE '% sp.%'
        AND species NOT LIKE '% x %'
    " |
    sqlite3 -separator $'\t' ~/.nwr/ar_genbank.sqlite |
    tsv-join -f rs.acc.tsv -k 1 -d 7 -e \
    >> raw.tsv

# 添加未知种
echo "
    SELECT
        genus || ' sp. ' || infraspecific_name || ' ' || assembly_accession AS name,
        genus || ' sp.', genus, ftp_path, biosample, assembly_level,
        gbrs_paired_asm
    FROM ar
    WHERE 1=1
        AND species_id IN ($SPECIES)
        AND species LIKE '% sp.%'
    " |
    sqlite3 -separator $'\t' ~/.nwr/ar_genbank.sqlite |
    tsv-join -f rs.acc.tsv -k 1 -d 7 -e \
    >> raw.tsv

# 统计、检查，保证每行字段相同
cat raw.tsv |
    rgr dedup stdin |
    datamash check
#20770 lines, 7 fields
```

- 创建基于目标基因组的表格

```shell
# 物种名称标准化及缩写
cat raw.tsv |
    grep -v '^#' |
    rgr dedup stdin |
    tsv-select -f 1-6 |
    perl ../abbr_name.pl -c "1,2,3" -s '\t' -m 3 --shortsub |
    (echo -e '#name\tftp_path\tbiosample\tspecies\tassembly_level' && cat ) |
    perl -nl -a -F"," -e '
        BEGIN{my %seen};
        /^#/ and print and next;
        /^organism_name/i and next;
        $seen{$F[3]}++; # ftp_path
        $seen{$F[3]} > 1 and next;
        $seen{$F[6]}++; # abbr_name
        $seen{$F[6]} > 1 and next;
        printf qq{%s\t%s\t%s\t%s\t%s\n}, $F[6], $F[3], $F[4], $F[1], $F[5];
        ' |
    tsv-filter --or --str-in-fld 2:ftp --str-in-fld 2:http |
    keep-header -- tsv-sort -k4,4 -k1,1 \
    > Bacillus.assembly.tsv

# 统计检查
datamash check < Bacillus.assembly.tsv
# 20759 lines, 5 fields

# 去重
cat Bacillus.assembly.tsv |
    tsv-uniq -f 1 --repeated

# 删除无ftp链接的
cat Bacillus.assembly.tsv |
    tsv-filter --str-not-in-fld 2:ftp
```

### 3.2 正式下载之前检查计数

在下载基因组数据refseq和genbank之前，进行检查计数。分类水平及统计，genus属水平数量统计

```shell
cd ~/project/nwr/Bacillus

# 准备下载用到的脚本，生成三个sh脚本和一个species.tsv
nwr template summary/Bacillus.assembly.tsv \
    --count \
    --rank genus

# 运行strains.sh，生成菌株的分类tsv文件、数量统计tsv文件
bash Count/strains.sh

# 数量统计tsv文件转化为markdown格式
cat Count/taxa.tsv |
    rgr md stdin --fmt

# 运行rank.sh，生成genus count文件和genus list文件
bash Count/rank.sh
mv Count/genus.count.tsv Count/genus.before.tsv

# 转化为markdown格式
cat Count/genus.before.tsv |
    keep-header -- tsv-sort -k1,1 |
    tsv-filter -H --ge 3:50 |
    rgr md stdin --num
```
ps：
- `strains.sh脚本`获得每个菌株所属种、属、科、目、纲的信息并存入strains.taxon.tsv
- 
- `rank.sh脚本`将strains.taxon.tsv中的属的信息进行统计，分别计算每个属下的species种、strains菌株的数量

| item    |  count |
| ------- | -----: |
| strain  | 19,322 |
| species |  1,481 |
| genus   |    193 |
| family  |     10 |
| order   |      2 |
| class   |      2 |

| genus            | #species | #strains |
| ---------------- | -------: | -------: |
| Alicyclobacillus |       27 |       73 |
| Bacillus         |      122 |    12657 |
| Brevibacillus    |       31 |      293 |
| Cohnella         |       36 |       66 |
| Cytobacillus     |       19 |      152 |
| Fictibacillus    |       16 |       56 |
| Halobacillus     |       25 |       64 |
| Heyndrickxia     |       13 |      198 |
| Lysinibacillus   |       27 |      440 |
| Metabacillus     |       25 |       64 |
| Neobacillus      |       27 |      130 |
| Niallia          |        9 |       84 |
| Oceanobacillus   |       35 |      101 |
| Paenibacillus    |      318 |     2027 |
| Peribacillus     |       19 |      279 |
| Planococcus      |       29 |       84 |
| Priestia         |       11 |      687 |
| Psychrobacillus  |       11 |       60 |
| Rossellomorea    |        8 |       83 |
| Shouchella       |       12 |       85 |
| Solibacillus     |        9 |       55 |
| Sporosarcina     |       27 |      137 |
| Virgibacillus    |       32 |      125 |

### 3.3 下载并检查

```shell
cd ~/project/nwr/Bacillus

nwr template summary/Bacillus.assembly.tsv \
    --ass

# 下载
bash ASSEMBLY/rsync.sh

# 检查
bash ASSEMBLY/check.sh

# 对n50等信息进行检查。生成两个文件（n50.tsv and n50.pass.tsv）
bash ASSEMBLY/n50.sh 50000 500 500000
# n50.sh需要提前安装hnsm


cat ASSEMBLY/n50.tsv |
    tsv-filter -H --str-in-fld "name:_GCF_" |
    tsv-summarize -H --min "N50" --max "C" --min "S"
# N50_min	C_max	S_min
# 5966	1976	1073956

cat ASSEMBLY/n50.tsv |         
    tsv-summarize -H --quantile "N50:0.1,0.5" --quantile "C:0.5,0.9" --quantile "S:0.1,0.5" |
    datamash transpose
# N50_pct10	60097.8
# N50_pct50	330584
# C_pct50	52
# C_pct90	256
# S_pct10	3766486.8
# S_pct50	5159240

# collect收集每个sample相关信息。生成collect.tsv
bash ASSEMBLY/collect.sh

# finish检验筛选并保存。生成
bash ASSEMBLY/finish.sh


cp ASSEMBLY/collect.pass.tsv summary/
cat ASSEMBLY/counts.tsv |
    rgr md stdin --fmt
```

| #item            | fields |  lines |
| ---------------- | -----: | -----: |
| url.tsv          |      3 | 20,758 |
| check.lst        |      1 | 20,758 |
| collect.tsv      |     20 | 20,759 |
| n50.tsv          |      4 | 20,758 |
| n50.pass.tsv     |      4 |  8,921 |
| collect.pass.tsv |     23 |  8,921 |
| pass.lst         |      1 |  8,920 |
| omit.lst         |      1 |    643 |
| rep.lst          |      1 |    198 |
| sp.lst           |      1 |  1,327 |

ps：
> `rsync.sh`从NCBI上进行下载数据
>
> `check.sh`对下载的文件进行检验
>
> `n50.sh`进行n50等组装质量的检验。 后面的三个数字分别是 LEN_N50 表示 N50 值 ， N_CONTIG 表示 contig 的数量 ， LEN_SUM 表示序列总长度，即所有序列长度的总和，just在计算之前作为一个参考
> 
> `collect.sh`对每个sample的相关信息进行收集
>
> `finish.sh`对下载好的各sample文件的质量的检验和筛选，并生成相关文件来保存这些结果。生成counts.tsv、collect.pass.tsv和4个lst文件

注意：安装hnsm（手动编译）
```shell
cd ~/biosoft/hnsm
# 安装rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 下载hnsm
wget https://github.com/wang-q/hnsm/archive/refs/tags/v0.3.2.tar.gz
tar -xvf hnsm-0.3.2.tar.gz
cd hnsm-0.3.2

# 编译中用到rust nightly版本
rustup install nightly
rustup override set nightly
cargo build --release

# 编译后生成taeget/release文件，hnsm在该文件中
# 添加环境变量
echo 'export PATH="$HOME/biosoft/hnsm/hnsm-0.3.2/target/release:$PATH"' >> ~/.bashrc
```
