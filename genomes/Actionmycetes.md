# 下载 *Actionmycetes* 基因组数据
分歧杆菌目( *Mycobacteriales* )通常是病原菌，下载基因组数据时排除掉

## 1 统计分类信息

```shell
nwr member Actinomycetes -r order |
    keep-header -- tsv-sort -k2,2 |
    rgr md stdin --num

nwr member \
    Acidothermales Actinomycetales Actinopolysporales Bifidobacteriales \
    'Candidatus Actinomarinales' 'Candidatus Nanopelagicales' \
    Catenulisporales Cryptosporangiales Frankiales Geodermatophilales \
    Glycomycetales Jatrophihabitantales Jiangellales Kineosporiales \
    Kitasatosporales Micrococcales Micromonosporales Motilibacterales \
    Nakamurellales Propionibacteriales Pseudonocardiales \
    Sporichthyales Streptosporangiales |
    tsv-summarize -H -g 3 --count |
    rgr md stdin --fmt

nwr member \
    Acidothermales Actinomycetales Actinopolysporales Bifidobacteriales \
    'Candidatus Actinomarinales' 'Candidatus Nanopelagicales' \
    Catenulisporales Cryptosporangiales Frankiales Geodermatophilales \
    Glycomycetales Jatrophihabitantales Jiangellales Kineosporiales \
    Kitasatosporales Micrococcales Micromonosporales Motilibacterales \
    Nakamurellales Propionibacteriales Pseudonocardiales \
    Sporichthyales Streptosporangiales \
    -r "species group" -r "species subgroup" |
    tsv-select -f 1-3 |
    keep-header -- tsv-sort -k3,3 -k2,2 |
    rgr md stdin --num

```

| #tax_id | sci_name                   | rank  | division |
| ------: | -------------------------- | ----- | -------- |
| 1643683 | Acidothermales             | order | Bacteria |
|    2037 | Actinomycetales            | order | Bacteria |
|  622450 | Actinopolysporales         | order | Bacteria |
|   85004 | Bifidobacteriales          | order | Bacteria |
| 1389450 | Candidatus Actinomarinales | order | Bacteria |
| 2039638 | Candidatus Nanopelagicales | order | Bacteria |
|  414714 | Catenulisporales           | order | Bacteria |
| 2495577 | Cryptosporangiales         | order | Bacteria |
|   85013 | Frankiales                 | order | Bacteria |
| 1643682 | Geodermatophilales         | order | Bacteria |
|   85014 | Glycomycetales             | order | Bacteria |
| 2805415 | Jatrophihabitantales       | order | Bacteria |
| 1217098 | Jiangellales               | order | Bacteria |
|  622452 | Kineosporiales             | order | Bacteria |
|   85011 | Kitasatosporales           | order | Bacteria |
|   85006 | Micrococcales              | order | Bacteria |
|   85008 | Micromonosporales          | order | Bacteria |
| 2793120 | Motilibacterales           | order | Bacteria |
|   85007 | Mycobacteriales            | order | Bacteria |
| 1643684 | Nakamurellales             | order | Bacteria |
|   85009 | Propionibacteriales        | order | Bacteria |
|   85010 | Pseudonocardiales          | order | Bacteria |
| 2495578 | Sporichthyales             | order | Bacteria |
|   85012 | Streptosporangiales        | order | Bacteria |

| rank             |  count |
| ---------------- | -----: |
| order            |     23 |
| no rank          |    474 |
| species          | 72,026 |
| family           |     51 |
| genus            |    442 |
| strain           |  1,025 |
| subspecies       |    202 |
| species group    |     20 |
| suborder         |      1 |
| clade            |      6 |
| species subgroup |     10 |
| varietas         |      1 |
| isolate          |      1 |

