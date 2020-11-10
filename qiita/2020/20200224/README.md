

本記事は、`PySpark`の特徴とデータ操作をまとめた記事です。

# PySparkについて

## PySpark(Spark)の特徴

* ファイルの入出力
  * 入力：単一ファイルでも可
  * 出力：出力ファイル名は付与が不可（フォルダ名のみ指定可能）。指定したフォルダの直下に複数ファイルで出力。
* 遅延評価
  * ファイル出力時 or 結果出力時に処理が実行
  * 通常は実行計画のみが計算

## `Partitioning` と `Bucketing`

`PySpark`の操作において重要な`Apache Hive`の概念について。

* Partitioning: ファイルの出力先をフォルダごとに分けること。読み込むファイルの範囲を制限できる。
* Bucketing: ファイル内にて、ハッシュ関数によりデータを再分割すること。効率的に読み込むことができる。

`Partitioning`と`Bucketing`の詳細については[こちら(英語)](https://data-flair.training/blogs/hive-partitioning-vs-bucketing/)をご覧ください。

## 計算リソース使用状況の確認
データの処理が遅い場合は、`Ganglia`を使って計算リソースの使用状況を見てみると良いです。

特に、ネットワーク通信量（＝データ転送量）が低くて、処理に時間がかかることが多いです。
この場合は、以下のような方法で対策をすると解決する場合があります。

* 単一ファイルよりも分散ファイルを読み込む
* 遅延評価のため、多くの処理を行った結果は出力が遅く、以下のどちらかを利用する
  * データフレームのキャッシュを利用：例 `df = df.cache()`
  * フォルダに一旦吐き出し、再度出力結果を読み込み、後続の処理を実行

# PySparkのコード片

以下の変数は生成済みとしています。
* `spark`: spark context
* `path`: なにかしらのファイルパス
* 次項で `import` した要素


**注意**

1. `> df.show()` で表示される内容は、イメージを掴むことが目的のため、必ずしも正しくない場合があります。
1.  上記であげた変数以外の変数を突然利用している場合もります。
1. AWS 上で実行することをイメージしているため、パスは`s3://`から始まっています。
1. 順次追記・修正をしていく予定です。
1. **構文や書いていることに間違がある場合はコメントをよろしくおねがいいたします。**

## import

`spark`利用時に主に import するのは以下の項目です。

```
# from pyspark.sql.functions import * とする場合もありますが、
# Fで関数の名前空間を明示した方がわかりやすくて好きです。
# ただ、FだとPEP8に違反していますが。。。
from pyspark.sql import functions as F
from pyspark.sql.types import FloatType, TimestampType, StringType
from pyspark.sql.window import Window
```

## 実行環境設定

* AWS 上の`EMR`を利用する場合は、インスタンス上の時刻が`UTC`のため、`JST`に設定

```
spark.conf.set("spark.sql.session.timeZone", "Asia/Tokyo")
```

## initialize spark
`EMR`の`JupyterHub`上では必要ありませんが、`Python`の script で実行する場合は、
`spark`のインスタンス初期化が必要です。

```
# spark initialization
spark = SparkSession.builder.appName("{your app name here}").getOrCreate()
```


## データ読込

* 文字列から読み込む場合

```
df = spark.read.parquet(path)
```

* `*`を利用して複数ファイル／フォルダを一度に読み込み可能

```
# dt=2020-01-01/ 以下にあるファイルを全て読み込む
df = spark.read.parquet("s3://some-bucket/data/dt=2020-01-01/*.parquet")

# dt=2020-01-01/ から dt=2020-01-31/ 以下にあるファイルを全て読み込む
df = spark.read.parquet("s3://some-bucket/data/dt=2020-01-*/*.parquet")
```

* from list<string>: 複数フォルダにまたがる場合はこちらでも可

```
# pathsリストに含まれるパスにあるファイルを読み込む
df = spark.read.parquet(*paths)
```

* `partition`を利用していて、パーティションを読み込んだデータフレームのカラムに入れたい場合

```
# partitionを列に加えて読み込む
df = spark.read.option("basePath", parent_path).parquet(*paths)
```

* データフレームのキャッシュ保存

遅延評価した結果をメモリに保存しておくことで、高速に処理が可能になります。
よく利用するデータを`cache()`し、特に処理の後段で利用するのがよいです。

```
# オンメモリのキャッシュ
df = df.cache()
# または
# デフォルトではオンメモリのキャッシュ、オプション引数でキャッシュ先をストレージなどに変更可能
df = df.persist()
```

* データ型の確認

```
df.printSchema()

root
 |-- id: string (nullable = true)
 |-- name: string (nullable = true)
```

## データ出力

```
# csv（この場合はheaderが付与されない）
df.write.csv(path)

# parquet
df.write.parquet(path)
```

* header: csv の場合のみ注意が必要

```
# csvの場合はheaderの出力設定をしないと付与されない
df.write.mode("overwrite").option("header", "True").csv(path)
# or
df.write.mode("overwrite").csv(path, header=True)
# parquetの場合はheaderを指定しなくてもdefaultで出力される
df.write.parquet(path)
```

* compression: 圧縮

```
# gzip with csv
df.write.csv(path, compression="gzip")

# snappy with parquet（デフォルトでsnappy圧縮されるはず？）
df.write.option("compression", "snappy").parquet(path)
```

* partitionBy: 出力する際にデータフレームのカラム名で`partition`をしたい場合

以下の例の場合`/dt={dt_col}/count={count_col}/{file}.parquet`というフォルダに出力されます。
```
df.repartition("dt", "count").write.partitionBy("dt", "count").parqeut(path)
```

* coalesce: 通常は複数ファイルで出力される内容を１つのファイルにまとめて出力可能

複数処理後に`coalesce`を行うと処理速度が落ちるため、可能ならば一旦通常にファイルを出力し、再度読み込んだものを`coalesce`した方がよいです。

```
# 複数処理後は遅くなることがある
df.coalesce(1).write.csv(path, header=True)

# 可能ならばこちらを推奨（出力→読み込み→出力）
df.write.parquet(path)
alt_df = spark.read.parquet(path)
alt_df.coalesce(1).write.csv(path, header=True)
```

* repartition: 出力するファイルの分割数を指定

```
df.repartition(20).write.parquet(path)
```

* write.mode(): 出力時の方法を選択可能

```
# write.mode()で使用できる引数 'overwrite', 'append', 'ignore', 'error', 'errorifexists'
# よく利用するのは overwrite
# 通常は出力先のフォルダにファイルが存在した場合はエラーがでる
df.write.parquet(path)

# 上書き保存したい場合
df.write.mode("overwrite").parquet(path)

# 現在のフォルダに追記したい場合
df.write.mode("append").parquet(path)
```

## データフレームの生成

ファイル読み込みからではなく、プログラム上でデータフレームを作成する方法です。

* 単一カラムの場合

```
# 単一カラムのデータフレームを作成
id_list = ["A001", "A002", "B001"]
df = spark.createDataFrame(id_list, StringType()).toDF("id")
```

* 複数カラムの場合

```
# 中の要素はtuple, 最後にカラムの名前を指定する
df = spark.createDataFrame([
    ("a", None, None),
    ("a", "code1", None),
    ("a", "code2", "name2"),
], ["id", "code", "name"])

> df.show()
+---+-----+-----+
| id| code| name|
+---+-----+-----+
|  a| null| null|
|  a|code1| null|
|  a|code2|name2|
+---+-----+-----+

# =======================
# rddを一旦利用して作成する場合
rdd = sc.parallelize(
    [
        (0, "A", 223, "201603", "PORT"), 
        (0, "A", 22, "201602", "PORT"), 
        (0, "A", 422, "201601", "DOCK"), 
        (1, "B", 3213, "201602", "DOCK"), 
        (1, "B", 3213, "201601", "PORT"), 
        (2, "C", 2321, "201601", "DOCK")
    ]
)
df_data = spark.createDataFrame(rdd, ["id","type", "cost", "date", "ship"])

> df.show()
+---+----+----+------+----+
| id|type|cost|  date|ship|
+---+----+----+------+----+
|  0|   A| 223|201603|PORT|
|  0|   A|  22|201602|PORT|
|  0|   A| 422|201601|DOCK|
|  1|   B|3213|201602|DOCK|
|  1|   B|3213|201601|PORT|
|  2|   C|2321|201601|DOCK|
+---+----+----+------+----+
```



## 列の追加（`withColumn()`）

`PySpark`では「新しい列を追加する処理」を利用して分析することが多いです。


```
# new_col_nameという新しい列を作成し、1というリテラル値（＝定数）を付与
df = df.withColumn("new_col_name", F.lit(1))
```


* F.input_file_name(): 読み込んだファイル名を取得

```
# 読み込んだファイルパスを付与
df = df.withColumn("file_path", F.input_file_name())

# 読み込んだファイルパスからファイル名を取得
df = df.withColumn("file_name", F.split(col("file_path"), "/").getItem({int: 最後のindex値}))
```


* cast(): 型変換

```
# 文字列で指定
df = df.withColumn("total_count", F.col("total_count").cast("double"))

# PySparkのtypesで指定        
df = df.withColumn("value", F.lit("1").cast(StringType()))
```

* F.when().otherwise(): 条件に応じて追加する値を変更

```
# 条件に応じた値の列を追加したい場合
# F.when(condtion, value).otherwise(else_value)
df = df.withColumn("is_even", F.when(F.col("number") % 2 == 0, 1).otherwise(0))

# 複数条件の場合
df = df.withColumn("search_result", F.when( (F.col("id") % 2 == 0) & (F.col("room") % 2 == 0), 1).otherwise(0))

```

* isNotNull(): null かどうかを判定

```
df = df.withColumn("is_touched", F.col("value").isNotNull())
```

* F.regexp_replace(): 正規表現を利用した文字の置換

```
df = df.withColumn("replaced_id", F.regexp_replace(F.col("id"), "A", "C"))
```

* 時間に関する操作

```
# date time -> epoch time
df = df.withColumn("epochtime", F.unix_timestamp(F.col("timestamp"), "yyyy-MM-dd HH:mm:ssZ"))

# epoch time -> date time
# 1555259647 -> 2019-04-14 16:34:07
df = df.withColumn("datetime", F.to_timestamp(df["epochtime"]))

# datetime -> string
# 2019-04-14 16:34:07 -> 2019-04-14
string_format = "yyyy-MM-dd"
df = df.withColumn("dt", F.date_format(F.col("datetime"), string_format))

# epoch time: およそ10桁の数字列。1970年1月1日からの秒数
df = df.withColumn("hour", F.hour(F.col("epochtime")))
df = df.withColumn("hour", F.hour(F.col("timestamp")))

# datetimeを、指定した時間幅に切り捨ている
df = df.withColumn("hour", F.date_trunc("hour", "datetime"))
df = df.withColumn("week", F.date_trunc("week", "datetime"))

# 時間の加算
df = df.withColumn('hour', F.col("hour") + F.expr('INTERVAL 2 HOURS'))
df1.show(truncate=False)
```

このほかにもたくたん `withColumn`にて利用できる関数はたくさんあります。
参考サイトもご覧ください。

## データフレームの結合

２つの`DataFrame`を横／縦に結合するメソッドは`join()/union()`です。

```
# onで結合する列を指定する
df = left_df.join(right_df, on="id")

# data-frameごとに異なる列の場合
df = left_df.join(right_df, left_df.id_1 == right_df.id_2)

# 結合方法も指定可能
# how:= inner, left, right, left_semi, left_anti, cross, outer, full, left_outer, right_outer
df = left_df.join(right_df, on="id", how="inner")
```

* 複数カラムで結合

```
df = left_df.join(right_df, on=["id", "dt"])
```

* F.broadcast() join: データを各クラスタに効率的に分配し、結合する方法
  * 各データフレームのデータサイズが以下のように不均衡の場合に使うと効率が上昇（することがある）
    * left_df: データ量：多、例：実データ
    * right_df: データ量：少、例：マスタデータ

```
df = left_df.join(F.broadcast(right_df), on="id")
```

* union(): データフレームを縦方向に結合

```
df = upper_df.union(bottom_df)
```

## カラム操作 (rename, drop, select)

* withColumnRenamed(before, after): カラム名の変更
 
よくカラム名のない csv を読み込んだときに利用することが多いです。

```
# カラム名がない場合、`_c0`から`_c{n}`というカラム名が与えられる
df = df.withColumnRenamed("_c0", "id")
```

* select("col_1", "col_2", ...): 列単位で取得

```
df = df.select("id")
```

* distinct(): 重複削除

```
df = df.select("id").distinct()
# count() と合わせてよく利用する
# 例：とあるidのユニーク数
print(df.select("id").distinct().count())
```

* drop("col_1", "col_2", ...): 列単位で削除

```
df = df.drop("id")
```

* drop Null Value

```
# simple
df = df.dropna()
# using subset
df = df.na.drop(subset=["lat", "lon"])
```


* collect_list(), collect_set(): 単一のカラムにリストとして値を入力

```
# 単純な場合
df = df.select("id").select(F.collect_list("id"))
id_list = df.first()[0]

> id_list => ["A001", "A002", "B001"]

# groupByと合わせて使うことも可能
df = df.groupBy("id").agg(F.collect_set("code"), F.collect_list("name"))

> 
+---+-----------------+------------------+
| id|collect_set(code)|collect_list(name)|
+---+-----------------+------------------+
|  a|   [code1, code2]|           [name2]|
+---+-----------------+------------------+
```

* collect(): 全ての要素を返す
* take(n): 最初の`n`個の要素を返す
* first(): 一番最初の要素を返す

```
# データフレームの値を直接取得する
df = df.groupBy().avg()
avg_attribute = df.collect()[0]

> print(avg_attribute["avg({col_name})"])
{averaged_value}
```

## filter

`F.col()`を利用して特定のカラムに対してフィルタ処理を適用できます。

```
# using spark.function
df = df.filter(F.col("id") == "A001")

# pandas-like
df = df.filter(df['id'] == "A001")
df = df.filter(df.id == "A001")
```

* isin(): list の中の要素にある値かどうかを判定

ただ、可能ならば date_list から`Spark`データフレームを作成し、join したほうがよいです。

```
df = df.filter(F.col("dt").isin(date_list))
```

## orderBy

ソートは分散処理では適さない処理のため、あまりしない方が良いです。

```
# 単一カラムのみ
df = df.orderBy("count", ascending=False)

# 複数条件ソート
df = df.orderBy(F.col("id").asc(), F.col("cound").desc())
```

## groupBy (aggregate)

```
# count()
df = df.groupBy("id").count()

# multiple
# alias() 関数にてカラム名の変更を行なっている
# 例：ユーザーの集計
df = df.groupBy("id").agg(
  F.count(F.lit(1)).alias("count"),
  F.mean(F.col("diff")).alias("diff_mean"),
  F.stddev(F.col("diff")).alias("diff_stddev"),
  F.min(F.col("diff")).alias("diff_min"),
  F.max(F.col("diff")).alias("diff_max")
)

> df.show()
（省略）

# =======================
# 例：ユーザーの日時ごとの集計
df = df.groupBy("id", "dt").agg(
  F.count(F.lit(1)).alias("count")
  )
  
> df.show()
+---+-----------+------+
| id|         dt| count|
+---+-----------+------+
|  a| 2020/01/01|     7|
|  a| 2020/01/02|     5|
|  a| 2020/01/03|     4|
+---+-----------+------+

# ===========================
# 例：ユーザーの日時・場所ごとの集計
df = df.groupBy("id", "dt", "location_id").agg(
  F.count(F.lit(1)).alias("count")
  )

> df.show()
+---+-----------+------------+------+
| id|         dt| location_id| count|
+---+-----------+------------+------+
|  a| 2020/01/01|           A|     2|
|  a| 2020/01/01|           B|     3|
|  a| 2020/01/01|           C|     2|
:   :           :            :      :
+---+-----------+------------+------+
```


* countDistinct(): 集計時に重複を削除し、count を行う

```
# 例：日付ごとのユーザユニーク数
df = df.groupBy("dt").agg(countDistinct("id").alias("id_count"))

> df.show()
+-----------+---------+
|         dt| id_count|
+-----------+---------+
| 2020/01/01|        7|
| 2020/01/02|        5|
| 2020/01/03|        4|
+-----------+---------+

# ===============================
# 例：ユーザごとの接触が1回以上あった日数
df = df.groupBy("id").agg(countDistinct("dt").alias("dt_count"))

> df.show()
+---+---------+
| id| dt_count|
+---+---------+
|  a|       10|
|  b|       15|
|  c|        4|
+---+---------+
```

* 要素の指定に list も指定可能

```
group_columns = ["id", "dt"]
df = ad_touched_visit_df.groupBy(*group_columns).count()
```

## window関数

* row_number(): 行番号を付与

```
w = Window().orderBy(F.col("id"))
df = df.withColumn("row_num", F.row_number().over(w))
```

* lag(): 行をずらす

```
# 前の行のデータをカラムとして追加
w = Window.partitionBy("id").orderBy("timestamp")
df = df.withColumn("prev_timestamp", F.lag(df["timestamp"]).over(w))
```

## loop処理

分散環境と相性が悪いため、強く非推奨です。
どうしても for じゃないといけない場合にのみ利用するようにした方がいいです。

```
for row in df.rdd.collect():
  do_some_with(row['id'])
```


# 参考サイト

* [Pyspark dataframe操作](https://qiita.com/wwacky/items/e687c0ef05ae7f1de980)
* [SparkSQLリファレンス](https://x1.inkenkun.com/archives/1202)
* [select_list](https://stackoverflow.com/questions/37580782/pyspark-collect-set-or-collect-list-with-groupby)
* [Add hour](https://www.datasciencemadesimple.com/add-hours-minutes-and-seconds-to-timestamp-in-pyspark/)


