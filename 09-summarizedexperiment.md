---
source: Rmd
title: SummarizedExperimentクラス
teaching: XX
exercises: XX
---

---



::::::::::::::::::::::::::::::::::::::: objectives

- 実験データとメタデータがどのように1つのオブジェクトに保存されるかを説明してください。
- データとメタデータを分析の過程全体で同期させることが重要な理由を説明してください。

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- SummarizedExperimentオブジェクトの情報はどのように整理されていますか？
- その情報はどのように追加、編集、アクセスできますか？

::::::::::::::::::::::::::::::::::::::::::::::::::





## パッケージのインストール

次のセクションに進む前に、必要なBioconductorのパッケージをいくつかインストールします。
最初に、*[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)*パッケージがインストールされているか確認します。それがない場合はインストールします。
その後、`BiocManager::install()`関数を使用して必要なパッケージをインストールします。


``` r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install(c("SummarizedExperiment"))
```

## モチベーション

実験は多面的なデータセットで、通常、分析に必要な少なくとも2つの重要な情報を含んでいます：

- アッセイデータは、通常、サンプルのセット内の特徴のセットの測定を表す行列です
  (例：RNAシーケンシング)。
- サンプルメタデータは、通常、サンプルに関する情報を表す`data.frame`です
  (例：治療群)。

これらすべての情報は、下流の分析で情報を正確に処理し、信頼できる結果を生成するために、
同じサンプルで同じ順序に保たれる必要があります。

分析ワークフローで、サンプルのサブセットを分析したり、より正確な下流の分析を可能にするために削除する必要のある外れ値を特定したりすることは非常に一般的です。
そのような場合、実験のすべての側面は同じサンプルのセットにサブセットされる必要があります
\-- 同じ順序で -- データセットの一貫性と正確な結果を保つために。