| #tax_id | sci_name                              | rank             |
| ------: | ------------------------------------- | ---------------- |
| 2893675 | Actinopolyspora alba group            | species group    |
| 2893673 | Amycolatopsis japonica group          | species group    |
| 2893674 | Amycolatopsis methanolica group       | species group    |
| 2893671 | Prauserella coralliicola group        | species group    |
| 2893672 | Prauserella salsuginis group          | species group    |
| 1477431 | Streptomyces albidoflavus group       | species group    |
| 2867120 | Streptomyces albogriseolus group      | species group    |
| 1852274 | Streptomyces albus group              | species group    |
| 2867194 | Streptomyces althioticus group        | species group    |
| 2838335 | Streptomyces aurantiacus group        | species group    |
| 1481185 | Streptomyces cinnamoneus group        | species group    |
| 2849069 | Streptomyces diastaticus group        | species group    |
| 2867193 | Streptomyces griseoincarnatus group   | species group    |
|  629295 | Streptomyces griseus group            | species group    |
|  661169 | Streptomyces melanosporofaciens group | species group    |
| 2838332 | Streptomyces phaeochromogenes group   | species group    |
| 2894086 | Streptomyces pseudogriseolus group    | species group    |
| 2867164 | Streptomyces rochei group             | species group    |
| 2867121 | Streptomyces violaceoruber group      | species group    |
| 2839105 | Streptomyces violaceusniger group     | species group    |
| 1482558 | Streptomyces albovinaceus subgroup    | species subgroup |
| 1482561 | Streptomyces anulatus subgroup        | species subgroup |
| 1482564 | Streptomyces atroolivaceus subgroup   | species subgroup |
| 1482566 | Streptomyces bacillaris subgroup      | species subgroup |
| 1482592 | Streptomyces fimicarius subgroup      | species subgroup |
| 1482593 | Streptomyces flavovirens subgroup     | species subgroup |
| 1482596 | Streptomyces griseus subgroup         | species subgroup |
| 1482599 | Streptomyces halstedii subgroup       | species subgroup |
| 1482601 | Streptomyces microflavus subgroup     | species subgroup |
| 1482603 | Streptomyces puniceus subgroup        | species subgroup |

## 2 统计组装信息

组装信息分为两种：`RefSeq`（参考基因组）、`Genbank`（所有测序数据）   
这一步需要从 refseq 和 genbank 中生成得到包含下载信息的两个文件

```shell
mkdir -p ~/project/nwr/Actionmycetes/summary
cd ~/project/nwr/Actionmycetes/summary

nwr member \
    Acidothermales Actinomycetales Actinopolysporales Bifidobacteriales \
    'Candidatus Actinomarinales' 'Candidatus Nanopelagicales' \
    Catenulisporales Cryptosporangiales Frankiales Geodermatophilales \
    Glycomycetales Jatrophihabitantales Jiangellales Kineosporiales \
    Kitasatosporales Micrococcales Micromonosporales Motilibacterales \
    Nakamurellales Propionibacteriales Pseudonocardiales \
    Sporichthyales Streptosporangiales \
    -r genus |
    sed '1d' |
    sort -n -k1,1 \
    > genus.list.tsv

wc -l genus.list.tsv 
#442 genus.list.tsv

#refseq信息
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

#genbank信息
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

wc -l RS*.tsv GB*.tsv
# 13542 RS1.tsv
# 14348 GB1.tsv

#统计总数
for C in RS GB; do
    for N in $(seq 1 1 10); do
        if [ -e "${C}${N}.tsv" ]; then
            printf "${C}${N}\t"
            cat ${C}${N}.tsv |
                tsv-summarize --sum 3
        fi
    done
done
#RS1	26874
#GB1	38096
```

## 3 下载数据

### 3.1 生成下载的数据表格

- 统计模式物种信息

从genus.list.tsv中得到属名的标识号，再用SQL筛选模式物种信息存入reference.tsv

```shell
cd ~/project/nwr/Actionmycetes/summary

GENUS=$(
    cat genus.list.tsv|
        cut -f 1 |
        tr "\n" "," | 
        sed 's/,$//'
)

echo "
.headers ON

    SELECT
        *
    FROM ar
    WHERE 1=1
        AND genus_id IN ($GENUS)
        AND refseq_category IN ('reference genome')
    " |
    sqlite3 -separator $'\t' ~/.nwr/ar_refseq.sqlite \
    > reference.tsv

wc -l reference.tsv 
# 3626 reference.tsv

cat reference.tsv |
    tsv-select -H -f organism_name,species,genus,ftp_path,biosample,assembly_level,assembly_accession \
    > raw.tsv
```

- refseq 中筛选上一步骤中得到的species信息

```shell
cd ~/project/nwr/Actionmycetes/summary

# refseq
SPECIES=$(
    cat RS1.tsv |
        cut -f 1 |
        tr "\n" "," |
        sed 's/,$//'
)

# 排除未知种sp.和杂交种x
echo "
    SELECT
        species || ' ' || infraspecific_name || ' ' || assembly_accession AS name,
        species, genus, ftp_path, biosample, assembly_level,
        assembly_accession
    FROM ar
    WHERE 1=1
        AND species_id IN ($SPECIES)
        AND species NOT LIKE '% sp.%'
        AND species NOT LIKE '% x %'  
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
cd ~/project/nwr/Actionmycetes/summary
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
#41722 lines, 7 fields
```

