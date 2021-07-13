# Clinvar db for ANNOVAR

[![如果显示failed，请谨慎使用](https://github.com/ryuzheng/clinvar_db_for_annovar/workflows/Update%20Clinvar%20database%20for%20ANNOVAR/badge.svg)](https://github.com/ryuzheng/clinvar_db_for_annovar/actions)
[![release](https://img.shields.io/github/v/release/ryuzheng/clinvar_db_for_annovar?include_prereleases)](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)
[![Github All Releases](https://img.shields.io/github/downloads/ryuzheng/clinvar_db_for_annovar/total.svg)](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)

使用Github Actions自动更新ANNOVAR所使用的Clinvar数据库。Action的运行频率为每3天一次。

本repo使用[mobidic/update_annovar_db](https://github.com/mobidic/update_annovar_db)的脚本，对[Clinvar的VCF文件](https://ftp.ncbi.nlm.nih.gov/pub/clinvar/)进行处理。

下载地址: [https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)

中国地区请使用[FastGit](https://doc.fastgit.org/zh-cn/guide.html#release-%E5%92%8C%E6%BA%90%E7%A0%81%E5%AD%98%E6%A1%A3%E7%9A%84%E4%B8%8B%E8%BD%BD)或[jsDelivr](https://www.jsdelivr.com/features#gh)进行加速下载。

示例如下：

```
# Release
# 假设下载链接为https://github.com/ryuzheng/clinvar_db_for_annovar/releases/download/tag-2021-05-21/hg19_clinvar_20210517.txt.gz
wget https://download.fastgit.org/ryuzheng/clinvar_db_for_annovar/releases/download/tag-2021-05-21/hg19_clinvar_20210517.txt.gz

# 假设文件位置为Clinvar_build/hg38/hg38_clinvar_20210517.txt.gz
wget https://cdn.jsdelivr.net/gh/ryuzheng/clinvar_db_for_annovar/Clinvar_build/hg38/hg38_clinvar_20210517.txt.gz
```

-----

Use Github Actions to automatically update the Clinvar database used by ANNOVAR. The Action runs every three days.

This repo handles [Clinvar VCF files](https://ftp.ncbi.nlm.nih.gov/pub/Clinvar/) using the [mobidic/update_annovar_db](https://github.com/mobidic/update_annovar_db) script.

Download sites: [https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest](https://github.com/ryuzheng/clinvar_db_for_annovar/releases/latest)
