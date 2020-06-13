
# はじめに

`matplotlib`で日本語を利用しようとすると、「豆腐」が表示されてうまく行かないことがあった。

## フォントリストを探す

手始めに以下のコードを実行して、`matplotlib`のフォントリストがどこにあるか見つけます。

```
import matplotlib as mpl
mpl.get_cachedir()
```

`ubuntu`の場合は`/home/[user_name]/.cache/matplotlib/fontlist-v300.json`とかになっていると思うので適当なエディタで開きます。

この中から、適当な`Gothic`フォントを指定すれば、日本語表記自体はされます。
今回は`Ubuntu`だったので`TakaoPGothic`というものを利用しました。

## 設定方法

```
import matplotlib.pyplot as plt
plt.rcParams['font.family'] = 'TakaoPGothic'
```


また、自分でダウンロードしてきたフォントを利用したい場合は、[ここ](https://qiita.com/hatunina/items/a77128c7f50b19ad2c51)とかを参考に追加してみてください。
