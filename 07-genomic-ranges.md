---
source: Rmd
title: Genomics ranges を扱う
teaching: XX
exercises: XX
---



::::::::::::::::::::::::::::::::::::::: objectives

- ゲノム座標と区間がBioconductorプロジェクトでどのように表現されるかを説明します。
- ゲノム座標の範囲を処理するために利用可能なBioconductorパッケージおよびメソッドを特定します。

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- Bioconductorでゲノムスケールの座標を表現する推奨方法は何ですか？
- ゲノムの範囲を効率的に処理するためのメソッドを提供するBioconductorパッケージはどれですか？
- さまざまなゲノムファイル形式から/へのゲノム座標のセットをインポート/エクスポートする方法は？

::::::::::::::::::::::::::::::::::::::::::::::::::





## パッケージのインストール

次のセクションに進む前に、必要となるいくつかのBioconductorパッケージをインストールします。
まず、*[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)*パッケージがインストールされているか確認し、ない場合はインストールします。
次に、`BiocManager::install()`関数を使用して必要なパッケージをインストールします。


``` r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("GenomicRanges")
```

## GenomicRangesパッケージとクラス

### ゲノム範囲のためにクラスが必要な理由は？

ゲノミクスの時代に、多くの観察は遺伝子の座標の範囲、すなわちゲノムスケールの区間として報告されます。
アッセイの性質に応じて、これらのゲノム範囲は遺伝子、トランスクリプト、エクソン、一塩基多型（SNP）、転写因子結合部位、またはChIP-seqやATAC-seqなどの次世代シーケンシングアッセイからのピークを表す場合があります。

ゲノム範囲は、アッセイされた値（例：遺伝子発現）の観察をゲノムまたは生物内の物理的位置に結びつけます。
例えば、これらのゲノム範囲は、アッセイされた特徴と既知の調節領域のデータベースとの物理的近接性や重複を照会するために使用することができます。

最終的に報告される測定値に使用されるゲノム範囲は、既知のゲノム特徴のデータベース内のゲノム範囲のセットに対する組み合わせや操作の結果であることが多いです。
例えば、RNAシーケンシングにおいて、次世代シーケンシングリードはしばしば個々のエクソン内でカウントされ、それらのカウントは後に各遺伝子のすべてのエクソンで集約されます。
また、プロモーターは、既知の転写開始サイト（TSS）の上流および/または下流の任意の幅の領域として定義されることが多いです。

重要なことに、ゲノム範囲は必ずしも複数の座標を跨ぐ必要はありません。
範囲の概念は数学的な方法で意味されており、単一塩基のゲノム範囲（例：SNP）は、同じ座標で開閉するか（または右開区間の場合は次の座標で開閉する）と説明できます。

多くの生物において、遺伝物質は複数の別々の核酸分子（例：染色体、プラスミド）に分割されています。
そのため、ゲノム範囲は配列の名前とその配列上の座標の数値区間によって記述されます。

![](fig/bioc-genomicranges.svg){alt='GenomicRanges代数の使用例。'}

**GenomicRanges代数の使用例。**
Huber, Carey, Gentleman, Anders, Carlson, Carvalho, Bravo, Davis, Gatto, Girke, Gottardo, Hahne, Hansen, Irizarry, Lawrence, Love, MacDonald, Obenchain, Oles, Pages, Reyes, Shannon, Smyth, Tenenbaum, Waldron, and Morgan (2015)から適応。
この図は、2つのトランスクリプトからなる遺伝子モデルの例と、その遺伝子モデルに対するさまざまなゲノム範囲の定義を示しています。
例えば、特定の図示において、スプライスされていないトランスクリプトは、最初のエクソンの開始から最後のエクソンの終了までのすべての座標の範囲を要約します。一方、遺伝子領域は、少なくとも1つのトランスクリプトの1エクソンに含まれる座標のセットとして定義されます。

### 区間の簡単な紹介

区間は、連続した座標の軸上の開始位置と終了位置を使用して数学的に説明されます。
区間は、その二つの座標の間のすべての実数を含み、各区間の幅は開始位置と終了位置の座標の差から計算できます。

一般的に、開始と終了の位置は、任意の有理数、包括浮動小数点数になり得ます。
ただし、ゲノム学では、整数座標は通常、ポリマー（例：核酸、タンパク質）内のモノマー（例：ヌクレオチド、アミノ酸）の位置を表すために使用されます。

さまざまなルールを使用して区間を定義するパッケージ、データベース、プログラミング言語に出会うかもしれません。
Rでは、インデックスは1ベース（すなわちシーケンスの最初の位置は1）ですが、Pythonは0ベース（すなわちシーケンスの最初の位置のインデックスは0）です。
同様に、UCSCゲノムブラウザの参照ファイルは0ベースであり、Ensemblゲノムブラウザのものは1ベースです。

