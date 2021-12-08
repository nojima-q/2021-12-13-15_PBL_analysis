# がん研究会PBL　解析　〜Transcriptome解析〜
## インストール
・Unix
SRA Toolkit、Trimmomatic、HISAT2、featureCountsはbinary fileをダウンロードして下さい。コンパイルなしでパスを指定するだけで実行できます。\
FastQCはソースコードをダウンロードし、コンパイルすることでコマンド操作で実行できます。またアプリケーションとしても公開されています。
・Windows
Cygwinを利用する場合は、Trimmomaticの実行にJava、HISAT2の実行にPerlが必要です。それぞれインストールして下さい。
## 使用データ
下記のpaired-endでシーケンスされた２サンプルのデータを使用します。\
今回のPBL用に公共データから１サンプル１０万リードランダムサンプリングしたものです。\
ダウンロードして作業ディレクトリに保存して下さい。\
[sample1_1_100K.fastq.gz](https://github.com/nojima-q/2021-12-13-15_PBL_analysis/raw/main/sample1_1_100K.fastq.gz)\
[sample1_2_100K.fastq.gz](https://github.com/nojima-q/2021-12-13-15_PBL_analysis/raw/main/sample1_2_100K.fastq.gz)\
[sample2_1_100K.fastq.gz](https://github.com/nojima-q/2021-12-13-15_PBL_analysis/raw/main/sample2_1_100K.fastq.gz)\
[sample2_2_100K.fastq.gz](https://github.com/nojima-q/2021-12-13-15_PBL_analysis/raw/main/sample2_2_100K.fastq.gz)

## 1-1 公共データベースの紹介
![20190605_metacore](https://user-images.githubusercontent.com/85273234/144177090-bbba1e07-08de-4acf-bf6f-b7395a1e104d.jpg)
### NCBI SRA (https://www.ncbi.nlm.nih.gov/sra)
<img width="1190" alt="スクリーンショット 2021-12-01 14 22 50" src="https://user-images.githubusercontent.com/85273234/144176897-1c463d7f-ca18-41cf-979e-b70fb2db9f0e.png">

### DDBJ Sequence Read Archive (https://www.ddbj.nig.ac.jp/dra/index.html)
<img width="1792" alt="スクリーンショット 2021-12-01 14 23 16" src="https://user-images.githubusercontent.com/85273234/144177226-15d63718-b705-4f71-b103-dc84aa997a14.png">

### EBI ENA (https://www.ebi.ac.uk/ena/browser/home)
<img width="1167" alt="スクリーンショット 2021-12-01 14 23 58" src="https://user-images.githubusercontent.com/85273234/144177440-38a84e15-0555-4ae3-9984-08590f751b7f.png">

### ※データ検索は、スクリーン上で実演します。

## 1-2 データ取得方法
１．解析したデータセットのAccession numberをNCBI SRA Run Selectorに入力し、『Search』をクリック。
<img width="1792" alt="スクリーンショット 2021-12-01 15 13 56" src="https://user-images.githubusercontent.com/85273234/144181792-1ac601bf-88d8-472e-a30f-d554e3b7d5a1.png">
２．データセット内の全てのデータを解析する場合は、Total行の『Accession List』をクリックし、サンプルごとのAccession numberが記載されたテキストファイル（SRR_Acc_List.txt）をダウンロードする。一部のデータのみ解析する場合は、必要なデータに☑をいれSelected行の『Accession List』をクリックし、SRR_Acc_List.txtをダウンロードする。
<img width="1792" alt="スクリーンショット 2021-12-01 15 19 26" src="https://user-images.githubusercontent.com/85273234/144182595-94ed6341-7722-4efb-8eec-5891f67ca4ae.png">
SRR_Acc_List.txtの内容\
<img width="201" alt="スクリーンショット 2021-12-01 15 24 07" src="https://user-images.githubusercontent.com/85273234/144183038-7209d17e-546a-43a9-8216-0f85d0b5d0b1.png">

SRA Toolkitの```prefetch```、```fastq-dump```を使ってデータを取得する。まず、```prefetch```でsraファイルがダウンロードされる。１つのsraファイルが20GBを超える場合は、-Xまたは--max-size 50Gなどのように最大数を変更する。--option-fileは使わずに直接Accession numberを入力して個別にダウンロードすることも可能です。
```
$ prefetch  --option-file ~/Downloads/SRR_Acc_list.txt
```
次に```fastq-dump```でsraファイルからfastqファイルを取得する。PATHにfastqファイルを格納したいディレクトリのパスを記載。\
```
$ find . -name '*.sra' -exec fastq-dump --gzip --split-files --outdir ./PATH {} \;
```
--gzip：圧縮ファイルとして出力する\
--split-files：レイアウトがpaired-endの際に指定する。single-endのときは不要。

## 2 FASTQファイルの形式、クオリティーコントロールについて
### １．FASTQファイルとは
@SRR8615662.1 1 length=101\
TGATGGCCCTGCCTTCGTGGGAACAGAGGCTAAGGCCTTGAG\
+SRR8615662.1 1 length=101\
CCCFFFFFHHHHHJJJJIJJJJIJIJJJJJJJJJJJJJJJJJFIIGIJJIJIJJIJJJJJHHHHHFFFFF\
\
１行目：＠から始まり、リードIDが記載されている\
２行目：シーケンスしたリードの塩基配列\
３行目：＋を記載\
４行目：２行目に記述した各塩基のクオリティ値

### ２．FASTQファイルのクオリティーコントロール
FastQCを使って、FASTQファイルの品質を確認します。\
端末から行う場合、下記のコマンドを実行します。
```
~/FastQC/fastqc -t 40 -o ~/fastqc_results/ ~/sample1_1_100K.fastq.gz
```
-t：スレッド数（使用するPC環境に合わせて設定して下さい。）\
-o：出力ディレクトリ

アプリケーション版を使用する場合は、File>Openでファイルを指定し実行して下さい。\
複数ファイル指定することで、バッチ処理が可能です。
<img width="912" alt="スクリーンショット 2021-12-02 17 19 34" src="https://user-images.githubusercontent.com/85273234/144384259-16f77dc6-b572-4e5e-b865-31aa4e7429d0.png">

### ３．FastQC解析結果
#### Per base sequence quality
リードの各塩基のクオリティスコアを示しています。 Phred quality scoreがだいたいグリーンの領域（Scoreが28以上）に収まっているかどうか確認します。 結果として、クオリティが低いリードは含まれていないことが確認できます。\
<img width="882" alt="スクリーンショット 2021-12-03 12 19 30" src="https://user-images.githubusercontent.com/85273234/144539782-0ec533eb-3533-4e1a-94f6-7b25533e3463.png">

#### Per tile sequence quality
フローセルの各タイルごとのクオリティスコアを示しています。\
Illumina社製の次世代シーケンサーでは、「フローセル」と呼ばれるガラス基板上でDNA合成反応を行います。このフローセルは「タイル」と呼ばれる単位に区切られており、各タイルごとに塩基配列を決定しています。\
シーケンスをかけたときに、例えば特定のタイルに気泡やゴミが入っているとクオリティの低下が見られることがあります。\
特定のタイルで著しいクオリティの低下が見られる場合は、シークエンス時に上記のような問題があったと考えられます。

<img width="883" alt="スクリーンショット 2021-12-03 12 19 40" src="https://user-images.githubusercontent.com/85273234/144539828-47dd135b-650c-46ea-9c12-0467566b7cbf.png">

#### Per sequence quality scores
各リードの全長のクオリティスコアの分布を示しています。 
<img width="854" alt="スクリーンショット 2021-12-03 12 19 59" src="https://user-images.githubusercontent.com/85273234/144539876-2bb62a89-aed8-486b-892c-212b3e9943c9.png">

#### Per base sequence content
各塩基のA, T, G, Cの含有量を示しています。 RNA-seqの場合、それぞれの含有量はほぼ25%ずつになりますが、PAR-CLIPのようにRNA結合タンパク質と結合しているRNA配列を抽出してきている場合、それぞれの含有率に偏りが見られます。
<img width="882" alt="スクリーンショット 2021-12-03 12 20 10" src="https://user-images.githubusercontent.com/85273234/144539910-8ddb14a5-cba5-4ae3-bcb1-efe42fe7320f.png">

#### Per sequence GC content
リードのGC contentsの分布を示しています。 
<img width="861" alt="スクリーンショット 2021-12-03 12 20 18" src="https://user-images.githubusercontent.com/85273234/144539949-a255551b-f00c-4c80-907c-b5c93767308a.png">

#### Per base N content
各塩基中に含まれるNの含有率（塩基を読めなかった箇所）を示しています。 
<img width="880" alt="スクリーンショット 2021-12-03 12 20 28" src="https://user-images.githubusercontent.com/85273234/144540515-9f9c67a5-b570-4262-8097-4d8f36ab8279.png">

#### Sequence Length Distribution
リード長の分布を示しています。 
<img width="855" alt="スクリーンショット 2021-12-03 12 20 37" src="https://user-images.githubusercontent.com/85273234/144540533-0a74b05a-fc62-4892-aad9-50443deeed4c.png">

#### Sequence Duplication Levels
Duplidate readsの含まれている数を示しています。 
<img width="855" alt="スクリーンショット 2021-12-03 12 20 37" src="https://user-images.githubusercontent.com/85273234/144540533-0a74b05a-fc62-4892-aad9-50443deeed4c.png">

#### Overrepresented sequences
頻出する特徴配列が示されています。リード中にアダプター配列などが混入している場合、その配列が示されます。
<img width="879" alt="スクリーンショット 2021-12-03 13 17 24" src="https://user-images.githubusercontent.com/85273234/144544469-dc72869e-1df7-4de3-af65-a9d6d6ef6dc1.png">

#### Adapter Content
各塩基ごとに見たときのリード中に含まれているアダプターの割合を示しています。 あくまで、FastQCに登録されているアダプター配列しか確認していないので、登録されていないアダプター配列を使っていた場合、そのアダプター配列がリード中に混入していても確認できないことがあります。 
<img width="898" alt="スクリーンショット 2021-12-03 13 15 53" src="https://user-images.githubusercontent.com/85273234/144544371-74ddb7ed-97d1-4ffa-b934-50ef78ebd7e5.png">

## 3 アダプターの除去およびリードのトリミング
Trimmomaticを使って、アダプターの除去および低スコアな塩基のトリミングを同時に行います。
アダプター配列を記載したFATSAファイルは下記よりダウンロードして下さい。（今回は、illumina社のTruSeqシリーズの配列を記載しています。自前データで実行する際は、ライブラリー作製キットで使用している配列が記載されたFASTAファイルを用意して実行して下さい。）\
下記はpaired-endでシーケンスしたFASTQファイルの場合です。
```
java -jar ~/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 4 -phred33\
./sample1_1_100K.fastq.gz\ #処理前Fowardリード
./sample1_2_100K.fastq.gz\ #処理前Reverseリード
./sample1_1_100K_trim_paired.fastq.gz\ #処理後ReverseリードとペアなFowardリード
./sample1_1_100K_trim_unpaired.fastq.gz\ #処理後ReverseリードとペアでないFowardリード
./sample1_2_100K_trim_paired.fastq.gz\ #処理後FowardリードとペアなReverseリード
./sample1_2_100K_trim_unpaired.fastq.gz\ #処理後FowardリードとペアでないReverseリード
ILLUMINACLIP:Truseq_stranded_totalRNA_adapter.fa:2:30:10 LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:25 #トリミング条件
```
ILLUMINACLIP：Truseq_stranded_totalRNA_adapter.faはillumina社のTruSeqシリーズのアダプター配列をFASTA形式で記載してもの。後ろの数字は、許容ミスマッチ数:palindrome clip threshold:simple clip thresholdを表す。\
LEADING：5'末端からスコアが 設定したスコア値未満の塩基をトリム\
TRAILING：3'末端からスコアが 設定したスコア値未満の塩基をトリム\
SLIDINGWINDOW：左の数字はウィンドウサイズ、右の数字は平均クオリティ値を表します。ウィンドウサイズの範囲内の塩基のスコア平均が設定値よりも低ければ、3'末端側の全ての塩基をトリム。\
MINLEN：設定値未満の塩基数になったリードを除去する。

