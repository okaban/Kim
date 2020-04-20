# Work Flow

## Assignment 1
**サーバーの環境構築を行う**

sshコマンドでサーバーにアクセスする:
```
$ ssh [ユーザー名]@cacao.bioinfo.ttck.keio.ac.jp
```
UNIXコマンドを復習したい場合は以下のリンクを確認すること
- [UNIXコマンド入門 [一般ユーザー編] (全24回) - プログラミングならドットインストール](https://dotinstall.com/lessons/basic_unix_v2)

Downloadsディレクトリを作成する:
```
$ mkdir Downloads
$ cd Downloads
```

カーネルを確認する:
```
$ uname -a
```
minicondaをインストールする:
```
$ wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ bash Miniconda2-latest-Linux-x86_64.sh
```
インストール時に色々聞かれるが、基本的に`Enter`を押すか、`Yes`を入力すればいい。
最終的にカレントディレクトリに`miniconda2`というディレクトリができており、.bashrcに以下の文が追加されている。
```
> .bashrc
# added by Miniconda2 4.2.12 installer
export PATH="/home/user/miniconda2/bin:$PATH"
```
インストール後、`.bashrc`を再度読み込む:
```
$ source ~/.bashrc
```
condaが正常にインストールされているかチェックする:
```
$ conda -h
```
biocondaのチャンネルを追加する:
```
$ conda config --add channels conda-forge
$ conda config --add channels defaults
$ conda config --add channels r
$ conda config --add channels bioconda
```
Brigerをインストールする:
```
conda install -c bioconda bioconductor-bridge
```
基本的に`Enter`と`Yes`でどうにかなる。

---
## Assignment 2
**ゲノムデータおよびトランスクリプトームデータをダウンロードする**

ホームディレクトリに戻る:
```
cd ~
```
プロジェクト用のディレクトリと各種ファイルを作成する:
```
mkdir my_projects
mkdir my_projects/vollenhovia
cd my_projects/vollenhovia
mkdir data alaysis scripts
```
Genbankのassembly_summary（全Genbankファイルの一覧）をダウンロードする:
```
cd data
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/ASSEMBLY_REPORTS/assembly_summary_genbank.txt
```
ウメマツアリのゲノム情報を抽出する:

ここでは検索するキーワードとして、[Accession ID](https://www.ncbi.nlm.nih.gov/bioproject/275948)を使用する。
```
grep "PRJDB3517" assembly_summary_genbank.txt
```
表示されたURLをコピーして、ダウンロードする
```
mkdir emeryi
cd emeryi
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/949/405/GCA_000949405.1_V.emery_V1.0/*gz ./
gunzip *gz
```

**参考**
- [Linuxが32bitか64bitを確認する](https://linux.just4fun.biz/?%E9%80%86%E5%BC%95%E3%81%8DUNIX%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89/Linux%E3%81%8C32bit%E3%81%8B64bit%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B)
- [biocondaを利用してNGS関連のソフトウェアを一括でインストールする](http://imamachi-n.hatenablog.com/entry/2017/01/14/212719)
- [bioconda/packages/bioconductor-bridge](https://anaconda.org/bioconda/bioconductor-bridge)