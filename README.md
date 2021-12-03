# がん研究会PBL　解析　〜Transcriptome解析〜
## インストール
・SRA Toolkit、Trimmomatic、HISAT2、featureCountsはbinary fileをダウンロードして下さい。コンパイルなしでパスを指定するだけで実行できます。\
・FastQCはソースコードをダウンロードし、コンパイルすることでコマンド操作で実行できます。またアプリケーションとしても公開されています。

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

### 1-2 データ取得方法
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

### 2 FASTQファイルの形式、クオリティーコントロールについて
#### １．FASTQファイルとは
@SRR8615662.1 1 length=101\
TGATGGCCCTGCCTTCGTGGGAACAGAGGCTAAGGCCTTGAG\
+SRR8615662.1 1 length=101\
CCCFFFFFHHHHHJJJJIJJJJIJIJJJJJJJJJJJJJJJJJFIIGIJJIJIJJIJJJJJHHHHHFFFFF\
\
１行目：＠から始まり、リードIDが記載されている\
２行目：シーケンスしたリードの塩基配列\
３行目：＋を記載\
４行目：２行目に記述した各塩基のクオリティ値

#### ２．FASTQファイルのクオリティーコントロール
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

#### ３．FastQC解析結果
##### Per base sequence quality
