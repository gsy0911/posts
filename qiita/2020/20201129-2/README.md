# 1. はじめに

みなさん、Azure 楽しんでいますでしょうか？
私は、Azure Functions や Databricks を主に利用し、とても楽しい毎日を送っております。

Azure が提供するサービスはどれもとても素晴らしいです。
ただ、Azure Functions から BlobStorage にアクセスしてファイル操作する際には、既存のライブラリ群ではちょっとラッパー関数が必要になって大変でした。
（もしかしたら、そんな操作をすることは少ないかもなのですが）

そこで、Azure から提供されているライブラリをラッパーして扱いやすいライブラリを作りました。
その名も`AzFS` (Azure File System)です。
リポジトリは[こちら](https://github.com/gsy0911/azfs)です。
（ただ、FileSystem としての機能は2020年11月現在ではほぼないです）

# 2. 利用方法

## AzFSとは

Azure Storage Account 上にある`csv`や`parquet`、`pickle`などのファイルを pandas の`DataFrame`として読み込めるライブラリです。
また、`json`の読み書きや、とあるディレクトリ以下のファイルのリストなども対応しています。

## 2.1 インストール

pip を利用してインストールできます。

```shell
$ pip install azfs
```

## 2.2 インスタンスの作成

動かす環境によって、認証情報の取得方法が異なり、インスタンスに渡す引数が異なります。

現在対応している認証方法は、以下の２つです。

* Azure Active Directory
* Connection String


```python
import azfs

# Azure Active Directory上で動かす場合は、`AzFileClient`への引数は不要です。
azc = azfs.AzFileClient()

# 接続文字列（connection string）を利用する場合
connection_string = "DefaultEndpointsProtocol=https;AccountName=xxxx;AccountKey=xxxx;EndpointSuffix=core.windows.net"
azc = azfs.AzFileClient(connection_string=connection_strin)

```

また、環境変数に以下の情報を入れてあげると、引数なしでも動くようになります。


## 2.3 ファイルのリスト

とあるディレクトリ以下のファイルリストを取得する方法は次の通りです。

```python
path = "https://testazfs.blob.core.windows.net/test_caontainer"
file_list = azc.ls(path)
```

また、特定のファイル名に合致するファイルだけを取得するような`glob関数`も対応しています。

```python
path_pattern = "https://testazfs.blob.core.windows.net/test_caontainer/*.csv"
file_list = azc.glob(path_pattern)
```

## 2.4 ファイルの読み込み

`csv`などのファイルは`pandas-like`と`spark-like`の２通りの方法で読み込みことができます。
また、いずれの方法も pandas の DataFrame を返すようになっています。



| 機能 | pandas-like | pyspark-like |
|:-:|:-:|:-:|
| ファイル読み込み | 単一 | 複数 |
| pandas の引数を渡す | O | O |
| 読み込み時のフィルタ処理 | X | O |
| 並列読み込み | X | O |
| csv | `azc.read_csv()` | `azc.read().csv()` |
| parquet | `azc.read_parquet()` | `azc.read().parquet()` |
| pickle | `azc.read_pickle()` | `azc.read().pickle()` |

 
### 2.4.1 pandas-likeな読み込み

pandas のメソッドのように記述し、DataFrame として読み込むことができます。

```python
# Blob Storageのpathを指定
path = "https://testazfs.blob.core.windows.net/test_caontainer/test_file.csv"
df = azc.read_csv(path)

# pandasの引数にも対応
df = azc.read_csv(path, skiprows=4, header=None)

# with句を利用して、pandasのメソッドのように振る舞うことも可能です。
# read_csv_az という別の関数名になってしまうところだけ注意
import pandas as pd
with azc:
    df = pd.read_csv_az(path)
```

### 2.4.2 spark-likeな読み込み

複数のデータを一度に読むことがある場合は、こちらの方法を利用した方が効率よく読み込めます。

```python
path = "https://testazfs.blob.core.windows.net/test_caontainer/*.csv"
df = azc.read().csv(path)

# また、並列処理も引数を指定するだけです。
df = azc.read(use_mp=True).csv(path)

# 並列処理でファイルを読み込んでいる際に、各ファイルに処理を加えたい場合は、
# 関数を定義し、それを渡してあげると処理します。
def some_function(_df: pd.DataFrame) -> pd.DataFrame:
    _df['new_column'] = _df['price'] * 1.1
    return _df

df = azc.read(use_mp=True).apply(function=some_function).csv(path)

```


## 2.5 ファイルの書き込み

ファイルの書き込みに関しては、`pandas-like`の方法しかないです。

```python
# 書き込み
path = "https://testazfs.blob.core.windows.net/test_caontainer/test_written_file.csv"
azc.write_csv(df, path, index=False)

# readの時と同じようにwith句でpandasのDataFrameから直接関数を実行することも可能です。
# ただし、こちらも関数名がread_csv_azと微妙に異なる点のみ注意してください。
with azc:
    df.write_csv_az(path, index=False)
```

# 3. おわりに

冒頭でもお伝えしたように、`AzFS`は、FileSystem としての機能はほぼないです。
そのため、`Pull Request`や FileSystem にするための`issue`を挙げてくれると大変嬉しいです。

このライブラリを利用して多くの方が Azure ライフを謳歌できたらいいいなと思います。