`SummarizedExperiment` -- *[SummarizedExperiment](https://bioconductor.org/packages/3.19/SummarizedExperiment)*パッケージで実装されている
\-- は、個々の実験の重要な側面を単一のオブジェクトに統合するコンテナを提供し、サブセットや再配置の操作中にデータとメタデータを調整します。
多くの生物学的データ型と包括的な機能セットを受け入れる柔軟性があり、
Bioconductor全体で再利用される人気のあるデータ構造です。
例えば、`SummarizedExperiment`に慣れていることは、
*[DESeq2](https://bioconductor.org/packages/3.19/DESeq2)*パッケージや、シングルセル解析のための*[SingleCellExperiment](https://bioconductor.org/packages/3.19/SingleCellExperiment)*拡張クラスで作業するために必要です。

## クラス構造

`SummarizedExperiment`は、行が対象の特徴（例：遺伝子、転写、エクソンなど）を表す行列のようなコンテナです。 列はサンプルを表します。

オブジェクトは1つ以上のアッセイを含むことができ、各アッセイは同じ次元の行列のようなオブジェクトで表されます。

特徴に関する情報は、`SummarizedExperiment`オブジェクト内にネストされた`DataFrame`オブジェクトに保存され、
関数`rowData()`を使用してアクセスできます。
`DataFrame`の各行は、SummarizedExperimentオブジェクトの対応する行の特徴に関する情報を提供します。
その情報には、実験とは無関係な注釈（例：遺伝子識別子）や、ワークフロー中にアッセイデータから計算された品質管理メトリックが含まれる場合があります。

同様に、サンプルに関する情報は、別の`DataFrame`オブジェクトに保存され、これも`SummarizedExperiment`オブジェクトにネストされ、
関数`colData()`を使用してアクセスできます。

次のグラフィックは、クラスのジオメトリを表示し、縦（列）および横（行）関係を強調します。
これは、*[SummarizedExperiment](https://bioconductor.org/packages/3.19/SummarizedExperiment)*パッケージのビネットから得られました。

![](fig/summarizedexperiment.svg){alt='SummarizedExperimentクラスの模式図です。'}

## SummarizedExperimentオブジェクトの作成

はじめにパッケージをロードしましょう：


``` r
library(SummarizedExperiment)
```

それでは、レッスンセッティング中にダウンロードしたファイルからアッセイデータをインポートします。

そのファイルは単純なテキストファイルで、
最初の列には作られた特徴識別子が含まれ、
他のすべての列には作られたサンプルのシミュレーションデータが含まれています。
そのため、基本R関数`read.csv`を使用してファイルを解析し、
`data.frame`オブジェクトに変換できます。

以下の例では、行名は最初の列にあることを示しており、
関数は出力オブジェクト内ですぐに行名を設定します。
指定しなければ、関数は通常の列として解析し、デフォルトの整数インデックスで行名をそのままに 設定します。


``` r
count_data <- read.csv("data/counts.csv", row.names = 1)
count_data
```

``` output
        sample_1 sample_2 sample_3 sample_4
gene_1       109       84       91      105
gene_2       111       97       98      108
gene_3        89      121      105       99
gene_4       105      109      122      101
gene_5        82       97      112       83
gene_6        89       96       90      116
gene_7       121       95       88      106
gene_8       101      101       86      103
gene_9        91      119       89       87
gene_10       81      111       81      118
gene_11       93      118       93       99
gene_12      103      111      116      103
gene_13       89      126      103      100
gene_14      101      107      111       79
gene_15       96       91      103      108
gene_16      110      102      128      103
gene_17       95      106      118      100
gene_18       99      115      114      102
gene_19      114      105       94      118
gene_20      110       88       99      102
gene_21      116       95       94      105
gene_22      114       96      107       91
gene_23       97      120       93       90
gene_24       91       84      118       97
gene_25       99      106       97      110
```

1つのアッセイデータ行列があれば、`SummarizedExperiment`オブジェクトを作成できますが、
サンプルメタデータがなければ、サンプルに関する情報を必要としない無監督分析のみが可能です。

以下の例では、カウントデータの行列を'counts'という名前で保存する`SummarizedExperiment`オブジェクトを作成します。
引数'Без'（複数形）は1つ以上のアッセイを受け入れることができ、
\-- 上記で説明されているように --、そのため、唯一のアッセイ行列を名前付きの`list`にカプセル化します。
アッセイに名前を付けることは、複数のアッセイを含むワークフローで、各アッセイを明確に識別し、取得するために重要です。


``` r
se <- SummarizedExperiment(
  assays = list(counts = count_data)
)
se
```

``` output
class: SummarizedExperiment 
dim: 25 4 
metadata(0):
assays(1): counts
rownames(25): gene_1 gene_2 ... gene_24 gene_25
rowData names(0):
colnames(4): sample_1 sample_2 sample_3 sample_4
colData names(0):
```

上の出力では、オブジェクトの要約ビューがアッセイを確認でき、
そのため、全体的な`SummarizedExperiment`オブジェクトが4つのサンプル内の25の特徴に関する情報を含んでいることを思い出させます。
オブジェクトには、行メタデータや列メタデータは含まれていません。

より包括的な`SummarizedExperiment`オブジェクトを作成するために、他の2つのファイルから遺伝子メタデータとサンプルメタデータをインポートします。

ファイルはカウントデータと同様にフォーマットされているため、
再び基本R関数`read.csv()`を使用して、`data.frame`オブジェクトに解析します。


``` r
sample_metadata <- read.csv("data/sample_metadata.csv", row.names = 1)
sample_metadata
```

``` output
         condition batch
sample_1         A     1
sample_2         A     2
sample_3         B     1
sample_4         B     2
```


``` r
gene_metadata <- read.csv("data/gene_metadata.csv", row.names = 1)
gene_metadata
```

``` output
        chromosome
gene_1           4
gene_2           4
gene_3           5
gene_4           4
gene_5           5
gene_6           1
gene_7           2
gene_8           1
gene_9           3
gene_10          1
gene_11          1
gene_12          5
gene_13          5
gene_14          1
gene_15          3
gene_16          4
gene_17          2
gene_18          5
gene_19          1
gene_20          3
gene_21          5
gene_22          5
gene_23          1
gene_24          4
gene_25          5
```

これで、遺伝子とサンプルメタデータを含めた`SummarizedExperiment`オブジェクトを再作成できます：


``` r
se <- SummarizedExperiment(
  assays = list(counts = count_data),
  colData = sample_metadata,
  rowData = gene_metadata
)
se
```

``` output
class: SummarizedExperiment 
dim: 25 4 
metadata(0):
assays(1): counts
rownames(25): gene_1 gene_2 ... gene_24 gene_25
rowData names(1): chromosome
colnames(4): sample_1 sample_2 sample_3 sample_4
colData names(2): condition batch
```

この出力を前の'アッセイのみ'バージョンと比較すると、`rowData`と`colData`
のコンポーネントがそれぞれ1つと4つのメタデータを含むことがわかります。

## 情報にアクセスする

いくつかの関数が`SummarizedExperiment`オブジェクトのさまざまなコンポーネントにアクセスできます。

`assays()`関数は、オブジェクトに保存されたアッセイのリストを返します。
出力は常に`List`であり、オブジェクトに単一のアッセイが含まれている場合でもです。


``` r
assays(se)
```

``` output
List of length 1
names(1): counts
```

`assayNames()`関数は、アッセイ名の文字ベクトルを返します。
これは、オブジェクトに大量のアッセイが含まれる場合に最も便利です。
`assays()`関数（上記参照）は、すべてを表示しないかもしれません。
さまざまなアッセイの名前を知ることは、個々のアッセイにアクセスするための鍵です。


``` r
assayNames(se)
```

``` output
[1] "counts"
```

`assay()`関数を使用して、オブジェクトから単一のアッセイを取得できます。
これに対して、関数には目的のアッセイの名前または整数位置を指定する必要があります。
未指定の場合、関数は自動的にオブジェクト内の最初のアッセイを返します。


``` r
head(assay(se, "counts"))
```

``` output
       sample_1 sample_2 sample_3 sample_4
gene_1      109       84       91      105
gene_2      111       97       98      108
gene_3       89      121      105       99
gene_4      105      109      122      101
gene_5       82       97      112       83
gene_6       89       96       90      116
```

`colData()`および`rowData()`関数を使用して、サンプルメタデータおよび行メタデータを取得できます。


``` r
colData(se)
```

``` output
DataFrame with 4 rows and 2 columns
           condition     batch
         <character> <integer>
sample_1           A         1
sample_2           A         2
sample_3           B         1
sample_4           B         2
```


``` r
rowData(se)
```

``` output
DataFrame with 25 rows and 1 column
        chromosome
         <integer>
gene_1           4
gene_2           4
gene_3           5
gene_4           4
gene_5           5
...            ...
gene_21          5
gene_22          5
gene_23          1
gene_24          4
gene_25          5
```

また、 `$`演算子を使用して、サンプルメタデータの単一列にアクセスできます。
この演算子の便利な機能は、RStudioで自動的にトリガーされるオートコンプリートと、ターミナルアプリケーションでのタブキーを使用して自動的にトリガーされることです。


``` r
se$batch
```

``` output
[1] 1 2 1 2
```

特に、特徴メタデータの単一列にアクセスするための演算子はありません。
そのため、ユーザーはまず`rowData()`によって返される完全な`DataFrame`にアクセスし、
その後、標準の$または[[演算子を使用して列にアクセスする必要があります。


``` r
rowData(se)[["chromosome"]]
```

``` output
 [1] 4 4 5 4 5 1 2 1 3 1 1 5 5 1 3 4 2 5 1 3 5 5 1 4 5
```

### 情報の追加と編集

情報は、`SummarizedExperiment`作成後に追加することができます。
実際、これは、正規化されたアッセイ値を計算し、
それをアッセイのリストに追加し、
特徴またはサンプルに対する品質管理メトリックを計算し、
それを`rowData`と`colData`のコンポーネントに適切に追加し、
全体のオブジェクト内に保存された情報の量を徐々に増やすプロセスの基礎となります。

情報にアクセスするための多くの関数が、上記のセクションで説明されていますが、
新しい値を追加したり、既存の値を編集したりするための対応関数があります。
編集は、既に使用されている名前の下に値を追加するだけの結果であり、
既存の値を置き換える効果があります。

以下の例では、'counts'アッセイに対して1の擬似カウントを追加した後に、
その結果としてアッセイ名'logcounts'を追加します。


``` r
assay(se, "logcounts") <- log1p(assay(se, "counts"))
se
```

``` output
class: SummarizedExperiment 
dim: 25 4 
metadata(0):
assays(2): counts logcounts
rownames(25): gene_1 gene_2 ... gene_24 gene_25
rowData names(1): chromosome
colnames(4): sample_1 sample_2 sample_3 sample_4
colData names(2): condition batch
```

上の出力では、オブジェクトが現在2つのアッセイを持っていることがわかります：
最初に作成されたオブジェクトに含まれている'counts'アッセイと、
今追加した'logcounts'アッセイです。

同様に、`colData()`および`rowData()`関数、
ならびに`$`演算子を使用して、対応するコンポーネントの値を追加および編集することができます。

以下の例では、各サンプルのカウントの合計を計算し、
結果を新しい名前'sum_counts'のサンプルメタデータテーブルに保存します。


``` r
colData(se)[["sum_counts"]] <- colSums(assay(se, "counts"))
colData(se)
```

``` output
DataFrame with 4 rows and 3 columns
           condition     batch sum_counts
         <character> <integer>  <numeric>
sample_1           A         1       2506
sample_2           A         2       2600
sample_3           B         1       2550
sample_4           B         2       2533
```

次の例では、各特徴の平均カウントを計算し、
新しい名前'mean_counts'の特徴メタデータテーブルに保存します。


``` r
rowData(se)[["mean_counts"]] <- rowSums(assay(se, "counts"))
rowData(se)
```

``` output
DataFrame with 25 rows and 2 columns
        chromosome mean_counts
         <integer>   <numeric>
gene_1           4         389
gene_2           4         414
gene_3           5         414
gene_4           4         437
gene_5           5         374
...            ...         ...
gene_21          5         410
gene_22          5         408
gene_23          1         400
gene_24          4         390
gene_25          5         412
```

:::::::::::::::::::::::::::::::::::::::: keypoints

- `SummarizedExperiment`クラスは、アッセイデータとメタデータの両方を保存するための単一のコンテナを提供します。
- アッセイデータとメタデータは、サブセット化と再配置の操作を通じて同期が保たれます。
- `SummarizedExperiment`オブジェクトのさまざまなコンポーネントに保存された情報にアクセスしたり、追加したり、編集したりするための包括的な関数セットが利用可能です。

::::::::::::::::::::::::::::::::::::::::::::::::::
[biocviews]: https://www.bioconductor.org/packages/release/BiocViews.html
[biomart-ensembl]: https://www.ensembl.org/biomart/martview
