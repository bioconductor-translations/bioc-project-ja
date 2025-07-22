---
source: Rmd
title: Bioconductor パッケージのインストール
teaching: XX
exercises: XX
---



::::::::::::::::::::::::::::::::::::::: objectives

- BiocManager をインストールします。
- Bioconductor パッケージをインストールします。

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- Bioconductor パッケージをどのようにインストールしますか？
- インストールしたパッケージの新しいバージョンが利用可能かどうかを確認するにはどうすればよいですか？
- Bioconductor パッケージをどのように更新しますか？
- Bioconductor リポジトリから利用可能なパッケージの名前をどのように調べますか？

::::::::::::::::::::::::::::::::::::::::::::::::::

## BiocManager

*[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)* パッケージは Bioconductor パッケージ リポジトリへのエントリ ポイントです。
技術的には、これは CRAN リポジトリで配布される唯一の Bioconductor パッケージです。

Bioconductor パッケージを安全にインストールし、利用可能な更新を確認するための関数を提供します。

パッケージがインストールされると、`BiocManager::install()` 関数を使用して Bioconductor リポジトリからパッケージをインストールできます。
この関数は、Bioconductor リポジトリ内にパッケージが見つからない場合、他のリポジトリ (例: CRAN) からパッケージをインストールすることも可能です。

![](fig/bioc-install.svg){alt='BiocManager パッケージは CRAN リポジトリから利用可能で、Bioconductor リポジトリからパッケージをインストールするために使用されます。'}

**BiocManager パッケージは CRAN リポジトリから利用可能で、Bioconductor リポジトリからパッケージをインストールするために使用されます。**
基本 R パッケージ `utils` の `install.packages()` 関数を使用して、CRAN リポジトリで配布される *[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)* パッケージをインストールできます。
その後、`BiocManager::install()` 関数を使用して、Bioconductor リポジトリで利用可能なパッケージをインストールできます。
特に、`BiocManager::install()` 関数は、Bioconductor リポジトリでパッケージが見つからない場合、CRAN リポジトリにフォールバックします。

以下のコードを使用してパッケージをインストールします。


``` r
install.packages("BiocManager")
```

:::::::::::::::::::::::::::::::::::::::::  callout

### さらに進む

基本 R インストールの一部ではない多くのパッケージも、さまざまなリポジトリからパッケージをインストールするための関数を提供します。
たとえば:

- `devtools::install()`
- `remotes::install_bioc()`
- `remotes::install_bitbucket()`
- `remotes::install_cran()`
- `remotes::install_dev()`
- `remotes::install_github()`
- `remotes::install_gitlab()`
- `remotes::install_git()`
- `remotes::install_local()`
- `remotes::install_svn()`
- `remotes::install_url()`
- `renv::install()`

これらの関数は、このレッスンの範囲を超えており、特定の動作について充分な知識を持って注意して使用する必要があります。
一般的な推奨事項は、Bioconductor パッケージの正しいバージョン管理を確保するために、他のインストールメカニズムの上に `BiocManager::install()` を使用することです。

::::::::::::::::::::::::::::::::::::::::::::::::::

## Bioconductor のリリースと現在のバージョン

*[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)* パッケージがインストールされると、`BiocManager::version()` 関数は現在の R セッションでアクティブな Bioconductor プロジェクトのバージョン (すなわちリリース) を表示します。


``` r
BiocManager::version()
```

``` output
[1] '3.19'
```

R と Bioconductor パッケージの正しいバージョンを使用することは、再現性の重要な側面です。
*[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)* パッケージは、現在のセッションで実行されている R のバージョンを使用して、現在の R ライブラリにインストールできる Bioconductor パッケージのバージョンを決定します。

Bioconductor プロジェクトは、毎年 4 月と 10 月の 2 回のリリースを行います。
Bioconductor の 4 月のリリースは、R の年次リリースと一致します。
Bioconductor の 10 月のリリースは、次のリリース（4 月）まで、その年次サイクルのために同じバージョンの R を引き続き使用します。

![](fig/bioc-release-cycle.svg){alt='選択された Bioconductor と R のバージョンのリリース日程。'}

**選択された Bioconductor と R のバージョンのリリース日程。**
タイムラインの上部には R プロジェクトのバージョンとおおよそのリリース日が示されています。
タイムラインの下部には Bioconductor プロジェクトのバージョンとリリース日が示されています。
出典: [Bioconductor][bioc-release-dates]。