- 创建基于目标基因组的表格

```shell
cd ~/project/nwr/Actionmycetes/summary
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
    > Actionmycetes.assembly.tsv

# 统计检查
datamash check < Actionmycetes.assembly.tsv
# 38074 lines, 5 fields

# 去重
cat Actionmycetes.assembly.tsv |
    tsv-uniq -f 1 --repeated

# 删除无ftp链接的
cat Actionmycetes.assembly.tsv |
    tsv-filter --str-not-in-fld 2:ftp
```

### 3.2 正式下载之前检查计数

在下载基因组数据refseq和genbank之前，进行检查计数。分类水平及统计，genus属水平数量统计

```shell
cd ~/project/nwr/Actionmycetes

# 准备下载用到的脚本，生成三个sh脚本和一个species.tsv
nwr template summary/Actionmycetes.assembly.tsv \
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
`Count/taxa.tsv`文件内容：
| item    |  count |
| ------- | -----: |
| strain  | 37,934 |
| species |  4,057 |
| genus   |    414 |
| family  |     52 |
| order   |     23 |
| class   |      1 |

`Count/genus.before.tsv`部分内容：
| genus              | #species | #strains |
| ------------------ | -------: | -------: |
| Actinomadura       |       71 |      176 |
| Actinomyces        |       48 |      496 |
| Actinoplanes       |       36 |      153 |
| Actinotignum       |        5 |       76 |
| Aeriscardovia      |        2 |       53 |
| Aeromicrobium      |       26 |      192 |
| Agromyces          |       45 |      147 |
| Amycolatopsis      |       89 |      313 |
| Aquiluna           |        2 |      115 |
| Arachnia           |        4 |       58 |
| Arcanobacterium    |       13 |       76 |
| Arthrobacter       |       82 |      703 |
| Bifidobacterium    |      108 |     7026 |
| Blastococcus       |       20 |       74 |
| Brachybacterium    |       34 |      141 |
| Brevibacterium     |       45 |      276 |
| Cellulomonas       |       48 |      211 |
| Cellulosimicrobium |        9 |       91 |
| Clavibacter        |       11 |      360 |
| Cryobacterium      |       35 |      160 |
| Curtobacterium     |       15 |      347 |
| Cutibacterium      |        8 |     2264 |
| Demequina          |       27 |       64 |
| Frankia            |       13 |       86 |
| Frigoribacterium   |        3 |       61 |
| Gardnerella        |        7 |      388 |
| Glutamicibacter    |       14 |      105 |
| Glycomyces         |       25 |       50 |
| Isoptericola       |       14 |       68 |
| Janibacter         |       14 |       70 |
| Kitasatospora      |       36 |      408 |
| Kocuria            |       23 |      285 |
| Kribbella          |       38 |      124 |
| Leifsonia          |       15 |      112 |
| Lentzea            |       26 |       64 |
| Leucobacter        |       35 |      148 |
| Marmoricola        |        2 |       77 |
| Microbacterium     |      161 |     1235 |
| Microbispora       |       15 |       79 |
| Micrococcus        |       12 |      531 |
| Micromonospora     |      124 |      888 |
| Mobiluncus         |        6 |       68 |
| Modestobacter      |       10 |       50 |
| Nakamurella        |       13 |       50 |
| Nesterenkonia      |       29 |       61 |
| Nocardioides       |      154 |      620 |
| Nocardiopsis       |       47 |      173 |
| Nonomuraea         |       67 |      288 |
| Paenarthrobacter   |        8 |      144 |
| Phycicoccus        |        9 |       56 |
| Pontimonas         |        2 |       98 |
| Propionibacterium  |        6 |      225 |
| Pseudarthrobacter  |       19 |      164 |
| Pseudoclavibacter  |        9 |       51 |
| Pseudonocardia     |       54 |      215 |
| Rathayibacter      |       12 |      182 |
| Rhodoluna          |        4 |      124 |
| Rothia             |       15 |      391 |
| Saccharopolyspora  |       34 |       88 |
| Saccharothrix      |       22 |       55 |
| Salinibacterium    |        7 |       56 |
| Schaalia           |       13 |      112 |
| Streptomyces       |      827 |    11828 |
| Streptosporangium  |       22 |      106 |
| Trebonia           |        2 |       77 |
| Trueperella        |        7 |      171 |
| Varibaculum        |        6 |       59 |