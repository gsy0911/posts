# PySparkについて

# DataFlow

![DataFlow_png](https://www.tutorialspoint.com/pyspark/images/sparkcontext.jpg)

## RDD : Resilient Distributed Dataset

* 「不変（immutable）で並列実行可能な（分割された）コレクション」のこと

> Apache Sparkのプログラミングでは、このRDDにデータを保持して操作することがメインとなります。RDDの操作には用意されているメソッドを使うことで、Sparkは自動的に分散処理を行い、開発者は分散処理を意識することなくプログラミングできます。

## RDDメソッドの種類

* transformations
> 「Transformations」はRDDを操作し、結果を新しいRDDとして返します。

| method | 説明 |
|:--:|:--:|
| map |  |
| filter |  |
| union |  |
| flatMap |  |

* Actions
> 「Actions」はRDDのデータを操作し、結果をRDD以外の形式で返すか保存を行います。


| method | 説明 |
|:--:|:--:|
| reduce |  |
| count |  |

> 大雑把に言えば、RDDを返すのが「Transformations」、そうではないものが「Actions」である。

# 参考

* [tutorial](https://www.tutorialspoint.com/pyspark/pyspark_quick_guide.htm)
* [classmethod](https://dev.classmethod.jp/etc/apache-spark_rdd_investigation/)