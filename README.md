# git-largefile

Manage binary files with git.

## How it works

コミットするときにハッシュ値だけをコミットし、ファイルの実態は `~/.gitasset/data`
に格納します。

別のマシンでチェックアウトする場合は、 `~/.gitasset/data` を rsync しておきます.

## Setup

### Install

gits3をパスが通った場所等に配置してください。

### S3 Configuration

予め S3 にアクセスできるキーとバケットを作っておいてください。

`~/.gitasset/gits3.ini` に次のように書いてください:

```
[DEFAULT]
awskey = Access Key Id:Secret Access Key
bucket = バケット名
```

### gitconfig

`~/.gitconfig` か `.git/config` に、次のように設定してください

```
[filter "s3"]
    clean = /path/to/gits3 store
    smudge = /path/to/gits3 load
    required
```

### gitattribute

git リポジトリの中に `.gitattributes` っていうファイルを作って、次のように設定してください。

```
*.png  filter=s3
*.jpeg filter=s3
*.jpg  filter=s3
*.gif  filter=s3
```

これで設定したファイルは largefile フィルターを通るようになります.



### アップロードの並列処理

ローカルモードを有効にしs3アップロードを停止する
`~/.gitconfig` か `.git/config` に、次のように設定してください

```
[filter "s3"]
    clean = /path/to/gits3 -local=true store
    smudge = /path/to/gits3 load
    required
```

通常のgit操作を一通り行った後に、
以下のコマンド実行でまとめてs3にアップロードします

```
$ gits3 -n=<並列数> upload
```

##### 並列アップロードの実験

79個のファイルをアップロード
```
$ find ~/.gitasset/data/ -type f |wc
79      79    5767
```

まずは並列なし
```
$ time ~/gits3 -n=1 upload
real	0m8.841s
user	0m0.548s
sys	0m0.183s
```
20並列で実行
```
$ time ./gits3 -n 20 upload
real	0m0.787s
user	0m0.493s
sys	0m0.121s
```


