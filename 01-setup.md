---
source: Rmd
title: イントロダクションとセットアップ
teaching: XX
exercises: XX
---



::::::::::::::::::::::::::::::::::::::: objectives

- 受講者がこのレッスンの内容を正確に再現するために、正しいバージョンのRを使用していることを確認してください。
- このレッスンのサンプルファイルをダウンロードしてください。

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- このレッスンのために正しいバージョンのRを使用していますか？
- なぜ私のRのバージョンが重要なのですか？
- このレッスンで使用されるファイルをどうやって取得しますか？

::::::::::::::::::::::::::::::::::::::::::::::::::

## Rのバージョン

このレッスンはR version 4.4.3 (2025-02-28)を使用して開発およびテストされました。

RStudioを起動し、あなたが4.4.xのRバージョンを使用しているか確認してください。xは任意のパッチバージョン、例えば4.4.3です。


``` r
R.version.string
```

``` output
[1] "R version 4.4.3 (2025-02-28)"
```

これは重要です。なぜなら、Bioconductorは現在のセッションで実行されているRのバージョンを使用して、現在のRセッションに関連付けられたRライブラリにインストールできるBioconductorパッケージのバージョンを決定するからです。
このレッスンに従いながら異なるバージョンのRを使用すると、予期しない結果が生じる可能性があります。

## ファイルをダウンロードする

このレッスンのいくつかのエピソードは、参加者がダウンロードする必要があるサンプルファイルに依存しています。

以下のコードを実行して、現在の作業ディレクトリに`data`というフォルダーをプログラムで作成し、そのフォルダー内にレッスンファイルをダウンロードします。


``` r
dir.create("data", showWarnings = FALSE)
download.file(
    url = "https://raw.githubusercontent.com/Bioconductor/bioconductor-teaching/master/data/TrimmomaticAdapters/TruSeq3-PE-2.fa", 
    destfile = "data/TruSeq3-PE-2.fa"
)
download.file(
    url = "https://raw.githubusercontent.com/Bioconductor/bioconductor-teaching/master/data/ActbGtf/actb.gtf", 
    destfile = "data/actb.gtf"
)
download.file(
    url = "https://raw.githubusercontent.com/Bioconductor/bioconductor-teaching/master/data/ActbOrf/actb_orfs.fasta", 
    destfile = "data/actb_orfs.fasta"
)
download.file(
  url = "https://raw.githubusercontent.com/Bioconductor/bioconductor-teaching/devel/data/SummarizedExperiment/counts.csv",
  destfile = "data/counts.csv"
)
download.file(
  url = "https://raw.githubusercontent.com/Bioconductor/bioconductor-teaching/devel/data/SummarizedExperiment/gene_metadata.csv",
  destfile = "data/gene_metadata.csv"
)
download.file(
  url = "https://raw.githubusercontent.com/Bioconductor/bioconductor-teaching/devel/data/SummarizedExperiment/sample_metadata.csv",
  destfile = "data/sample_metadata.csv"
)
```

:::::::::::::::::::::::::::::::::::::::::  callout

## 注意

理想的には、参加者は新しい[RStudioプロジェクト][external-rstudio-project]を作成し、そのプロジェクトのサブディレクトリにレッスンファイルをダウンロードしたいと思うかもしれません。

RStudioプロジェクトを使用すると、作業ディレクトリがそのプロジェクトのルートディレクトリに設定されます。
その結果、コードはそのルートディレクトリに対して実行されるため、ファイルからデータをインポート/エクスポートするために絶対パスを使用する必要がない場合がよくあります。

::::::::::::::::::::::::::::::::::::::::::::::::::

[external-rstudio-project]: https://support.rstudio.com/hc/en-us/articles/200526207-Using-Projects

:::::::::::::::::::::::::::::::::::::::: keypoints

- 参加者は、このレッスンで説明されているBioconductorパッケージのバージョンをインストールし、その出力を正確に再現するには、正しいバージョンのRを使用する必要があります。
- このレッスンで使用されるファイルは、Rセッションから簡単にアクセスできるローカルパスにダウンロードする必要があります。

::::::::::::::::::::::::::::::::::::::::::::::::::