共有座標系における区間の定義は、二つの区間間の距離を計算したり、重複する区間を特定したりするなどの計算を可能にします。

![](fig/intervals.svg){alt='区間の例。'}

**区間の例。**
A、B、およびCという名前の3つの区間が表されています。
区間Aは位置5から始まり、位置9で終わり、幅は4ユニットです。
区間Bは位置1から始まり、位置3で終わり、幅は2ユニットです。
区間Cは位置3から始まり、位置6で終わり、幅は3ユニットです。
区間AとCは、座標5から6の間で重複し、一方、区間BとCは座標3で会い、厳密には互いに重複していません。

### ゲノム範囲の簡単な紹介

ゲノム範囲は、本質的に生物学的シーケンス（例：染色体）上の数学的な区間の概念を拡張します。
つまり、ゲノム範囲は、それが存在する生物学的配列の名前と、その配列内でゲノム範囲が跨ぐ整数座標の範囲を結合します。
これは、異なる生物学的配列上の重複する範囲のゲノム特徴を区別するための重要な要素です。

さらに、DNA配列の二重鎖特性も、ゲノム範囲に対してストランド性の概念を加えます。
もし知られている場合、ゲノム特徴のストランド情報は重要な情報であり、追跡されるべきで、後続の解析に使用されるかもしれません。
たとえば、同じDNA配列の反対のストランドで共通の座標範囲を跨ぐゲノム範囲は、重なっていると見なされない場合があります（例：ストランド特異的な次世代シーケンシングアッセイの目的のため）。

ゲノム範囲は_閉じた_区間です - 開始位置と終了位置は区間に含まれます。核酸の例では、開始位置は区間内の最初のヌクレオチドを示し、終了位置は区間内の最後のヌクレオチドを示します。

![](fig/genomic-intervals.svg){alt='ゲノム区間の例。'}

**ゲノム範囲の例。**
ゲノム範囲は、存在する生物学的配列の名前（ここでは「chr1」）と、その配列内での開始と終了の位置で定義されます。
ここで、数値位置は明示的に示されていませんが、核酸のシーケンスと座標が左から右に増加していることを示す矢印によって暗示されています。
この例では、ゲノム範囲を使用して個々のエクソンを記述でき、メタデータはそれらのエクソンをトランスクリプトと遺伝子にグループ化します。
さらに、エクソン、トランスクリプト、遺伝子のストランド性は、各ゲノム範囲の正確な位置を記述するための重要な情報です。

## GenomicRangesパッケージ

### 概要

