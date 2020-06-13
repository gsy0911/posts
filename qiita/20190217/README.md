# はじめに


`python`のdictでkeyを日本語に設定すると、keyerrorに直面しました。
見た目は同じなのに、`KeyError`ですと言われたときの解決方法です。

半ば強引ですがとりあえずこれでなんとかなります。

# 解決策

```
import unicodedata
KeyMatched_str = unicodedata.normalize('NKFC', KeyError_str)
```



# 原因

`dict`で`KeyError`になる原因は、日本語の濁点や半濁点にありました。
どういうことかというと、`utf-8`では「ど」などの濁点・半濁点のつく文字には以下の二通りの表現方法があるためです。

* 単純に「ど」
* 「と」＋「濁点」

パソコンで表示される上では、上の２つは全く同じ文字ですが`utf-8`などで表示すると違うことがわかります。
試しに以下のコードを実行してみます。

```
print(b'\xe3\x81\xa9'.decode('utf-8'))
print(b'\xe3\x81\xa8\xe3\x82\x99'.decode('utf-8'))
print(b'\xe3\x81\xa8'.decode('utf-8'))

--result--
ど
ど
と
```

このように、違うことがわかります。
`Python`の`dict`はこの違いも判別しているらしく、`KeyError`が発生してしまうことがあるようです。

# 参考
<sup><sub>ほぼ参考サイトのコピペです。。。</sub></sup>
* [文字と濁点・半濁点が分かれていて，それらを結合したい時](https://qiita.com/tma15/items/61c38fba5ca6c5f7bd77)

