# Clinvar db for ANNOVAR

[![如果显示failed，请谨慎使用](https://github.com/ryuzheng/clinvar_db_for_annovar/workflows/Update%20Clinvar%20database%20for%20ANNOVAR/badge.svg)](https://github.com/ryuzheng/clinvar_db_for_annovar/actions)
[![release](https://img.shields.io/github/v/release/ryuzheng/clinvar_db_for_annovar?include_prereleases)](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)
[![Github All Releases](https://img.shields.io/github/downloads/ryuzheng/clinvar_db_for_annovar/total.svg)](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)

使用 Github Actions 自动更新 ANNOVAR 所使用的 ClinVar 数据库。本 repo 使用
[mobidic/update_annovar_db](https://github.com/mobidic/update_annovar_db) 的脚本，
对 [ClinVar 的 VCF 文件](https://ftp.ncbi.nlm.nih.gov/pub/clinvar/) 进行处理，
生成可直接用于 ANNOVAR 的注释数据库，同时支持 `hg19`（GRCh37）与 `hg38`（GRCh38）。

下载地址: [https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)

## 重要说明（请务必阅读）

- **数据库文件只通过 Release 发布，不保存在仓库里。**
  生成的 `*.txt.gz`（ANNOVAR 数据库）与 `*.txt.idx.gz`（索引）作为 GitHub Release
  的附件发布。仓库中**不再包含** `Clinvar_build/` 目录及其数据文件，请始终从
  [Releases 页面](https://github.com/ryuzheng/clinvar_db_for_annovar/releases)
  下载。
- **只有数据真正变化时才发布新 Release。**
  ClinVar 约每周更新一次，而本仓库的 Action 每天运行两次。为避免产生大量内容重复的
  Release，脚本会比较本次生成数据与上一次 Release 的**内容**（解压后的 SHA-256），
  仅在数据确实发生变化时才创建新 Release；数据未变的运行不会产生新 Release。
- **历史版本仍然可以下载。** 需要某个日期对应的旧版本时，请在 Releases 页面按
  tag（`tag-YYYY-MM-DD`）选择并下载。

## Release 内容

每个 Release 固定包含 4 个文件：

| 文件 | 说明 |
| --- | --- |
| `hg19_clinvar_YYYYMMDD.txt.gz` | hg19 / GRCh37 的 ANNOVAR 数据库（主文件） |
| `hg19_clinvar_YYYYMMDD.txt.idx.gz` | hg19 的 ANNOVAR 索引（配合主文件使用） |
| `hg38_clinvar_YYYYMMDD.txt.gz` | hg38 / GRCh38 的 ANNOVAR 数据库（主文件） |
| `hg38_clinvar_YYYYMMDD.txt.idx.gz` | hg38 的 ANNOVAR 索引（配合主文件使用） |

- 其中 `YYYYMMDD` 为 **ClinVar 官方数据的版本日期**（来自 NCBI FTP），并非本仓库
  Release 的发布日期。
- `.txt.gz` 与 `.txt.idx.gz` 是**成对使用**的：前者是数据表，后者是其索引，ANNOVAR
  做区间注释时会用到索引，两者都要下载。

## 下载方法

### 方法一：浏览器直接下载（最简单）

1. 打开 [Releases 页面](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)。
2. 在最新（标记为 `Latest`）的 Release 下方，找到 `Assets` 区域。
3. 点击所需文件（例如 `hg19_clinvar_20261108.txt.gz`）即可下载。

### 方法二：命令行下载「最新版本」

下面的脚本会自动取「最新 Release」的 tag，并下载 hg19 的数据库与索引：

```bash
REPO="ryuzheng/clinvar_db_for_annovar"
# 取最新 Release 的 tag（例如 tag-2026-11-09）
TAG=$(curl -s "https://api.github.com/repos/${REPO}/releases/latest" \
      | grep -o '"tag_name": *"[^"]*"' | head -1 | sed 's/.*"\([^"]*\)"$/\1/')
echo "latest tag = ${TAG}"

# 从最新 Release 的 assets 列表中，取出 hg19 的文件名（含数据版本日期）
FILES=$(curl -s "https://api.github.com/repos/${REPO}/releases/tags/${TAG}" \
        | grep -o '"name": *"[^"]*hg19[^"]*\.gz"' | sed 's/.*"\([^"]*\)"$/\1/')

for f in ${FILES}; do
  echo "downloading ${f}"
  wget "https://github.com/${REPO}/releases/download/${TAG}/${f}"
done
```

> 也可以把上面最后的 `wget` 换成 `curl -LO`，或使用 GitHub CLI：
> `gh release download --repo ryuzheng/clinvar_db_for_annovar --pattern 'hg19_*.gz'`。

## 使用方法（在 ANNOVAR 中注释 ClinVar）

ANNOVAR 的注释数据库放在其 `humandb/` 目录中（也可用 `-hp` 指定其他目录）。把下载的
文件**解压**后放进去即可：

```bash
# 解压（会得到 hg19_clinvar_20261108.txt 与 hg19_clinvar_20261108.txt.idx）
gunzip hg19_clinvar_20261108.txt.gz
gunzip hg19_clinvar_20261108.txt.idx.gz

# 放到 ANNOVAR 的 humandb 目录
mv hg19_clinvar_20261108.txt     /path/to/annovar/humandb/
mv hg19_clinvar_20261108.txt.idx /path/to/annovar/humandb/
```

然后即可用该数据库做注释（`-dbtype` 使用前缀 `clinvar_<日期>`）：

```bash
# 基于 hg19 的示例
table_annovar.pl your.vcf /path/to/annovar/humandb \
  -buildver hg19 \
  -out myanno \
  -remove \
  -protocol clinvar_20261108 \
  -operation f \
  -nastring . \
  -vcfinput
```

要点说明：

- `-buildver hg19`（或 `hg38`）必须与你下载的数据库版本、以及 your.vcf 的参考基因组
  **一致**。
- `-protocol` 填**去掉 `hg19_` 前缀、去掉 `.txt` 后缀**的文件名，即 `clinvar_20261108`
  （`_20261108` 来自文件名中的 ClinVar 数据版本日期，会随版本变化，请按实际文件名填）。
- `-operation f` 表示按数据库的字段做过滤/筛选注释。本数据库的字段为：

  | 列 | 含义 |
  | --- | --- |
  | `#Chr` | 染色体 |
  | `Start` | 起始位置（VCF POS） |
  | `End` | 终止位置（VCF POS，单碱基） |
  | `Ref` | 参考等位基因 |
  | `Alt` | 变异等位基因 |
  | `CLNALLELEID` | ClinVar 的 Allele ID |
  | `CLNDN` | 关联疾病名称（disease name） |
  | `CLNDISDB` | 疾病数据库引用（如 OMIM、MedGen） |
  | `CLNREVSTAT` | 审阅状态（review status） |
  | `CLNSIG` | 临床意义（clinical significance，如 Pathogenic、Benign 等） |

- 若只需注释而不筛选，可把 `-operation f` 改成 `-operation g`（按基因注释）。
- 注释结果中，ClinVar 相关字段会带 `clinvar_20261108` 前缀出现，可据此解析。

### 选择 hg19 还是 hg38？

根据你的 VCF 所用的参考基因组选择：GRCh37 / hg19 用 `hg19_*`，GRCh38 / hg38 用
`hg38_*`。两者不可混用。

## 常见问题（FAQ）

- **Q：为什么仓库里看不到数据文件？**
  A：为了避免仓库体积膨胀，数据只放在 Release 里，不提交进 git。请从 Releases 页面下载。
- **Q：为什么每天运行，却不是每天都有新 Release？**
  A：ClinVar 约每周更新一次。只有数据真正变化时才会产生新 Release，数据相同的运行会
  被跳过，所以不会刷出一堆重复 Release。
- **Q：`.txt.gz` 和 `.txt.idx.gz` 都要下载吗？**
  A：要。主文件是数据表，索引是 ANNOVAR 做区间注释所需的辅助文件，两者成对使用。
- **Q：`-protocol` 里到底填什么？**
  A：填文件名去掉 `hg19_`/`hg38_` 前缀、去掉 `.txt` 后缀的部分，例如文件
  `hg19_clinvar_20261108.txt` → `-protocol clinvar_20261108`。

## 关于仓库体积

> 说明：本仓库曾把每期生成的 ClinVar 数据库提交进 git 历史，导致历史体积累积过大。
> 现已改为「只发布 Release、不提交数据」，并对历史中 `Clinvar_build/` 下的数据文件
> 做了清理。因此**历史 commit / tag 的 SHA 已发生变化**；如果你曾 clone 过本仓库，
> 建议重新 clone 一次。清理后，完整 clone（含全部 tag）的体积已非常小，不影响正常
> 使用。

-----

Use Github Actions to automatically update the ClinVar database used by ANNOVAR.
This repo handles [ClinVar VCF files](https://ftp.ncbi.nlm.nih.gov/pub/Clinvar/)
using the [mobidic/update_annovar_db](https://github.com/mobidic/update_annovar_db)
script, producing ready-to-use ANNOVAR annotation databases for both `hg19`
(GRCh37) and `hg38` (GRCh38).

Download: [https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)

**Key notes:**

- **Database files are published only as GitHub Release assets, never committed to
  the repository.** Download them from the
  [Releases page](https://github.com/ryuzheng/clinvar_db_for_annovar/releases).
  Each release contains 4 files: `hg19_clinvar_YYYYMMDD.txt.gz`,
  `hg19_clinvar_YYYYMMDD.txt.idx.gz`, `hg38_clinvar_YYYYMMDD.txt.gz`,
  `hg38_clinvar_YYYYMMDD.txt.idx.gz` (`YYYYMMDD` is the ClinVar data version date).
- **A new release is created only when the ClinVar data actually changes.**
  ClinVar updates roughly weekly while the workflow runs twice a day; the workflow
  compares the decompressed content against the previous release and skips
  publishing when nothing changed, so there are no redundant releases.
- **Historical versions remain downloadable** — pick the desired `tag-YYYY-MM-DD`
  on the Releases page.

**Usage:** gunzip the `.txt.gz` and `.txt.idx.gz` into ANNOVAR's `humandb/` directory,
then annotate with e.g.:

```bash
table_annovar.pl your.vcf /path/to/annovar/humandb -buildver hg19 -out myanno \
  -remove -protocol clinvar_20261108 -operation f -nastring . -vcfinput
```

where `-protocol` is the file name without the `hg19_`/`hg38_` prefix and `.txt`
suffix, and `-buildver` must match both the database build and your VCF.