各 6 か月のパッケージ開発サイクル中に、Bioconductor は次のリリースサイクルで利用可能になる R のバージョンとの互換性をチェックします。
その後、新しい Bioconductor リリースが生成されるたびに、Bioconductor リポジトリ内のすべてのパッケージのバージョンがインクリメントされます。これには Bioconductor プロジェクトのバージョンを決定する *[BiocVersion](https://bioconductor.org/packages/3.19/BiocVersion)* パッケージも含まれます。


``` r
packageVersion("BiocVersion")
```

``` output
[1] '3.19.1'
```

これは、前のリリース以降まったく更新されていないパッケージにも当てはまります。
各パッケージの新しいバージョンは、対応する R のバージョン用に指定されています。
言い換えれば、そのパッケージのバージョンは、正しいバージョンの R を使用する R セッションでのみインストールおよびアクセスできます。
このバージョンのインクリメントは、Bioconductor パッケージの各バージョンを Bioconductor プロジェクトのユニークなリリースに関連付けるために不可欠です。

4 月のリリースに続いて、ユーザーは新しい Bioconductor パッケージのリリースを利用するために新しいバージョンの R をインストールする必要があります。

一方、10 月には、ユーザーは新しい Bioconductor パッケージのリリースを利用するために同じバージョンの R を使用し続けることができます。
ただし、4 月の Bioconductor リリースから 10 月の Bioconductor リリースに R ライブラリを更新するには、`BiocManager::install()` 関数を呼び出し、`version` オプションとして正しい Bioconductor のバージョンを指定する必要があります。たとえば:


``` r
BiocManager::install(version = "3.14")
```

これは 1 回だけ実行する必要があります。*[BiocVersion](https://bioconductor.org/packages/3.19/BiocVersion)* パッケージは対応するバージョンに更新され、現在の R ライブラリで使用している Bioconductor のバージョンを示します。

:::::::::::::::::::::::::::::::::::::::::  callout

### さらに進む

このレッスンの [Discussion][discuss-release-cycle] 記事には、Bioconductor プロジェクトのリリースサイクルに関するセクションが含まれています。

::::::::::::::::::::::::::::::::::::::::::::::::::

## 更新を確認する

`BiocManager::valid()` 関数は、ユーザーライブラリに現在インストールされているパッケージのバージョンを検査し、いずれかのパッケージの新しいバージョンが Bioconductor リポジトリで利用可能かどうかを確認します。

すべてが最新の場合、関数は単に `TRUE` を返します。


``` r
BiocManager::valid()
```

``` warning
Warning: 5 packages out-of-date; 0 packages too new
```

``` output

* sessionInfo()

R version 4.4.3 (2025-02-28)
Platform: x86_64-pc-linux-gnu
Running under: Ubuntu 22.04.5 LTS

Matrix products: default
BLAS:   /usr/lib/x86_64-linux-gnu/openblas-pthread/libblas.so.3 
LAPACK: /usr/lib/x86_64-linux-gnu/openblas-pthread/libopenblasp-r0.3.20.so;  LAPACK version 3.10.0

locale:
 [1] LC_CTYPE=C.UTF-8       LC_NUMERIC=C           LC_TIME=C.UTF-8       
 [4] LC_COLLATE=C.UTF-8     LC_MONETARY=C.UTF-8    LC_MESSAGES=C.UTF-8   
 [7] LC_PAPER=C.UTF-8       LC_NAME=C              LC_ADDRESS=C          
[10] LC_TELEPHONE=C         LC_MEASUREMENT=C.UTF-8 LC_IDENTIFICATION=C   

time zone: UTC
tzcode source: system (glibc)

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
[1] BiocStyle_2.32.1

loaded via a namespace (and not attached):
 [1] digest_0.6.37          assertthat_0.2.1       R6_2.6.1              
 [4] fastmap_1.2.0          xfun_0.52              magrittr_2.0.3        
 [7] glue_1.8.0             knitr_1.50             sandpaper_0.16.13.9000
[10] htmltools_0.5.8.1      rmarkdown_2.29         lifecycle_1.0.4       
[13] xml2_1.3.8             ps_1.9.1               cli_3.6.5             
[16] processx_3.8.6         callr_3.7.6            vctrs_0.6.5           
[19] renv_1.1.4             withr_3.0.2            compiler_4.4.3        
[22] purrr_1.1.0            tools_4.4.3            tinkr_0.3.0           
[25] evaluate_1.0.4         yaml_2.3.10            BiocManager_1.30.26   
[28] pegboard_0.7.9         rlang_1.1.6           

Bioconductor version '3.19'

  * 5 packages out-of-date
  * 0 packages too new

create a valid installation with

  BiocManager::install(c(
    "httr2", "pillar", "purrr", "Rcpp", "RSQLite"
  ), update = TRUE, ask = FALSE, force = TRUE)

more details: BiocManager::valid()$too_new, BiocManager::valid()$out_of_date
```

便利なことに、更新できるパッケージがある場合、関数はそれらのパッケージを更新するために必要なコマンドを生成して表示します。
ユーザーは単にそのコマンドをコピーして R コンソールで実行する必要があります。

:::::::::::::::::::::::::::::::::::::::::  callout

### 古くなったパッケージライブラリの例

以下の例では、`BiocManager::valid()` 関数は `TRUE` を返しませんでした。
その代わりに、アクティブなユーザー セッションに関する情報が含まれ、ユーザーが古いライブラリ内のすべての古いパッケージを最新バージョンに置き換えるために実行すべき `BiocManager::install()` への正確な呼び出しが表示されます。

```
> BiocManager::valid()

* sessionInfo()

R バージョン 4.1.0 (2021-05-18)
プラットフォーム: x86_64-apple-darwin17.0 (64-bit)
実行環境: macOS Big Sur 11.6

行列積: デフォルト
LAPACK: /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRlapack.dylib

ロケール:
[1] en_GB.UTF-8/en_GB.UTF-8/en_GB.UTF-8/C/en_GB.UTF-8/en_GB.UTF-8

添付された基本パッケージ:
[1] stats     graphics  grDevices datasets  utils     methods   base     

名前空間を介して読み込まれたパッケージ (添付されていない):
[1] BiocManager_1.30.16 compiler_4.1.0      tools_4.1.0         renv_0.14.0        

Bioconductor バージョン '3.13'

  * 18 パッケージが古くなっています
  * 0 パッケージが新しすぎます

以下のコマンドを実行して、有効なインストールを作成します。

  BiocManager::install(c(
    "cpp11", "data.table", "digest", "hms", "knitr", "lifecycle", "matrixStats", "mime", "pillar", "RCurl",
    "readr", "remotes", "S4Vectors", "shiny", "shinyWidgets", "tidyr", "tinytex", "XML"
  ), update = TRUE, ask = FALSE)

詳細: BiocManager::valid()$too_new, BiocManager::valid()$out_of_date

警告メッセージ:
18 パッケージが古くなっています; 0 パッケージが新しすぎます 
```

具体的には、この例では、メッセージがユーザーに次のコマンドを実行してインストールを最新の状態にするように指示しています。

```
  BiocManager::install(c(
    "cpp11", "data.table", "digest", "hms", "knitr", "lifecycle", "matrixStats", "mime", "pillar", "RCurl",
    "readr", "remotes", "S4Vectors", "shiny", "shinyWidgets", "tidyr", "tinytex", "XML"
  ), update = TRUE, ask = FALSE)
```

::::::::::::::::::::::::::::::::::::::::::::::::::

## パッケージ リポジトリを探る

Bioconductor [biocViews][glossary-biocviews] は、以前のエピソード [Introduction to Bioconductor][crossref-intro-biocviews] で示されたように、テーマに沿って Bioconductor パッケージの階層的分類をブラウズすることで新しいパッケージを発見する優れた方法です。

さらに、`BiocManager::available()` 関数は、Bioconductor および CRAN リポジトリからインストールできるパッケージ名の完全なリストを返します。
たとえば、*[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)* を使用してインストールできるパッケージの総数


``` r
length(BiocManager::available())
```

``` output
[1] 26080
```

具体的には、現在の Bioconductor リポジトリと検索パス上の他のリポジトリの和集合を次のように表示できます。


``` r
BiocManager::repositories()
```

``` output
                                                BioCsoft 
           "https://bioconductor.org/packages/3.19/bioc" 
                                                 BioCann 
"https://bioconductor.org/packages/3.19/data/annotation" 
                                                 BioCexp 
"https://bioconductor.org/packages/3.19/data/experiment" 
                                           BioCworkflows 
      "https://bioconductor.org/packages/3.19/workflows" 
                                               BioCbooks 
          "https://bioconductor.org/packages/3.19/books" 
                                             carpentries 
                    "https://carpentries.r-universe.dev" 
                                     carpentries_archive 
                    "https://carpentries.github.io/drat" 
                                                    CRAN 
                           "https://cloud.r-project.org" 
```

各リポジトリの URL は Web ブラウザでアクセスでき、そのリポジトリから利用可能なパッケージの完全なリストが表示されます。
たとえば、[https://bioconductor.org/packages/3.14/bioc](https://bioconductor.org/packages/3.14/bioc) に移動します。

:::::::::::::::::::::::::::::::::::::::::  callout

### さらに進む

`BiocManager::repositories()` 関数は、基本関数 `available.packages()` と組み合わせて、特定のパッケージリポジトリから利用できるパッケージを照会できます。たとえば、Bioconductor [ソフトウェア パッケージ][glossary-software-package] リポジトリです。

```
> db = available.packages(repos = BiocManager::repositories()["BioCsoft"])
> dim(db)
[1] 1948   17
> head(rownames(db))
[1] "a4"          "a4Base"      "a4Classif"   "a4Core"      "a4Preproc"
[6] "a4Reporting"
```

::::::::::::::::::::::::::::::::::::::::::::::::::

便利なことに、`BiocManager::available()` には `pattern=` 引数が含まれ、特に注釈リソースをナビゲートするために便利です (元のユースケースがその理由です)。
たとえば、マウスモデル生物に対して利用可能なさまざまな [注釈データパッケージ][glossary-annotation-package] を次のようにリストできます。


``` r
BiocManager::available(pattern = "*Mmusculus")
```

``` output
 [1] "BSgenome.Mmusculus.UCSC.mm10"        "BSgenome.Mmusculus.UCSC.mm10.masked"
 [3] "BSgenome.Mmusculus.UCSC.mm39"        "BSgenome.Mmusculus.UCSC.mm8"        
 [5] "BSgenome.Mmusculus.UCSC.mm8.masked"  "BSgenome.Mmusculus.UCSC.mm9"        
 [7] "BSgenome.Mmusculus.UCSC.mm9.masked"  "EnsDb.Mmusculus.v75"                
 [9] "EnsDb.Mmusculus.v79"                 "PWMEnrich.Mmusculus.background"     
[11] "TxDb.Mmusculus.UCSC.mm10.ensGene"    "TxDb.Mmusculus.UCSC.mm10.knownGene" 
[13] "TxDb.Mmusculus.UCSC.mm39.knownGene"  "TxDb.Mmusculus.UCSC.mm39.refGene"   
[15] "TxDb.Mmusculus.UCSC.mm9.knownGene"  
```

## パッケージのインストール

`BiocManager::install()` 関数を使用して、パッケージをインストールまたは更新します。

この関数は、パッケージ名の文字列ベクトルを受け取り、Bioconductor リポジトリからこれらをインストールしようとします。


``` r
BiocManager::install(c("S4Vectors", "BiocGenerics"))
```

ただし、Bioconductor リポジトリでパッケージが見つからない場合、関数はグローバルオプション `repos` にリストされているリポジトリでこれらのパッケージも検索します。

:::::::::::::::::::::::::::::::::::::::::  callout

### 貢献する！

BiocManager を使用してインストールできる非 Bioconductor パッケージの例を追加します。
できれば、このレッスンで後で使用されるパッケージです。

::::::::::::::::::::::::::::::::::::::::::::::::::

## パッケージのアンインストール

Bioconductor パッケージは、基本 R 関数 `remove.packages()` を使用して、他の R パッケージと同様に R ライブラリから削除できます。
本質的に、この関数は単にインストールされたパッケージを削除し、必要に応じてインデックス情報を更新します。
その結果、そのパッケージをセッションに添付したり、そのパッケージのドキュメントを参照したりすることはできなくなります。


``` r
remove.packages("S4Vectors")
```

[bioc-release-dates]: https://bioconductor.org/about/release-announcements/
[discuss-release-cycle]: discuss.html#the-bioconductor-release-cycle
[glossary-biocviews]: reference.html#biocviews
[crossref-intro-biocviews]: https://carpentries-incubator.github.io/bioc-project/02-introduction-to-bioconductor/index.html#package-classification-using-biocviews
[glossary-software-package]: reference.html#software-package
[glossary-annotation-package]: reference.html#annotationdata-package

:::::::::::::::::::::::::::::::::::::::: keypoints

- BiocManager パッケージは CRAN リポジトリから利用可能です。
- `BiocManager::install()` は、Bioconductor パッケージ (CRAN および GitHub からも) のインストールと更新に使用されます。
- `BiocManager::valid()` は、利用可能なパッケージの更新をチェックするために使用されます。
- `BiocManager::version()` は、現在インストールされている Bioconductor のバージョンを報告します。
- `BiocManager::install()` は、特定の Bioconductor のバージョンに対して R ライブラリ全体を更新するためにも使用できます。

::::::::::::::::::::::::::::::::::::::::::::::::::


