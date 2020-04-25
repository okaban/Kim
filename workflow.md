# ワークフロー

## ステップ 1
**サーバーの環境構築を行う**

sshコマンドでサーバーにアクセスする。
```
$ ssh [ユーザー名]@cacao.bioinfo.ttck.keio.ac.jp
```
UNIXコマンドを復習したい場合は以下のリンクを確認すること
- [UNIXコマンド入門 [一般ユーザー編] (全24回) - プログラミングならドットインストール](https://dotinstall.com/lessons/basic_unix_v2)

Downloadsディレクトリを作成する。
```
$ mkdir Downloads
$ cd Downloads
```

カーネルを確認する。
```
$ uname -a
```
minicondaをインストールする。インストール時に色々聞かれるが、基本的に`Enter`を押すか、`Yes`を入力すればいい。
最終的にホームディレクトリに`miniconda3`というディレクトリが作成されたらOK。
```
$ wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ bash Miniconda3-latest-Linux-x86_64.sh
```
vimを立ち上げて`~/.bashrc`に以下のスクリプトを追記する。
```
# miniconda3のパス
export PATH="~/miniconda3/bin:$PATH"
```
`~/.bashrc`を再度読み込む。
```
$ source ~/.bashrc
```
condaが正常にインストールされているかチェックする。
```
$ conda -h
```
biocondaのチャンネルを追加する。
```
$ conda config --add channels conda-forge
$ conda config --add channels defaults
$ conda config --add channels r
$ conda config --add channels bioconda
```
Brigerをインストールする。
```
$ conda install -c bioconda bioconductor-bridge
```
基本的に`Enter`と`Yes`でどうにかなる。

**参考**
- [Linuxが32bitか64bitを確認する](https://linux.just4fun.biz/?%E9%80%86%E5%BC%95%E3%81%8DUNIX%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89/Linux%E3%81%8C32bit%E3%81%8B64bit%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B)
- [biocondaを利用してNGS関連のソフトウェアを一括でインストールする](http://imamachi-n.hatenablog.com/entry/2017/01/14/212719)
- [bioconda / packages / bioconductor-bridge 1.50.0](https://anaconda.org/bioconda/bioconductor-bridge)

---
## ステップ 2
**ウメマツアリのゲノムデータをダウンロードする**

ホームディレクトリに戻る。
```
$ cd ~/
```
プロジェクト用のディレクトリと各種ファイルを作成する。
```
$ mkdir my_projects
$ mkdir my_projects/vollenhovia
$ cd my_projects/vollenhovia
$ mkdir data analysis scripts
```
Genbankのassembly_summary（全Genbankファイルの一覧）をダウンロードする。
```
$ cd data
$ wget ftp://ftp.ncbi.nlm.nih.gov/genomes/ASSEMBLY_REPORTS/assembly_summary_genbank.txt
```
ウメマツアリのゲノム情報を抽出する。
ここでは検索するキーワードとして、[Accession ID](https://www.ncbi.nlm.nih.gov/bioproject/275948)を使用する。
```
$ grep "PRJDB3517" assembly_summary_genbank.txt
```
表示されたURLをコピーして、ダウンロードする。
```
$ mkdir emeryi
$ cd emeryi
$ wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/949/405/GCA_000949405.1_V.emery_V1.0/*gz ./
$ gunzip *gz
```

---
## ステップ 3
**ヤドリウメマツアリのトランスクリプトームデータをダウンロードする**

neoサーバーに接続する。
```
ssh neo
```

sratoolkitをインストールする。
`-y`を付与するとyes/noを聞かれずにインストールが進行する。
```
$ conda install -c bioconda sra-tools -y
```
ダウンロード先のディレクトリを作成する。
```
$ cd ~/my_projects/vollenhovia
$ mkdir nipponica
$ cd nipponica
```
WebブラウザでVollenhovia nipponicaの[BioProject](https://www.ncbi.nlm.nih.gov/bioproject/?term=422773)から[SRA Experiments](https://www.ncbi.nlm.nih.gov/sra?linkname=bioproject_sra_all&from_uid=422773)を開く。
![](https://i.gyazo.com/982a1a6ddfff798b214688c7e7b08cab.png)
Sra Experimentsの一覧からAccession IDを抽出する。右上の"Send to"から"File">"Accession List">"Creat File"を選択する。
![](https://i.gyazo.com/238b04c139f76c847f6c0aa8e975338b.png)
自分のPCにダウンロードしたSraAccList.txtをテキストエディタで開き、クリップボードにコピーする。サーバー上のカレントディレクトリにSraAccList.txtを作成する。
```
vi SraAccList.txt
```
vimを編集モード（"i"を入力）にしてクリップボードのテキストをペーストする。編集が終わったらvimを終了する（"esc"+"ZZ"）。同様の操作で以下のスクリプトファイル（run-sratool.sh）を作成する。
```
#run-sratool.sh

#!/bin/bash
set -euo pipefail

## Download sra data
prefetch -O sra --option-file SraAccList.txt

## Splist fo fastq data
sra_list=$(<SraAccList.txt)
for sra_id in ${sra_list}
do
fastq-dump $(pwd)/sra/${sra_id}/${sra_id}.sra --split-files -O fastq
done
```
run-sratool.shを起動する。ダウンロードにはかなりの時間がかかるため、ログアウトしてもコマンドが終了しないように`nohup`コマンドを使用する。log.fastq-dump.txtmにはダウンロードのログが記録される。
```
$ (nohup bash run-sratool.sh &) >& log.sratool.txt
```
run-sratool.shが実行されたら、バックグラウンドで実行されているかを確認する。自分のユーザー名（USERに表示）でprefetch（COMMANDを確認）が実行されていればOK。
```
$ top
```
ダウンロードが完了すると、`~/my_projects/vollenhovia/data/nipponica/fastq`に計55個のファイルが作成される。うちDRR050273-278はシングルエンドリード、DRR050279-302はペアエンドリードである。
![](https://i.gyazo.com/209a6950663f60885116d055af62d489.png)

**参考**
- [bioconda / packages / sra-tools 2.10.3](https://anaconda.org/bioconda/sra-tools)
- [SRA Toolkit 使い方 公開データのダウンロードとsra fastq変換](http://bioinfo-dojo.net/2017/04/19/sra-toolkit_data_download_sra_fastq/)
- [【 nohup 】コマンド――端末を閉じてもログアウトしても処理を続ける](https://www.atmarkit.co.jp/ait/articles/1708/24/news022.html)