*[GenomicRanges](https://bioconductor.org/packages/3.19/GenomicRanges)*パッケージは、ゲノム範囲をS4オブジェクトとして表現するための[S4クラス][glossary-s4-class]を実装しています。

具体的には、`GRanges`クラスは、機能が存在するシーケンスの名前と、そのシーケンス内で機能が跨ぐ整数座標の範囲を含む区間のセットを保存するために設計されています。

より一般的には、`IRanges`クラスは、シーケンス名の概念なしで、整数座標の範囲にわたる区間のセットを保存するために設計されています。
このように、`GRanges`オブジェクトは、`IRanges`オブジェクトとシーケンス名のベクターを組み合わせたものであるに過ぎません。

これらのS4クラスは、自動的な有効性チェック機能を提供し、整数区間およびゲノム範囲上の一般的な操作を実装するさまざまなメソッドを提供します。
区間間の距離の計算から重複するゲノム範囲の特定まで。

*[GenomicRanges](https://bioconductor.org/packages/3.19/GenomicRanges)*パッケージで定義されている基本クラスの簡単な説明は、パッケージのヴィネットの1つに利用可能で、`vignette("GenomicRangesIntroduction")`としてアクセス可能。本より詳細な情報は、他のパッケージのヴィネットに提供され、`browseVignettes("GenomicRanges")`としてアクセス可能です。

### 最初のステップ

始めるには、パッケージをロードします。


``` r
library(GenomicRanges)
```

### IRangesクラス

多くの生物のゲノム空間は複数の配列（例：染色体）に分割されていますが、ゲノム範囲上での多くの操作は、整数位置のみが重要な個々の配列内で行われます。
`IRanges`クラスは、次の三つの情報のうち二つに基づいて定義される「単純な」範囲のためのコンテナを提供します：

- 範囲の開始位置
- 範囲の幅
- 範囲の終了位置

`IRanges()`コンストラクタ関数は、`start=`, `width=`, `end=`の引数でこれら三つの情報を受け入れます。
例えば、開始位置と幅から二つの整数範囲を作成します：

- ある範囲は位置10から始まり、幅は10です。
- 別の範囲は位置15から始まり、幅は5です。


``` r
demo_iranges <- IRanges(start = c(10, 15), width = c(10, 5))
demo_iranges
```

``` output
IRanges object with 2 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]        10        19        10
  [2]        15        19         5
```

オブジェクトは、要求した_開始_と_幅_の情報だけでなく、他の二つの情報から自然に算出される_終了_位置も表示されることに注意します。

:::::::::::::::::::::::::::::::::::::::  challenge

### チャレンジ

`IRanges()`コンストラクタ関数の`start=`および`end=`の引数を使用して、上記と同じ二つの範囲を作成します。

:::::::::::::::  solution

### ソリューション


``` r
IRanges(start = c(10, 15), end = c(19, 19))
```

``` output
IRanges object with 2 ranges and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]        10        19        10
  [2]        15        19         5
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

すべての区間の開始および終了位置、ならびに各区間の幅は、`start()`, `end()`および`width()`関数を使用して数値ベクターとして抽出できます。


``` r
start(demo_iranges)
```

``` output
[1] 10 15
```

``` r
end(demo_iranges)
```

``` output
[1] 19 19
```

``` r
width(demo_iranges)
```

``` output
[1] 10  5
```

`IRanges`ファミリーのオブジェクトは`Vector`クラスを拡張し、インデックスの観点から一次元ベクターとして扱われます。
そのため、個々の範囲は通常のベクターのように整数インデックスで抽出できます。


``` r
demo_iranges[1]
```

``` output
IRanges object with 1 range and 0 metadata columns:
          start       end     width
      <integer> <integer> <integer>
  [1]        10        19        10
```

### IRangesのメタデータ

`IRanges`クラスは、各範囲のメタデータ情報を収容でき、名前は`names=`引数に渡され、さまざまなメタデータは名前付きベクターとして渡されます。

例えば、"A"と"B"という名前の二つの範囲を作成します。
さらに、例として文字値と数値を保存するメタデータフィールドを定義します。
この例では、メタデータフィールドの名前と値は完全に任意です。


``` r
demo_with_metadata <- IRanges(
  start = c(10,  15),
  end   = c(19,  19),
  names = c("A", "B"),
  character_metadata = c("control", "target"),
  numeric_metadata = c(100, 200)
)
demo_with_metadata
```

``` output
IRanges object with 2 ranges and 2 metadata columns:
        start       end     width | character_metadata numeric_metadata
    <integer> <integer> <integer> |        <character>        <numeric>
  A        10        19        10 |            control              100
  B        15        19         5 |             target              200
```

メタデータ列は、`mcols()`関数を使用して`DataFrame`として抽出できます（"メタデータ列"の略）。


``` r
mcols(demo_with_metadata)
```

``` output
DataFrame with 2 rows and 2 columns
  character_metadata numeric_metadata
         <character>        <numeric>
A            control              100
B             target              200
```

名前の文字ベクターは、`names()`関数を使用して抽出できます。


``` r
names(demo_with_metadata)
```

``` output
[1] "A" "B"
```

基本データ型の名前付きベクターに類似して、個々の範囲は名前で抽出できます。


``` r
demo_with_metadata["A"]
```

``` output
IRanges object with 1 range and 2 metadata columns:
        start       end     width | character_metadata numeric_metadata
    <integer> <integer> <integer> |        <character>        <numeric>
  A        10        19        10 |            control              100
```

### IRangesでの操作

`IRanges`は、数値座標の範囲に対するほとんどの操作の基礎を提供します。

例えば、クエリセットとサブジェクトセットの二つの範囲が与えられた場合、`findOVerlaps()`関数を使用して、二つのセットの範囲のどのペアが互いに重なっているかを確認できます。


``` r
query_iranges <- IRanges(
  start = c(8, 16),
  end   = c(14, 18)
)
overlaps_iranges <- findOverlaps(query = query_iranges, subject = demo_iranges)
overlaps_iranges
```

``` output
Hits object with 3 hits and 0 metadata columns:
      queryHits subjectHits
      <integer>   <integer>
  [1]         1           1
  [2]         2           1
  [3]         2           2
  -------
  queryLength: 2 / subjectLength: 2
```

結果は、まだ紹介していない`Hits`オブジェクトの形式で返されます。
`Hits`オブジェクトは、`queryHits`および`subjectHits`という二つの整数列からなるテーブルとして視覚化されます。
テーブルの各行は、クエリセット内の1つの範囲とサブジェクトセット内の1つの範囲の重複を報告し、各列の整数値は重複に関与する各セット内の範囲のインデックスを示します。

この例では、クエリセットの最初の範囲がサブジェクトセットの最初の範囲と重なっていることを確認し、クエリセットの二番目の範囲がサブジェクトセット内の両範囲と重なっています。

:::::::::::::::::::::::::::::::::::::::::  callout

### さらに深く

後処理用に、`Hits`オブジェクトから二つのコンポーネントを、それぞれの名前を使用して抽出できます：


``` r
queryHits(overlaps_iranges)
```

``` output
[1] 1 2 2
```

``` r
subjectHits(overlaps_iranges)
```

``` output
[1] 1 1 2
```

テーブルとして表示される間、`Hits`オブジェクトは実際にはベクターのように処理されます。
クエリ範囲とサブジェクト範囲の間の個々のヒットは、インデックスで抽出できます：


``` r
overlaps_iranges[1]
```

``` output
Hits object with 1 hit and 0 metadata columns:
      queryHits subjectHits
      <integer>   <integer>
  [1]         1           1
  -------
  queryLength: 2 / subjectLength: 2
```

::::::::::::::::::::::::::::::::::::::::::::::::::

### GRangesクラス

整数範囲を定義した後、ゲノム範囲を定義するために必要な追加情報は、各範囲が位置するゲノム配列の名前です。

例えば、次のように二つのゲノム範囲を定義します。

- 1番染色体（略称"chr1"）の位置10から25の範囲
- 2番染色体（略称"chr2"）の位置20から35の範囲

それを行うために、`GRanges()`コンストラクタ関数を使用します。
シーケンス名を`seqnames=`引数に、開始位置と終了位置の両方を`ranges=`引数に、`IRanges`オブジェクトとして提供します。


``` r
demo_granges <- GRanges(
  seqnames = c("chr1", "chr2"),
  ranges = IRanges(
    start = c(10, 20),
    end   = c(25, 35))
)
demo_granges
```

``` output
GRanges object with 2 ranges and 0 metadata columns:
      seqnames    ranges strand
         <Rle> <IRanges>  <Rle>
  [1]     chr1     10-25      *
  [2]     chr2     20-35      *
  -------
  seqinfo: 2 sequences from an unspecified genome; no seqlengths
```

コンソールで、オブジェクトは`seqnames`コンポーネントにシーケンス名を表示し、`ranges`コンポーネントに`start-end`形式で範囲を表示します。
さらに、上記の例は、`GRanges`オブジェクトが`strand`というコンポーネントを持ち、シンボル`*`がストランドのないゲノム範囲を示すことも示しています。これは、その情報が提供されていないためです。

ストランド情報は、`strand=`引数に提供できます。例えば：


``` r
demo_granges2 <- GRanges(
  seqnames = c("chr1", "chr2"),
  ranges = IRanges(
    start = c(10, 20),
    end   = c(25, 35)),
  strand  = c("+", "-")
)
demo_granges2
```

``` output
GRanges object with 2 ranges and 0 metadata columns:
      seqnames    ranges strand
         <Rle> <IRanges>  <Rle>
  [1]     chr1     10-25      +
  [2]     chr2     20-35      -
  -------
  seqinfo: 2 sequences from an unspecified genome; no seqlengths
```

最後に、上記の例は、`GRanges`オブジェクトに`seqinfo`というコンポーネントが含まれていることを示しています。これは、`seqnames`コンポーネントに含まれる各シーケンスに関する情報を保存するために使用されます。
前の例では、シーケンスについての情報を提供していません。
そのため、`seqinfo`コンポーネントはオブジェクトを作成するために使用したシーケンスの名前で自動的に埋められ、残りの情報は指定されていないままとなっています。


``` r
seqinfo(demo_granges2)
```

``` output
Seqinfo object with 2 sequences from an unspecified genome; no seqlengths:
  seqnames seqlengths isCircular genome
  chr1             NA         NA   <NA>
  chr2             NA         NA   <NA>
```

上記の例は、シーケンスに関する情報に、それぞれの名前と長さだけでなく、それらが環状ポリマー（例：プラスミド）を表すかどうか、またそれらが属するゲノムの名前が含まれていることも示しています。

この情報は、オブジェクトの作成時にコンストラクタに直接提供したり、既存のオブジェクトに対して`seqinfo()`アクセサと`Seqinfo()`コンストラクタを使用して編集することができます。


``` r
seqinfo(demo_granges2) <-  Seqinfo(
    seqnames = c("chr1", "chr2"),
    seqlengths = c(1234, 5678),
    isCircular = c(FALSE, TRUE),
    genome = c("homo_sapiens", "homo_sapiens")
)
demo_granges2
```

``` output
GRanges object with 2 ranges and 0 metadata columns:
      seqnames    ranges strand
         <Rle> <IRanges>  <Rle>
  [1]     chr1     10-25      +
  [2]     chr2     20-35      -
  -------
  seqinfo: 2 sequences (1 circular) from homo_sapiens genome
```

各範囲の個々の開始および終了位置、ならびにすべての区間の幅は、`start()`, `end()`および`width()`関数を使用して数値ベクターとして抽出できます。


``` r
start(demo_granges2)
```

``` output
[1] 10 20
```

``` r
end(demo_granges2)
```

``` output
[1] 25 35
```

``` r
width(demo_granges2)
```

``` output
[1] 16 16
```

シーケンス名とストランド情報は、それぞれ`seqnames()`および`strand()`の関数を使用して抽出できます。


``` r
seqnames(demo_granges2)
```

``` output
factor-Rle of length 2 with 2 runs
  Lengths:    1    1
  Values : chr1 chr2
Levels(2): chr1 chr2
```

``` r
strand(demo_granges2)
```

``` output
factor-Rle of length 2 with 2 runs
  Lengths: 1 1
  Values : + -
Levels(3): + - *
```

### GRangesのメタデータ

`IRanges`と同様に、メタデータは直接`GRanges`コンストラクタ関数に渡すことができます。
例えば：


``` r
demo_granges3 <- GRanges(
  seqnames = c("chr1", "chr2"),
  ranges = IRanges(
    start = c(10, 20),
    end = c(25, 35)),
  metadata1 = c("control", "target"),
  metadata2 = c(1, 2)
)
demo_granges3
```

``` output
GRanges object with 2 ranges and 2 metadata columns:
      seqnames    ranges strand |   metadata1 metadata2
         <Rle> <IRanges>  <Rle> | <character> <numeric>
  [1]     chr1     10-25      * |     control         1
  [2]     chr2     20-35      * |      target         2
  -------
  seqinfo: 2 sequences from an unspecified genome; no seqlengths
```

### ファイルからのゲノム範囲のインポート

頻繁に、大規模なゲノム範囲のコレクションは、手動で記述されたコードではなく、ファイルからインポートされます。
特に、既知の遺伝子機能のゲノム全体の注釈は、[Ensembl FTP][ensembl-ftp] や [UCSC Genome Data][ucsc-genome-data] のようなウェブサイトにファイルとして配布されています。

さまざまなファイル形式が、バイオインフォマティクスのワークフローでゲノム範囲を格納するために一般的に使用されています。
例えば、BED (Browser Extensible Data) 形式は、クロマチン免疫沈降シーケンシング (ChIP-Seq) で一般的に見られますが、GTF (Gene Transfer Format, GTF2.2) は、エクソン、転写物、遺伝子などのゲノム機能を記述するための_デファクト_標準ファイル形式です。

次の例では、私たちは小さなGTFファイルからアクチンベータ (ACTB) の遺伝子モデルを一連のゲノム範囲としてインポートします。
例示のファイルは、[Ensembl FTP][ensembl-ftp] サイトからダウンロードされた_Homo sapiens_種のGTFファイルのサブセットを表しています。
元のファイルは300万行以上と22のメタデータフィールドを持っており、その中からサブセットが抽出されてこのレッスン用の小さなファイルになりました。

特に、私たちは `import()` ジェネリックを使用します。
*[BiocIO](https://bioconductor.org/packages/3.19/BiocIO)* パッケージで定義され、*[rtracklayer](https://bioconductor.org/packages/3.19/rtracklayer)* パッケージで実装されたメソッドを用いて、一般的なファイル拡張子を認識し、それぞれの特定のファイル形式を解析するための適切なメソッドに関連付けることができる多目的関数です。


``` r
library(rtracklayer)
```

``` warning
Warning: replacing previous import 'S4Arrays::makeNindexFromArrayViewport' by
'DelayedArray::makeNindexFromArrayViewport' when loading 'SummarizedExperiment'
```

``` r
actb_gtf_data <- rtracklayer::import("data/actb.gtf")
actb_gtf_data
```

``` output
GRanges object with 267 ranges and 7 metadata columns:
        seqnames          ranges strand |      source           type     score
           <Rle>       <IRanges>  <Rle> |    <factor>       <factor> <numeric>
    [1]        7 5526409-5563902      - | rtracklayer     gene              NA
    [2]        7 5526409-5530601      - | rtracklayer     transcript        NA
    [3]        7 5530542-5530601      - | rtracklayer     exon              NA
    [4]        7 5529535-5529684      - | rtracklayer     exon              NA
    [5]        7 5529535-5529657      - | rtracklayer     CDS               NA
    ...      ...             ...    ... .         ...            ...       ...
  [263]        7 5540676-5540771      - | rtracklayer five_prime_utr        NA
  [264]        7 5529658-5529663      - | rtracklayer five_prime_utr        NA
  [265]        7 5561852-5562716      - | rtracklayer transcript            NA
  [266]        7 5562390-5562716      - | rtracklayer exon                  NA
  [267]        7 5561852-5561949      - | rtracklayer exon                  NA
            phase         gene_id   gene_name   transcript_id
        <integer>     <character> <character>     <character>
    [1]      <NA> ENSG00000075624        ACTB            <NA>
    [2]      <NA> ENSG00000075624        ACTB ENST00000674681
    [3]      <NA> ENSG00000075624        ACTB ENST00000674681
    [4]      <NA> ENSG00000075624        ACTB ENST00000674681
    [5]      <NA> ENSG00000075624        ACTB ENST00000674681
    ...       ...             ...         ...             ...
  [263]      <NA> ENSG00000075624        ACTB ENST00000414620
  [264]      <NA> ENSG00000075624        ACTB ENST00000414620
  [265]      <NA> ENSG00000075624        ACTB ENST00000646584
  [266]      <NA> ENSG00000075624        ACTB ENST00000646584
  [267]      <NA> ENSG00000075624        ACTB ENST00000646584
  -------
  seqinfo: 1 sequence from an unspecified genome; no seqlengths
```

:::::::::::::::::::::::::::::::::::::::::  callout

### さらに進む

特定のファイル形式を解析するための個別のメソッドを直接呼び出すことができます。
例えば、この場合、GTFファイル形式はGFFバージョン2ファイル形式と同一であるため、私たちは `rtracklayer::import.gff2()` 関数を直接呼び出すことができます。同じ効果を得ることができます。

すべてのメソッドのフルリストは、*[rtracklayer](https://bioconductor.org/packages/3.19/rtracklayer)* パッケージのドキュメントを参照してください。

::::::::::::::::::::::::::::::::::::::::::::::::::

上記の例では、GTFファイルの内容が`GRanges`オブジェクトにインポートされました。 ファイルの各エントリについて、配列名、開始位置と終了位置、ストランド情報がオブジェクトの専用コンポーネントをポピュレートするために使用され、他のすべての情報はメタデータの別のカラムとして保存されます。

ここから、この`GRanges`オブジェクトは、私たちがこのエピソードの初めに作成した他の`GRanges`オブジェクトのように操作できます。

### GRangesおよびGRangesListクラスの操作

これまでに示したように、`GRanges`オブジェクトは手動で定義またはファイルからインポートできます。
それらは、興味のあるゲノム領域を表し、既知のゲノム機能のデータベースをそれぞれ表します。
いずれにせよ、生物情報学のワークフロー全体で`GRanges`オブジェクトに一般的に適用される多数の操作があります。

#### サブセット

例えば、`subset()`メソッドは、配列名、開始位置と終了位置、ストランド、または任意のメタデータフィールドの条件に一致するゲノム範囲のセットを抽出するのに非常に便利です。
以下の例では、位置`5527147`で開始する`transcript`型のすべてのレコードを抽出します。


``` r
subset(actb_gtf_data, type == "transcript" & start == 5527147)
```

``` output
GRanges object with 5 ranges and 7 metadata columns:
      seqnames          ranges strand |      source       type     score
         <Rle>       <IRanges>  <Rle> |    <factor>   <factor> <numeric>
  [1]        7 5527147-5529949      - | rtracklayer transcript        NA
  [2]        7 5527147-5530581      - | rtracklayer transcript        NA
  [3]        7 5527147-5530604      - | rtracklayer transcript        NA
  [4]        7 5527147-5530604      - | rtracklayer transcript        NA
  [5]        7 5527147-5530604      - | rtracklayer transcript        NA
          phase         gene_id   gene_name   transcript_id
      <integer>     <character> <character>     <character>
  [1]      <NA> ENSG00000075624        ACTB ENST00000642480
  [2]      <NA> ENSG00000075624        ACTB ENST00000676397
  [3]      <NA> ENSG00000075624        ACTB ENST00000676319
  [4]      <NA> ENSG00000075624        ACTB ENST00000676189
  [5]      <NA> ENSG00000075624        ACTB ENST00000473257
  -------
  seqinfo: 1 sequence from an unspecified genome; no seqlengths
```

#### 分割

別々に、`split()`メソッドは、最初に単一の`GRanges`オブジェクトに格納されていたゲノム範囲のセットを、名前付きの`GRanges`オブジェクトのリストに分割するのに役立ちます。
便利なことに、`GRangesList`クラスは、`GRanges`オブジェクトのリストを効率的に表示および処理するためのコンテナを提供します。

以下の例では、最初にエクソンを表すエントリのサブセットを抽出し、次にそれらのエクソンを転写物識別子で分割し、結果を`GRangesList`オブジェクトとして得ます。


``` r
actb_exons <- subset(actb_gtf_data, type == "exon")
actb_exons_by_transcript <- split(actb_exons, actb_exons$transcript_id)
actb_exons_by_transcript
```

``` output
GRangesList object of length 23:
$ENST00000414620
GRanges object with 4 ranges and 7 metadata columns:
      seqnames          ranges strand |      source     type     score
         <Rle>       <IRanges>  <Rle> |    <factor> <factor> <numeric>
  [1]        7 5562574-5562790      - | rtracklayer     exon        NA
  [2]        7 5540676-5540771      - | rtracklayer     exon        NA
  [3]        7 5529535-5529663      - | rtracklayer     exon        NA
  [4]        7 5529282-5529400      - | rtracklayer     exon        NA
          phase         gene_id   gene_name   transcript_id
      <integer>     <character> <character>     <character>
  [1]      <NA> ENSG00000075624        ACTB ENST00000414620
  [2]      <NA> ENSG00000075624        ACTB ENST00000414620
  [3]      <NA> ENSG00000075624        ACTB ENST00000414620
  [4]      <NA> ENSG00000075624        ACTB ENST00000414620
  -------
  seqinfo: 1 sequence from an unspecified genome; no seqlengths

$ENST00000417101
GRanges object with 3 ranges and 7 metadata columns:
      seqnames          ranges strand |      source     type     score
         <Rle>       <IRanges>  <Rle> |    <factor> <factor> <numeric>
  [1]        7 5529806-5529982      - | rtracklayer     exon        NA
  [2]        7 5529535-5529663      - | rtracklayer     exon        NA
  [3]        7 5529235-5529400      - | rtracklayer     exon        NA
          phase         gene_id   gene_name   transcript_id
      <integer>     <character> <character>     <character>
  [1]      <NA> ENSG00000075624        ACTB ENST00000417101
  [2]      <NA> ENSG00000075624        ACTB ENST00000417101
  [3]      <NA> ENSG00000075624        ACTB ENST00000417101
  -------
  seqinfo: 1 sequence from an unspecified genome; no seqlengths

$ENST00000425660
GRanges object with 7 ranges and 7 metadata columns:
      seqnames          ranges strand |      source     type     score
         <Rle>       <IRanges>  <Rle> |    <factor> <factor> <numeric>
  [1]        7 5530524-5530601      - | rtracklayer     exon        NA
  [2]        7 5529535-5529663      - | rtracklayer     exon        NA
  [3]        7 5529161-5529400      - | rtracklayer     exon        NA
  [4]        7 5529019-5529059      - | rtracklayer     exon        NA
  [5]        7 5528281-5528719      - | rtracklayer     exon        NA
  [6]        7 5528004-5528185      - | rtracklayer     exon        NA
  [7]        7 5527156-5527891      - | rtracklayer     exon        NA
          phase         gene_id   gene_name   transcript_id
      <integer>     <character> <character>     <character>
  [1]      <NA> ENSG00000075624        ACTB ENST00000425660
  [2]      <NA> ENSG00000075624        ACTB ENST00000425660
  [3]      <NA> ENSG00000075624        ACTB ENST00000425660
  [4]      <NA> ENSG00000075624        ACTB ENST00000425660
  [5]      <NA> ENSG00000075624        ACTB ENST00000425660
  [6]      <NA> ENSG00000075624        ACTB ENST00000425660
  [7]      <NA> ENSG00000075624        ACTB ENST00000425660
  -------
  seqinfo: 1 sequence from an unspecified genome; no seqlengths

...
<20 more elements>
```

コンソールで上記のオブジェクトを印刷すると、最初の行はそのオブジェクトのクラスを`GRrangesList`として確認し、そのリスト内の各名前付き`GRanges`がドル記号とそのアイテムの名前によって紹介され、通常のRの名前付きリストのように表示されます。

#### 長さ

性質上、`list`オブジェクトに適用可能な多くのメソッドは、直接`GRangesList`オブジェクトに適用できます。
例えば、`lengths()`関数を`GRangesList`に使用して、リスト内の各`GRanges`オブジェクトの長さを整数ベクトルとして表示できます。

上記の最新の例では、各転写物内のエクソンの数を、`GRangesList`内の各`GRanges`オブジェクトの長さとして計算できます：


``` r
lengths(actb_exons_by_transcript)
```

``` output
ENST00000414620 ENST00000417101 ENST00000425660 ENST00000432588 ENST00000443528 
              4               3               7               5               3 
ENST00000462494 ENST00000464611 ENST00000473257 ENST00000477812 ENST00000480301 
              5               3               5               5               2 
ENST00000484841 ENST00000493945 ENST00000642480 ENST00000645025 ENST00000645576 
              5               6               5               4               5 
ENST00000646584 ENST00000646664 ENST00000647275 ENST00000674681 ENST00000675515 
              2               6               3               6               6 
ENST00000676189 ENST00000676319 ENST00000676397 
              6               3               6 
```

:::::::::::::::::::::::::::::::::::::::  challenge

### チャレンジ

重要なことに、上記の `lengths()` 関数 (最終的に `s` が付いています) は、`length()` 関数 ( `s` が付いていないもの) とは異なります。
前者はリストオブジェクトに対して使用されることを目的としており、リスト内の各要素の長さを返すベクトルを返します。後者はリスト自体の長さを表す単一の数値スカラーを返します (つまり、リスト内の要素数)。

`length(actb_exons_by_transcript)` は何を返し、この数は生物学的に何を表しますか？

:::::::::::::::  solution

### ソリューション


``` r
length(actb_exons_by_transcript)
```

``` output
[1] 23
```

このコードは、単一の整数値 `23` を返します。それは、`GRangesList`オブジェクト内の`GRanges`の数であり、遺伝子ACTBの転写物の数でもあります。

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

### オーバーラップによるサブセット

ゲノム範囲で作業するときの最も一般的な操作の一つは、特定のゲノム領域に位置するゲノム範囲の任意の大きなコレクションをサブセット化することです。たとえば、ゲノムブラウザに情報をトラックとして視覚化するときなどです。

示すために、私たちは興味のある領域を表す新しい`GRanges`を手動で定義し、その領域に重なる以前にインポートされたすべてのゲノム範囲をGTFファイルから抽出するために使用します。


``` r
region_of_interest <- GRanges(
    seqnames = "7",
    ranges = IRanges(start = 5525830, end = 5531239)
)
actb_in_region <- subsetByOverlaps(x = actb_gtf_data, ranges = region_of_interest)
actb_in_region
```

``` output
GRanges object with 256 ranges and 7 metadata columns:
        seqnames          ranges strand |      source           type     score
           <Rle>       <IRanges>  <Rle> |    <factor>       <factor> <numeric>
    [1]        7 5526409-5563902      - | rtracklayer     gene              NA
    [2]        7 5526409-5530601      - | rtracklayer     transcript        NA
    [3]        7 5530542-5530601      - | rtracklayer     exon              NA
    [4]        7 5529535-5529684      - | rtracklayer     exon              NA
    [5]        7 5529535-5529657      - | rtracklayer     CDS               NA
    ...      ...             ...    ... .         ...            ...       ...
  [252]        7 5529535-5529657      - | rtracklayer CDS                   NA
  [253]        7 5529655-5529657      - | rtracklayer start_codon           NA
  [254]        7 5529282-5529400      - | rtracklayer exon                  NA
  [255]        7 5529282-5529400      - | rtracklayer CDS                   NA
  [256]        7 5529658-5529663      - | rtracklayer five_prime_utr        NA
            phase         gene_id   gene_name   transcript_id
        <integer>     <character> <character>     <character>
    [1]      <NA> ENSG00000075624        ACTB            <NA>
    [2]      <NA> ENSG00000075624        ACTB ENST00000674681
    [3]      <NA> ENSG00000075624        ACTB ENST00000674681
    [4]      <NA> ENSG00000075624        ACTB ENST00000674681
    [5]      <NA> ENSG00000075624        ACTB ENST00000674681
    ...       ...             ...         ...             ...
  [252]      <NA> ENSG00000075624        ACTB ENST00000414620
  [253]      <NA> ENSG00000075624        ACTB ENST00000414620
  [254]      <NA> ENSG00000075624        ACTB ENST00000414620
  [255]      <NA> ENSG00000075624        ACTB ENST00000414620
  [256]      <NA> ENSG00000075624        ACTB ENST00000414620
  -------
  seqinfo: 1 sequence from an unspecified genome; no seqlengths
```

`subset()`メソッドと同様に、`subsetByOverlaps()`メソッドは新しい`GRanges`オブジェクトを返します。
私たちは、新しいサブセットオブジェクト内の情報（新しいサブセットされたオブジェクト内に256の範囲があり、元のオブジェクトには267の範囲がある）を視覚的に比較することができ、あるいは新しい`GRanges`オブジェクトが元の`GRanges`オブジェクトよりも小さいかどうかを確認するために、プログラム的に2つのオブジェクトの長さを比較することもできます。


``` r
length(actb_in_region) - length(actb_gtf_data)
```

``` output
[1] -11
```

上記の例では、新しい`GRanges`オブジェクトは、元の`GRanges`オブジェクトよりも11レコード少ないことがわかります。

:::::::::::::::::::::::::::::::::::::::::  callout

### さらに進む

`GRanges`および`GRangesList`オブジェクトで動作するための方法が他にもたくさんありますが、ここで示すことができるのはそれだけです。

`GenomicRanges`パッケージで定義された関数の完全なリストは、パッケージのドキュメントのインデックスページで見つけることができ、`help(package="GenomicRanges")`を使用してアクセスできます。
`GenomicRanges`パッケージのビネットには、より多くの例やユースケースも見つけることができ、`browseVignettes("GenomicRanges")`を使用してアクセスできます。

::::::::::::::::::::::::::::::::::::::::::::::::::

[glossary-s4-class]: reference.html#s4-class
[ensembl-ftp]: https://www.ensembl.org/info/data/ftp/
[ucsc-genome-data]: https://hgdownload.soe.ucsc.edu/downloads.html

:::::::::::::::::::::::::::::::::::::::: keypoints

- `GenomicRanges`パッケージは、ゲノムスケールでの座標の範囲を表すクラスを定義します。
- `GenomicRanges`パッケージは、ゲノム範囲を効率よく処理するためのメソッドも定義します。
- `rtracklayer`パッケージは、一般的なゲノムファイル形式からゲノム範囲をインポートおよびエクスポートするための関数を提供します。

::::::::::::::::::::::::::::::::::::::::::::::::::


