# 手法
Vector AutoRegression（ベクトル自己回帰）を用いて複数時系列のデータを用いて将来のデータの予測などを行う？

# 環境
* Windows 10
* Anaconda3
* Python 3.5.2

Anacondaのインストールおよび，仮想環境の構築は[ここ](https://gist.github.com/gsy0911/03a1b58dc85bb218ff84f39e2e032c29)を参照のこと。


# pyfluxのインストール
[公式サイト](http://www.pyflux.com/installation/) によると，

```prompt:cmd
> pip install pyflux
```

と入力することで PyFlux 0.3.2(2017年8月4日現在)がインストールされらしいがどうやらうまくいかない。。。

エラーを見てみるとscipyというパッケージのインストールに失敗しているみたい。
なのでscipyを別にinstallする。
[anaconda公式サイト](https://anaconda.org/anaconda/scipy)より，以下のコマンドでscipyをinstallした。

```cmd:cmd
> conda install -c anaconda scipy
```

すると簡単にscipyのinstallは完了し，再度pyfluxを上記のコマンドでinstallしたら，あっさりとできた。
一応pythonの対話モードで以下のように確認したら，エラーなくできた。

```cmd:cmd
> python
>>> import pyflux as pf
```

# その他のパッケージのインストール
続いて，pandasとmatplotlibをインストールする

```cmd:cmd
> conda install -c anaconda pandas
> conda install -c conda-forge matplotlib
```

そして[github](https://github.com/pydata/pandas-datareader) よりpandasのデータを取得する

```cmd:cmd
> pip install pandas-datareader
```

最後にstatsmodelsのインストール

```cmd:cmd
> conda install -c statsmodels statsmodels
```

このstatsmodelsは～に利用する。

# プログラムの実行
とりあえず，pythonで以下のコードを書いてみて，yahooの株価を取得してみる。
[公式サイト](http://www.pyflux.com/vector-autoregression/) のVARを参考に作る。
ただ，このサイトのは若干古くpandasのデータセット？の部分がうまく動かなかったので
[pandas](https://pandas-datareader.readthedocs.io/en/latest/remote_data.html) を参考にした。

```python:stock.py
import numpy as np
import pyflux as pf
from pandas_datareader import data as web
import datetime
import matplotlib.pyplot as plt

start = datetime.datetime(2010,1,1)
end = datetime.datetime(2013,1,27)
f=web.DataReader("F", 'yahoo', start, end)
# print(f)
print(f.ix['2010-01-04'])
```

実行結果は以下の通り

```result:res
Open         1.017000e+01
High         1.028000e+01
Low          1.005000e+01
Close        1.028000e+01
Adj Close    8.201456e+00
Volume       6.085580e+07
Name: 2010-01-04 00:00:00, dtype: float64
```

次に時系列データとしてyahooの株価を表示してみた。

```python:stock.py
import numpy as np
import pyflux as pf
from pandas_datareader import data as web
import datetime
import matplotlib.pyplot as plt

start = datetime.datetime(2013, 1, 1)
end = datetime.datetime(2017, 1, 27)
f=web.DataReader("F", 'yahoo', start, end)
opening_prices = np.log(f['Open'])
plt.figure(figsize=(15,5));
plt.plot(opening_prices.index, opening_prices);
plt.title("Logged opening price");
plt.show()
```

# 参考にしたサイト
* [http://qiita.com/HirofumiYashima/items/36e2dce63026842a4020](http://qiita.com/HirofumiYashima/items/36e2dce63026842a4020)
* [http://qiita.com/HirofumiYashima/items/8d937ebaec8dfe265147](http://qiita.com/HirofumiYashima/items/8d937ebaec8dfe265147)
* [http://logics-of-blue.com/var%E3%83%A2%E3%83%87%E3%83%AB/](http://logics-of-blue.com/var%E3%83%A2%E3%83%87%E3%83%AB/)
* [PyConJP 2016: pandasでの時系列処理についてお話させていただきました](http://sinhrks.hatenablog.com/category/%E5%89%8D%E5%87%A6%E7%90%86)
* [Matplotlibで画像を表示](http://qiita.com/zaburo/items/5637b424c655b136527a)

* [Pythonの覚え書き(Scikit-learn, statsmodels編)](http://hirotaka-hachiya.hatenablog.com/entry/2014/09/21/100707)
* [SVARモデルでBitCoin予測](http://ki-chi.jp/?p=688)
* [時系列分析II―ARMAモデル（自己回帰移動平均モデル）の評価と将来予測](http://www.atmarkit.co.jp/ait/articles/1409/01/news006.html)
