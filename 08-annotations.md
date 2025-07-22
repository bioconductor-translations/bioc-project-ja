---
source: Rmd
title: アノテーションの取り扱い
teaching: XX
exercises: XX
---

---



::::::::::::::::::::::::::::::::::::::: objectives

- バイオコンダクタープロジェクトでの遺伝子アノテーションの管理方法を説明してください。
- 遺伝子アノテーションを取得および使用するためのバイオコンダクターパッケージとメソッドを特定します。

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- どのバイオコンダクターパッケージが、遺伝子アノテーションを効率的に取得および使用するためのメソッドを提供していますか？
- 異なる遺伝子識別子間を変換するために、遺伝子アノテーションパッケージをどのように使用できますか？

::::::::::::::::::::::::::::::::::::::::::::::::::





## パッケージのインストール

次のセクションに進む前に、必要なバイオコンダクターパッケージをインストールします。
まず、*[BiocManager](https://bioconductor.org/packages/3.19/BiocManager)*パッケージがインストールされていることを確認します。そうでない場合は、それをインストールします。
次に、`BiocManager::install()`関数を使用して必要なパッケージをインストールします。


``` r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install(c("biomaRt", "org.Hs.eg.db"))
```

## 概要

遺伝子アノテーションをクエリするために特化したパッケージは、それぞれの特性に基づいてバイオコンダクターの「ソフトウェア」と「アノテーション」カテゴリに存在します。

「ソフトウェア」セクションでは、実際に遺伝子アノテーションを含まないパッケージを見つけますが、それらはオンラインリソースから動的に取得します（例えば、[Ensembl BioMart][biomart-ensembl]）。
そのようなバイオコンダクターパッケージの一つが*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*です。

その代わりに、「アノテーション」セクションでは、アノテーションを含むパッケージを見つけます。
例として、*[org.Hs.eg.db](https://bioconductor.org/packages/3.19/org.Hs.eg.db)*、*[EnsDb.Hsapiens.v86](https://bioconductor.org/packages/3.19/EnsDb.Hsapiens.v86)*、および*[TxDb.Hsapiens.UCSC.hg38.knownGene](https://bioconductor.org/packages/3.19/TxDb.Hsapiens.UCSC.hg38.knownGene)*があります。

このエピソードでは、2つのアプローチを示します：

- *[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*パッケージを使用してEnsembl BioMart APIからアノテーションをクエリします。
- *[org.Hs.eg.db](https://bioconductor.org/packages/3.19/org.Hs.eg.db)*アノテーションパッケージからアノテーションをクエリします。

## オンラインリソースまたはバイオコンダクタアノテーションパッケージ？

### 最新情報へのアクセス

バイオコンダクターの6ヶ月のリリースサイクルは、最新の安定版リリースブランチから提供されるパッケージが6ヶ月間更新されないことを示唆しています（バグ修正のみが許可され、機能の更新はありません）。
その結果、アノテーションパッケージは最大で6ヶ月間の古い情報を含む可能性があります。

その代わりに、独立したオンラインリソースは、更新情報のリリースを促進する異なるポリシーを持ちます。
一部のデータベースは頻繁に更新されますが、他のデータベースは数年にわたって更新されていない場合があります。

最新の情報へのアクセスは、再現性とのバランスも考慮しなければなりません。
最新の情報を1回ダウンロードしただけでは、いつその情報がダウンロードされたかを記録していなければ意味がありません。

### ストレージ要件

本質的に、バイオコンダクターアノテーションパッケージはソフトウェアパッケージよりも大きいです。
他のRパッケージと同様に、アノテーションパッケージは使用する前にユーザーのコンピューターにインストールする必要があります。
これにより、無視できない量のディスクスペースを使用する可能性があります。

逆に、オンラインリソースは一般にプログラムでアクセスされ、通常、ユーザーが再現可能な分析を行うためにコードを記録するだけで済みます。

### インターネット接続

オンラインリソースを使用しているときは、オンラインリソースからダウンロードしたアノテーションをローカルファイルに書き込み、分析中にそのローカルファイルを参照するのが良いアイデアです。

オンラインリソースが何らかの理由で使用できなくなる（例えば、ダウンタイムやインターネット接続の喪失）と、ローカルファイルを使用する分析は続行できますが、オンラインリソースに依存する分析はできません。

対照的に、バイオコンダクターアノテーションパッケージは、インストール時にのみインターネット接続が必要です。
一度インストールされると、ローカルに保存された情報に依存するため、インターネット接続は必要ありません。

### 再現性

バイオコンダクターアノテーションパッケージは自然にバージョン管理されているため、ユーザーは分析で使用したパッケージのバージョンを自信を持って報告できます。
ソフトウェアパッケージと同様に、ユーザーはアノテーションパッケージをいつ、どのように更新するかを制御します。

オンラインリソースは再現可能な分析を促進するための異なるポリシーを持っています。
オンラインリソースの一部はアノテーションの過去のバージョンを保持しており、ユーザーは時間の経過とともに同じ情報に一貫してアクセスできます。
これがない場合、一つの時点でアノテーションのコピーをダウンロードし、そのコピーをプロジェクトの存続期間中保持する必要があるかもしれません。一貫したアノテーションセットを使用することを保証するために。

### 一貫性

今後のこのエピソードの実用例で見るように、バイオコンダクターアノテーションパッケージは一般的に一貫したデータ構造のセットを再利用します。
これにより、1つのアノテーションパッケージに精通しているユーザーは、他のパッケージを迅速に開始できます。

独立したオンラインリソースは、データをさまざまな方法で整理していることが多く、ユーザーはそれぞれのデータにアクセス、取得、処理するためのカスタムコードを書く必要があります。

## Ensembl BioMartからアノテーションをクエリする

### Ensembl BioMart

[Ensembl BioMart][biomart-ensembl]は、生物学的データの広範な配列へのアクセスを容易にするために設計された堅牢なデータマイニングツールです。

[BioMart web interface][biomart-ensembl]は、研究者が複数の種にわたって遺伝子、タンパク質、およびその他のゲノムの特徴に関するデータを効率的にクエリし、取得できるようにします。
ユーザーに対して遺伝子ID、染色体位置、機能的アノテーションなどのさまざまな属性に基づいてデータをフィルタリング、並べ替え、エクスポートすることを許可します。

### バイオコンダクター`biomaRt`パッケージ

*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*は、Rセッションから直接Ensembl BioMartテーブルから大量のデータを取得できるバイオコンダクターパッケージです。

まずパッケージを読み込みます：


``` r
library(biomaRt)
```

### 利用可能なマートのリスト

Ensembl BioMartは、それぞれ「マート」または「バイオマート」とも呼ばれる4つのデータベースに多様な生物学的信息を整理します。
各マートは異なるタイプのデータに焦点を合わせています。

ユーザーは、関心のあるデータタイプに対応するマートを選択する必要があります。その後、情報をクエリできます。

関数`listMarts()`は、これらのマートの名前を表示するために使用できます。
これによりユーザーはマートの名前を記憶する必要がなくなり、また、関数はマートが名前変更、追加、または削除される場合にも更新された名前リストを返します。


``` r
listMarts()
```

``` output
               biomart                version
1 ENSEMBL_MART_ENSEMBL      Ensembl Genes 114
2   ENSEMBL_MART_MOUSE      Mouse strains 114
3     ENSEMBL_MART_SNP  Ensembl Variation 114
4 ENSEMBL_MART_FUNCGEN Ensembl Regulation 114
```

このデモでは、Ensembl遺伝子セットを含む`ENSEMBL_MART_ENSEMBL`というバイオマートを使用します。

特に、`version`カラムはバイオマートのバージョンも示しています。
Ensembl BioMartは定期的に更新されます（年に複数回）。
デフォルトでは、*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*関数は各バイオマートの最新バージョンにアクセスします。
これは再現性には理想的ではありません。

幸いなことに、Ensembl BioMartは、過去のバージョンのアーカイブを行い、プログラムからでも、ウェブサイトからでもアクセスできます。

関数`listEnsemblArchives()`は、アクセス可能なすべてのEnsembl BioMartのバージョンを表示するために使用できます。


``` r
listEnsemblArchives()
```

``` output
             name     date                                 url version
1  Ensembl GRCh37 Feb 2014          https://grch37.ensembl.org  GRCh37
2     Ensembl 114 May 2025 https://may2025.archive.ensembl.org     114
3     Ensembl 113 Oct 2024 https://oct2024.archive.ensembl.org     113
4     Ensembl 112 May 2024 https://may2024.archive.ensembl.org     112
5     Ensembl 111 Jan 2024 https://jan2024.archive.ensembl.org     111
6     Ensembl 110 Jul 2023 https://jul2023.archive.ensembl.org     110
7     Ensembl 109 Feb 2023 https://feb2023.archive.ensembl.org     109
8     Ensembl 108 Oct 2022 https://oct2022.archive.ensembl.org     108
9     Ensembl 107 Jul 2022 https://jul2022.archive.ensembl.org     107
10    Ensembl 106 Apr 2022 https://apr2022.archive.ensembl.org     106
11    Ensembl 105 Dec 2021 https://dec2021.archive.ensembl.org     105
12    Ensembl 104 May 2021 https://may2021.archive.ensembl.org     104
13    Ensembl 103 Feb 2021 https://feb2021.archive.ensembl.org     103
14    Ensembl 102 Nov 2020 https://nov2020.archive.ensembl.org     102
15    Ensembl 101 Aug 2020 https://aug2020.archive.ensembl.org     101
16    Ensembl 100 Apr 2020 https://apr2020.archive.ensembl.org     100
17     Ensembl 99 Jan 2020 https://jan2020.archive.ensembl.org      99
18     Ensembl 80 May 2015 https://may2015.archive.ensembl.org      80
19     Ensembl 77 Oct 2014 https://oct2014.archive.ensembl.org      77
20     Ensembl 75 Feb 2014 https://feb2014.archive.ensembl.org      75
21     Ensembl 54 May 2009 https://may2009.archive.ensembl.org      54
   current_release
1                 
2                *
3                 
4                 
5                 
6                 
7                 
8                 
9                 
10                
11                
12                
13                
14                
15                
16                
17                
18                
19                
20                
21                
```

上記の出力での重要な情報は、`url`カラムです。これは、*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*関数が対応するEnsembl BioMartのスナップショットからデータにアクセスするために必要なURLを提供します。

執筆時点では、現在のリリースはEnsembl 112であるため、対応するurl`https://may2024.archive.ensembl.org`を使用して、いつこのレッスンが配信されても再現可能な結果を確保します。

### バイオマートへの接続

上記で収集した2つの情報、すなわちバイオマートの名前とスナップショットのURLが、バイオマートデータベースに再現可能に接続するために必要なすべてです。

関数`useMart()`を使用して接続を作成します。
接続は伝統的に`mart`というオブジェクトに保存され、次のステップでオンラインマートから情報をクエリするために再利用されます。


``` r
mart <- useMart(biomart = "ENSEMBL_MART_ENSEMBL", host = "https://may2024.archive.ensembl.org")
```

### 利用可能なデータセットのリスト

各バイオマートには複数のデータセットが含まれています。

関数`listDatasets()`は、これらのデータセットの情報を表示するために使用されます。
これにより、ユーザーはデータセットの名前を記憶する必要がなくなり、関数が返す情報には各データセットの簡単な説明とそのバージョンが含まれます。

以下の例では、出力テーブルを最初の数行に制限します。完全なテーブルは214行です。


``` r
head(listDatasets(mart))
```

``` output
                       dataset                           description
1 abrachyrhynchus_gene_ensembl Pink-footed goose genes (ASM259213v1)
2     acalliptera_gene_ensembl      Eastern happy genes (fAstCal1.3)
3   acarolinensis_gene_ensembl       Green anole genes (AnoCar2.0v2)
4    acchrysaetos_gene_ensembl       Golden eagle genes (bAquChr1.2)
5    acitrinellus_gene_ensembl        Midas cichlid genes (Midas_v5)
6    amelanoleuca_gene_ensembl       Giant panda genes (ASM200744v2)
      version
1 ASM259213v1
2  fAstCal1.3
3 AnoCar2.0v2
4  bAquChr1.2
5    Midas_v5
6 ASM200744v2
```

上記の出力での重要な情報は、`dataset`カラムです。これは、*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*関数が対応するバイオマートテーブルからデータにアクセスするために必要な識別子を提供します。

このデモでは、出力に表示されていないヒトのEnsembl遺伝子セットを使用します。

利用可能なデータセットの数を考えると、手動でテーブルを検索するのではなく、パターンマッチングを使用して情報のテーブルをプログラムでフィルタリングしてみましょう：


``` r
subset(listDatasets(mart), grepl("sapiens", dataset))
```

``` output
                 dataset              description    version
80 hsapiens_gene_ensembl Human genes (GRCh38.p14) GRCh38.p14
```

上記の出力から、欲しいデータセットの識別子が`hsapiens_gene_ensembl`であることが分かります。

### データセットへの接続

使用したいデータセットを選択したら、再度関数`useMart()`を呼び出し、選択したデータセットを指定する必要があります。

一般的には、以前の`useMart()`の呼び出しをコピーして必要に応じて編集することが多いです。
また、通常、`mart`オブジェクトを新しい接続で置き換えることが一般的な慣行です。


``` r
mart <- useMart(
  biomart = "ENSEMBL_MART_ENSEMBL",
  dataset = "hsapiens_gene_ensembl",
  host = "https://may2024.archive.ensembl.org")
```

### データセット内の情報をリスト表示する

BioMartテーブルには、属性としても知られる多くの情報が含まれています。
実際には、それらは「ページ」として知られるカテゴリにグループ化されています。

関数`listAttributes()`は、それらの属性に関する情報を表示するために使用できます。
これにより、ユーザーはパッケージ内の利用可能な属性の名前を記憶する必要がなくなり、関数が返す情報には各属性の簡単な説明とそのページ分類が含まれます。

以下の例では、出力テーブルを最初の数行に制限します。完全なテーブルは3157行です。


``` r
head(listAttributes(mart))
```

``` output
                           name                  description         page
1               ensembl_gene_id               Gene stable ID feature_page
2       ensembl_gene_id_version       Gene stable ID version feature_page
3         ensembl_transcript_id         Transcript stable ID feature_page
4 ensembl_transcript_id_version Transcript stable ID version feature_page
5            ensembl_peptide_id            Protein stable ID feature_page
6    ensembl_peptide_id_version    Protein stable ID version feature_page
```

上記の出力での重要な情報は、`name`カラムです。これは、*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*関数が対応するバイオマートデータセットからその情報をクエリするために必要な識別子を提供します。

クエリしたい属性の選択は、達成したいことによって決まります。

例えば、遺伝子識別子のセットがあり、それに対してクエリしたいことを想像しましょう：

- 遺伝子符号
- 遺伝子が位置する染色体の名前
- その染色体上の遺伝子の開始位置と終了位置
- 遺伝子がコーディングされているストランド

ユーザーは通常、含めたい属性を特定するために、属性テーブルを手動で調べます。
また、経験と直感に基づいて属性テーブルをプログラム的にフィルタリングして、検索を絞り込むことも可能です：


``` r
subset(listAttributes(mart), grepl("position", name) & grepl("feature", page))
```

``` output
             name     description         page
10 start_position Gene start (bp) feature_page
11   end_position   Gene end (bp) feature_page
```

### BioMartテーブルから情報をクエリする

これで、実際のクエリを実行するために必要な情報がすべて揃いました：

- BioMartデータセットへの接続
- そのデータセット内の利用可能な属性のリスト

関数`getBM()`は、主な*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)*クエリ関数です。
フィルターとそれに対応する値のセットを指定すると、接続されているBioMartデータセットからユーザーが要求した属性を取得します。

以下の例では、クエリ用の任意の遺伝子識別子のベクトルを手動で作成します。
実際には、クエリは先行する分析から始まることがよくあります（例えば、差次的な遺伝子発現）。

以下の例では、まだ紹介していない属性もクエリされます。
前のセクションで、`listAttributes()`から返される属性テーブルを検索して、クエリに含める属性を特定する方法を説明しました。


``` r
query_gene_ids <- c(
  "ENSG00000133101",
  "ENSG00000145386",
  "ENSG00000134057",
  "ENSG00000157456",
  "ENSG00000147082"
)
getBM(
  attributes = c(
    "ensembl_gene_id",
    "hgnc_symbol",
    "chromosome_name",
    "start_position",
    "end_position",
    "strand"
  ),
  filters = "ensembl_gene_id",
  values = query_gene_ids,
  mart = mart
)
```

``` error
Error in .processResults(postRes, mart = mart, hostURLsep = sep, fullXmlQuery = fullXmlQuery, : Query ERROR: caught BioMart::Exception::Database: Error during query execution: Table 'ensembl_mart_112.hsapiens_gene_ensembl__ox_hgnc__dm' doesn't exist
```

フィルタリング属性`ensembl_gene_id`をデータセットから取得した属性に含めたことに注意してください。
これは、新しく取得された属性をクエリで使用される属性と確実に一致させるための鍵です。

## アノテーションパッケージからアノテーションをクエリする

### アノテーションパッケージのファミリー

包括的な情報を必要としつつ、適切なパッケージサイズを維持するために、バイオコンダクターアノテーションパッケージはリリース、データタイプ、種によって編成されています。

バイオコンダクターアノテーションパッケージの主要なファミリーは以下の通りです：

- `OrgDb`パッケージは、さまざまなタイプの遺伝子識別子と経路情報の間のマッピングを提供します。
- `EnsDb`パッケージは、Ensemblアノテーションの個々のリリースを提供します。
- `TxDb`パッケージは、UCSCアノテーションの個々のリリースを提供します。

これらのアノテーションファミリーは、*[AnnotationDbi](https://bioconductor.org/packages/3.19/AnnotationDbi)*パッケージで定義された`AnnotationDb`基底クラスから派生しています。
その結果、これらのアノテーションパッケージは、以下のセクションで示される同じ一連のR関数を使用してアクセスします。

### OrgDbパッケージの使用

この例では、*[org.Hs.eg.db](https://bioconductor.org/packages/3.19/org.Hs.eg.db)*パッケージを使用して、ヒト種の遺伝子アノテーションの使用方法を示します。

最初にパッケージを読み込みましょう：


``` r
library(org.Hs.eg.db)
```

各`OrgDb`パッケージには、パッケージ自体と同名のオブジェクトが含まれています。
そのオブジェクトには、パッケージが提供することを目的とするアノテーションが含まれています。

情報をクエリすることに加えて、オブジェクト全体を呼び出すことができ、含まれるアノテーションのスナップショットが作成された日付などの情報を印刷します。


``` r
org.Hs.eg.db
```

``` output
OrgDb object:
| DBSCHEMAVERSION: 2.1
| Db type: OrgDb
| Supporting package: AnnotationDbi
| DBSCHEMA: HUMAN_DB
| ORGANISM: Homo sapiens
| SPECIES: Human
| EGSOURCEDATE: 2024-Mar12
| EGSOURCENAME: Entrez Gene
| EGSOURCEURL: ftp://ftp.ncbi.nlm.nih.gov/gene/DATA
| CENTRALID: EG
| TAXID: 9606
| GOSOURCENAME: Gene Ontology
| GOSOURCEURL: http://current.geneontology.org/ontology/go-basic.obo
| GOSOURCEDATE: 2024-01-17
| GOEGSOURCEDATE: 2024-Mar12
| GOEGSOURCENAME: Entrez Gene
| GOEGSOURCEURL: ftp://ftp.ncbi.nlm.nih.gov/gene/DATA
| KEGGSOURCENAME: KEGG GENOME
| KEGGSOURCEURL: ftp://ftp.genome.jp/pub/kegg/genomes
| KEGGSOURCEDATE: 2011-Mar15
| GPSOURCENAME: UCSC Genome Bioinformatics (Homo sapiens)
| GPSOURCEURL: 
| GPSOURCEDATE: 2024-Feb29
| ENSOURCEDATE: 2023-Nov22
| ENSOURCENAME: Ensembl
| ENSOURCEURL: ftp://ftp.ensembl.org/pub/current_fasta
| UPSOURCENAME: Uniprot
| UPSOURCEURL: http://www.UniProt.org/
| UPSOURCEDATE: Thu Apr 18 21:39:39 2024
```

``` output

Please see: help('select') for usage information
```

そのオブジェクトは、クエリを実行し、アノテーションを取得するための*[AnnotationDbi](https://bioconductor.org/packages/3.19/AnnotationDbi)*関数に供給する必要があるものです。

### アノテーションパッケージ内の利用可能な情報をリスト表示する

関数`columns()`を使用してオブジェクト内で利用可能なアノテーションを表示できます。

ここで「カラム」という言葉は、データベース内の情報を格納するために使用されるテーブルのカラムを指し、BioMartの「属性」と同じ概念です。
言い換えれば、カラムはオブジェクトから取得可能なすべてのアノテーションのタイプを表します。

これにより、ユーザーはパッケージ内の利用可能なアノテーションのカラムの名前を記憶する必要がなくなります。


``` r
columns(org.Hs.eg.db)
```

``` output
 [1] "ACCNUM"       "ALIAS"        "ENSEMBL"      "ENSEMBLPROT"  "ENSEMBLTRANS"
 [6] "ENTREZID"     "ENZYME"       "EVIDENCE"     "EVIDENCEALL"  "GENENAME"    
[11] "GENETYPE"     "GO"           "GOALL"        "IPI"          "MAP"         
[16] "OMIM"         "ONTOLOGY"     "ONTOLOGYALL"  "PATH"         "PFAM"        
[21] "PMID"         "PROSITE"      "REFSEQ"       "SYMBOL"       "UCSCKG"      
[26] "UNIPROT"     
```

### キーとキータイプのリスト

データベース用語で言う_キー_は、データベーステーブルから情報をクエリするための値です。

カラムに整理されている情報のため、_キータイプ_はキー値が保存されているカラムの名前です。

データベーステーブル内のカラムの可変数を考慮すると、一部のテーブルでは複数のキーで情報をクエリできる場合があります。
その結果、クエリの一部としてキーとキーの型の両方を指定することが重要です。

関数 `keytypes()` を使用すると、オブジェクトから情報をクエリするために使用できる列の名前を表示できます。


``` r
keytypes(org.Hs.eg.db)
```

``` output
 [1] "ACCNUM"       "ALIAS"        "ENSEMBL"      "ENSEMBLPROT"  "ENSEMBLTRANS"
 [6] "ENTREZID"     "ENZYME"       "EVIDENCE"     "EVIDENCEALL"  "GENENAME"    
[11] "GENETYPE"     "GO"           "GOALL"        "IPI"          "MAP"         
[16] "OMIM"         "ONTOLOGY"     "ONTOLOGYALL"  "PATH"         "PFAM"        
[21] "PMID"         "PROSITE"      "REFSEQ"       "SYMBOL"       "UCSCKG"      
[26] "UNIPROT"     
```

関数 `keys()` を使用すると、特定のキータイプのすべての可能な値を表示できます。

一般的には、クエリされるキーの型を指定する方が良い慣行です（あいまいさを避けるため）。ただし、データベーステーブルには、ユーザーが自分で型を指定しない場合に使用される「主キー」が通常あります。

以下の例では、遺伝子記号のキーのリストを最初のいくつかの値に制限します。
完全なセットは 193279 値を含みます。


``` r
head(keys(org.Hs.eg.db, keytype = "SYMBOL"))
```

``` output
[1] "A1BG"  "A2M"   "A2MP1" "NAT1"  "NAT2"  "NATP" 
```

### アノテーションパッケージから情報をクエリする

関数 `select()` は、主な *[AnnotationDbi](https://bioconductor.org/packages/3.19/AnnotationDbi)* クエリ関数です。
`AnnotationDb` オブジェクト、キー値、および列（および必要に応じて、主キーでない場合は指定されたキーの型）を考えると、アノテーションオブジェクトからユーザーが要求した列を取得します。

以下の例では、いくつかのセクション上の BioMart 例で使用された任意の遺伝子識別子のベクターを再利用します。

`columns()` 関数の出力からわかるように、アノテーションオブジェクトには、BioMart 例でクエリした一部の属性が含まれていません。
この場合、クエリしてみましょう：

- 遺伝子記号
- 遺伝子名
- 遺伝子タイプ


``` r
select(
  x = org.Hs.eg.db,
  keys = query_gene_ids,
  columns = c(
    "SYMBOL",
    "GENENAME",
    "GENETYPE"
  ),
  keytype = "ENSEMBL"
)
```

``` output
'select()' returned 1:1 mapping between keys and columns
```

``` output
          ENSEMBL SYMBOL  GENENAME       GENETYPE
1 ENSG00000133101  CCNA1 cyclin A1 protein-coding
2 ENSG00000145386  CCNA2 cyclin A2 protein-coding
3 ENSG00000134057  CCNB1 cyclin B1 protein-coding
4 ENSG00000157456  CCNB2 cyclin B2 protein-coding
5 ENSG00000147082  CCNB3 cyclin B3 protein-coding
```

*[biomaRt](https://bioconductor.org/packages/3.19/biomaRt)* との小さな違いは、`select()` の出力は、クエリで使用されたキータイプに対応する列を自動的に含むことです。
言い換えれば、取得する列にキータイプを再度指定する必要はありません。

### ベクトル化された 1:1 マッピング

アノテーションが 1 対 多の関係を表示することがある場合があります。
例えば、個々の遺伝子には通常、ユニークな Ensembl 遺伝子識別子があり、複数の遺伝子名別名の下で知られることがあります。

前のセクションで示した `select()` 関数は、指定されたキーに対して要求された列の _すべて_ の値を自動的に返します。
これは、アノテーションが返される表形式であるため可能です。
行が追加され、関連するすべての値と同じ行に表示するために、必要に応じて値を繰り返します。

場合によっては、その動作は望ましくないことがあります。
その代わりに、ユーザーは入力した各キーに対して単一の値を取得したいと考える場合があります。
一般的なシナリオは、差次的遺伝子発現 (DGE) の際に発生し、遺伝子識別子が分析全体で遺伝子を一意に識別するために使用され、遺伝子記号が DGE 統計の最終表に追加され、読みやすい人間に優しい遺伝子識別子を提供します。
ただし、DGE 統計の行を複製することは望ましくないため、各遺伝子に注釈を付けるために単一の遺伝子記号が必要です。

この目的には `mapIds()` 関数を使用できます。
関数 `mapIds()` と `select()` の主な違いは、それぞれの引数 `column`（単数）と `columns`（複数）です。
関数 `mapIds()` は単一の列名を受け入れ、名前付けされた文字ベクターを返します。ここで名前は入力クエリ値であり、値は要求された列内の対応する値です。

1 対 多の関係を処理するために、関数 `mapIds()` には `multiVals` という引数があり、関数が複数の値を処理する方法を指定できます。
デフォルトは最初の値を取得し、他の値は無視することです。

以下の例では、一連の Ensembl 遺伝子識別子の遺伝子記号をクエリします。


``` r
mapIds(
  x = org.Hs.eg.db,
  keys = query_gene_ids,
  column = "SYMBOL",
  keytype = "ENSEMBL"
)
```

``` output
'select()' returned 1:1 mapping between keys and columns
```

``` output
ENSG00000133101 ENSG00000145386 ENSG00000134057 ENSG00000157456 ENSG00000147082 
        "CCNA1"         "CCNA2"         "CCNB1"         "CCNB2"         "CCNB3" 
```

:::::::::::::::::::::::::::::::::::::::  challenge

### Challenge

パッケージ *[EnsDb.Hsapiens.v86](https://bioconductor.org/packages/3.19/EnsDb.Hsapiens.v86)* と *[TxDb.Hsapiens.UCSC.hg38.knownGene](https://bioconductor.org/packages/3.19/TxDb.Hsapiens.UCSC.hg38.knownGene)* を読み込みます。
次に、それらのパッケージで利用可能なアノテーションの列を表示します。

:::::::::::::::  solution

### Solution


``` r
library(EnsDb.Hsapiens.v86)
columns(EnsDb.Hsapiens.v86)
```

``` output
 [1] "ENTREZID"            "EXONID"              "EXONIDX"            
 [4] "EXONSEQEND"          "EXONSEQSTART"        "GENEBIOTYPE"        
 [7] "GENEID"              "GENENAME"            "GENESEQEND"         
[10] "GENESEQSTART"        "INTERPROACCESSION"   "ISCIRCULAR"         
[13] "PROTDOMEND"          "PROTDOMSTART"        "PROTEINDOMAINID"    
[16] "PROTEINDOMAINSOURCE" "PROTEINID"           "PROTEINSEQUENCE"    
[19] "SEQCOORDSYSTEM"      "SEQLENGTH"           "SEQNAME"            
[22] "SEQSTRAND"           "SYMBOL"              "TXBIOTYPE"          
[25] "TXCDSSEQEND"         "TXCDSSEQSTART"       "TXID"               
[28] "TXNAME"              "TXSEQEND"            "TXSEQSTART"         
[31] "UNIPROTDB"           "UNIPROTID"           "UNIPROTMAPPINGTYPE" 
```


``` r
library(TxDb.Hsapiens.UCSC.hg38.knownGene)
columns(TxDb.Hsapiens.UCSC.hg38.knownGene)
```

``` output
 [1] "CDSCHROM"   "CDSEND"     "CDSID"      "CDSNAME"    "CDSPHASE"  
 [6] "CDSSTART"   "CDSSTRAND"  "EXONCHROM"  "EXONEND"    "EXONID"    
[11] "EXONNAME"   "EXONRANK"   "EXONSTART"  "EXONSTRAND" "GENEID"    
[16] "TXCHROM"    "TXEND"      "TXID"       "TXNAME"     "TXSTART"   
[21] "TXSTRAND"   "TXTYPE"    
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: keypoints

- Bioconductor は幅広いアノテーションパッケージを提供します。
- いくつかの Bioconductor ソフトウェアパッケージは、オンラインリソースにプログラム的にアクセスするために使用できます。
- ユーザーは、自分のニーズと期待に基づいてアノテーションのソースを慎重に選択するべきです。

::::::::::::::::::::::::::::::::::::::::::::::::::

[biocviews]: https://www.bioconductor.org/packages/release/BiocViews.html
[biomart-ensembl]: https://www.ensembl.org/biomart/martview
