# はじめに


本記事では，QiitaAPIを用いてQiitaに投稿された記事データを取得して，
Qiitaの記事データに現れたタグの数を集計し，記事データのタグ情報の共起関係を見てみたいと思います．
それによって，自分が知らない領域について知見を得るきっかけになればいいかなと思っています．

## 共起関係とは？

共起関係とは

> ある単語がある文章（または文）中に出たとき、その文章（文）中に別の限られた単語が頻繁に出現すること。
(wikipediaより)

です．
今回のケースで言うと，ある記事データに設定されたタグ情報同士が，ほかの記事データでも頻繁に出現するかどうかです．
タグ情報同士がある記事データに設定されたことを（タグ情報に対して）共起関係といいます．
この共起関係が多ければ多いほどタグ情報同士の結びつきが強く，逆に少なければ結びつきは弱くなります．

詳細は[こちらの記事](https://qiita.com/tellusium/items/91ada7c868e4f0a140af)をご覧ください．

# 環境設定

今回このプログラムを動かす環境などは以下の通りです．

* Ubuntu on GCP
* python 3.7.1 on pipenv
* Postgresql

## GCPの環境設定

まず，今回はGCPのVMインスタンス上でプログラムを書いているのですが，
GCPの設定については[こちらの記事](https://qiita.com/gremito/items/d53560e3cd4a3b1acbe8)が分かりやすいと思うので，参考にしてください．

## pipenvの設定

pythonを実行する環境として`pipenv`の設定を行います．
`pipenv`とは***Python公式が正式に推薦するPythonパッケージングツール***でpythonのバージョン管理を行う`venv`と，
パッケージ管理を行う`pip`が連動し，ユーザにとってパッケージ管理を行いやすいツールです．
これを利用することで，各フォルダごとに固有の`python環境`を作ることができ，
パッケージの管理もフォルダごとに行えるようになります．
以下の`pipenv`の導入手順は，[このサイト](http://pynote.hatenablog.com/entry/python_ubuntu_pipenv)に則っています

まずは，開いた環境のパッケージなどを最新化します．

```
$ sudo apt update
$ sudo apt upgrade -y
```


次に，pyenvの依存ライブラリをインストールします．

```
$ sudo apt install -y --no-install-recommends \
    build-essential \
    curl \
    libbz2-dev \
    libffi-dev \
    liblzma-dev \
    libncurses5-dev \
    libncursesw5-dev \
    libreadline-dev \
    libsqlite3-dev \
    libssl-dev \
    llvm \
    tk-dev \
    wget \
    xz-utils \
    zlib1g-dev \
    ca-certificates \
    git \
    python-dev
```

そうしたら，`pyenv`を`GitHub`より`clone`します．

```
$ git clone https://github.com/pyenv/pyenv.git ./pyenv
```

つぎに`~/.bashrc`ファイルに以下の内容を追記します．
\[your_name\]の箇所は適宜変更しましょう．

```rc:~/.bashrc
# ~~前略~~

export PYENV_ROOT="/home/[your_name]/pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

`clone`した`pyenv`をインストールします．

```
$ source ~/.bashrc
$ pyenv -v
pyenv 1.2.8-12-g775a4b63
```

そうしたら，`pyenv`を使って`python 3.7.1`をインストールします．

```
$ pyenv install 3.7.1
$ pyenv global 3.7.1
```

`pyenv`を使ってpythonをインストールしたら，ようやく`pipenv`をインストールします．

```
~ $ pip install pipenv
~ $ echo 'eval "$(pipenv --completion)"' >> ~/.bashrc

~ $ pipenv --version
> pipenv, version 2018.11.26
```

インストールしたら，これから作業を行うフォルダを作成し，適当な`test.py`を用意しきちんと動くか試します．
中身は`print('hello, world!')`とかで十分です．

```
~ $ mkdir qiita
~/qiita $ pipenv install

~/qiita $ pipenv run python test.py
> hello, world!
```

動いたら，一旦`pipenv`の設定は終わりです．

## データベースの設定

データをためておく場所として今回はデータベースの`Postgresql`を利用します．
csvなどで保持してもいいのですが，保存する記事データの数と今後の利用を考えると，
データベースを利用した方がいいのでそうします．
（実際は，RDBではなくて，GCP上のNoSQLを使った方がいいのだけれど，それは今後の課題．．．）

まず，インストールと初期設定をします．
最初に`postgres`というユーザに対してパスワードを設定し，
`qiita`という名前のデータベースを作成します．


```
~/$ sudo apt install postgresql postgresql-contrib -y

~/$ sudo -u postgres psql postgres
postgres=# \password
Enter new password: 
Enter it again:

postgres=# create database qiita;
postgres=# \q
```

次に，今回は6種類のテーブルを作成します．
これらのテーブルを作成するにあたっては，[この記事](https://qiita.com/nkmk/items/fbac13cd05b80334eb2b)と[この記事](https://qiita.com/pocket_kyoto/items/64a5ae16f02023df883e)を参考にしました．
（使わないテーブルも載せているのは，今後利用するからです．）

| テーブル名 | 用途 |
|:--:|:--:|
| user_list | ユーザのpermanent_idとidを管理するテーブル |
| user_info | ユーザのgithubとかtwitterとかの情報を管理するテーブル．（今回は使わない） |
| user_list_used2search | ユーザのfollowerとかfolloweeの情報から新たなユーザを取得するためのテーブル．（今回は使わない） |
| item_list | 記事データを管理するテーブル ．|
| tag_appear_count | タグの出現回数を記録するテーブル． |
| value_list | 記事データを取得する際に利用する（記事の取得期間を保持するための）補助テーブル ．|

これらの6つのテーブルを作成していきます．
コマンドは以下の通り．
（`primary key`も一度に設定できますが，一回一回`alter table`してます．）

```
~/$ psql -h localhost -U postgres -d qiita -W

qiita=# create table user_list (permanent_id integer, id text, followers_count integer, followees_count integer);
qiita=# alter table user_list add primary key(permanent_id);
qiita=# create table user_info (permanent_id integer, description text, facebook_id text, github_login_id text, items_coun
t integer, linkedin_id text, twitter_screen_name text);
qiita=# alter table user_info add primary key(permanent_id);
qiita=# create table user_list_used2search (permanent_id integer, id text, search_count integer, prcs_status integer);
qiita=# alter table user_list_used2search add primary key(permanent_id);
qiita=# create table item_list (item_id text, permanent_id integer, title text, body text, created_at text, updated_at text, likes_count integer, tags_str text);
qiita=# alter table item_list add primary key(item_id, permanent_id);
qiita=# create table value_list (key text, value text);
qiita=# alter table value_list add primary key(key);
qiita=# create table tag_appear_count (tag_name text, calc_date text, period text, count integer);
qiita=# alter table tag_appear_count add primary key(tag_name, calc_date);

qiita=# \d
                 List of relations
 Schema |         Name          | Type  |  Owner   
--------+-----------------------+-------+----------
 public | item_list             | table | postgres
 public | tag_appear_count      | table | postgres
 public | user_info             | table | postgres
 public | user_list             | table | postgres
 public | user_list_used2search | table | postgres
 public | value_list            | table | postgres
(6 rows)

qiita=# \q

```


上記のように表示されたらとりあえずテーブルの設定は完了です．

## Qiitaの記事データ取得

さて，ここから`python`を使って実際に記事データを取得していきます．
その前に，データ取得に必要なライブラリをインストールします．

```
~/qiita $ pipenv install requests pandas psycopg2
```

### 補助関数群の定義

まずは，記事のデータを本格的に取得する前にQiitaAPIから取得できるjsonをデータベースに入れる補助関数群`funcs.py`を定義していきます．
最初に，`funcs.py`のimportは以下の通りです．

```~/qiita/funcs.py
import http.client
import datetime
import urllib.request
import json

import psycopg2
import http.client
import datetime
import urllib.request
import json
import requests
import time

import sys
```

この`funcs.py`を定義して，
別のファイルから簡単に呼ぶことができます．

```other_file.py
import sys
sys.path.append("/home/[your_name]/qiita")
import funcs

# 以下のようにして呼べます．
funcs.[function_name()]
```

まずは，ユーザの情報を取得する関数を定義します．
先ほどのimport群の下に定義していきます．
この関数は，QiitaAPIで取得したユーザをデータベースに登録する関数です．
それぞれの引数`psgr`，`cur`は，postgresqlへの接続情報，`json`，`jsonstr`はユーザの情報のjsonです．

```~/qiita/funcs.py
# import_s

def insert_single_user(psgr, cur, jsonstr):
	permanent_id = jsonstr['permanent_id']
	id = jsonstr['id']
	followers_count = jsonstr['followers_count']
	followees_count = jsonstr['followees_count']
	description = "null" if jsonstr['description'] == None else jsonstr['description']
	facebook_id = "null" if jsonstr['facebook_id'] == None else jsonstr['facebook_id']
	github_login_id = "null" if jsonstr['github_login_name'] == None else jsonstr['github_login_name']
	items_count = jsonstr['items_count']
	linkedin_id = "null" if jsonstr['linkedin_id'] == None else jsonstr['linkedin_id']
	twitter_screen_name = "null" if jsonstr['twitter_screen_name'] == None else jsonstr['twitter_screen_name']
	try:
		cur.execute("insert into user_info (permanent_id, description, facebook_id, github_login_id, items_count, linkedin_id, twitter_screen_name) values (%s, %s, %s, %s, %s, %s, %s)", (permanent_id, description, facebook_id, github_login_id, items_count, linkedin_id, twitter_screen_name))
		cur.execute("insert into user_list (permanent_id, id, followers_count, followees_count) values (%s, %s, %s, %s)", (permanent_id, id, followers_count, followees_count))
		psgr.commit()
	except Exception as e:
		psgr.commit()

def insert_user(psgr, cur, json):
	for jsonstr in json:
        insert_single_user(jsonstr)
```

次に記事データをDBに挿入する関数を定義します．
それぞれの引数は，`inser_user()`と同じです．
Qiitaでは1つの記事に対してタグを5つ設定できるのですが，データベースに入れるときにはそれを1つの文字列として結合し，挿入しています．

```~/qiita/funcs.py
# import_s
# insert_user()_s

def insert_item(psgr, cur, json):
	for jsonstr in json:
		permanent_id = jsonstr['user']['permanent_id']
		item_id = jsonstr['id']
		title = jsonstr['title']
		body = jsonstr['rendered_body']
		created_at = jsonstr['created_at']
		updated_at = jsonstr['updated_at']
		likes_count = jsonstr['likes_count']

		#concatenate
		tag_list = []
		for t in jsonstr['tags']:
			tag_list.append(t['name'])
		tags_str = ",".join(tag_list)
		try:
			cur.execute("insert into item_list (item_id, permanent_id, title, body, created_at, updated_at, likes_count, tags_str) values (%s, %s, %s, %s, %s, %s, %s, %s)", (item_id, permanent_id, title, body, created_at, updated_at, likes_count, tags_str))
			psgr.commit()
		except Exception as e:
			psgr.commit()
		insert_single_user(psgr, cur, jsonstr['user'])
```

最後に，データベースとはあまり関係ないですが，
ソースコードを簡単にするための補助関数を定義します．
`my_header()`はQiitaAPIを利用するときに1時間あたりに1000回APIを叩けるようにするための情報を取得する関数です．
`get_connection()`は，Postgresqlの情報をDataFrameに読み込む際に必要な情報をまとめた関数です．
後々に複数回呼ぶことになるので，ここにまとめておきます．
[your_code]と書かれた部分の取得方法については[この記事](https://qiita.com/pocket_kyoto/items/64a5ae16f02023df883e)を参考に取得してください．

```~/qiita/funcs.py
# import_s
# insert_user()_s
# isert_item()

def my_header():
	"""
	# get qiita header with specific header
	"""
	h = {
	        "Authorization": "Bearer [your_code]",
        	"content-type" : "application/json"
	}
	return h

def get_connection():
	# connection info
	connection_config = {
	 	'host': 'localhost',
		'database': 'qiita',
		'user': 'postgres',
		'password': '***'
	}
	return psycopg2.connect(**connection_config)


```

さて，これで記事データを取得する準備はできました．


### 記事データの取得

それでは，次に記事データの取得を行っていきます．

まずは，importと事前準備の部分です．
QiitaAPIでは普通に取得しようとすると，今日の日付から10000件過去のデータしか取得することができません．
そこで，取得する期間を絞って取得していきます．
変数の`start`と`end`に関しては，後々にデータベースから値を取得して都度変更していきます．

```~/qiita/get_new_item_backwards.py
import sys
sys.path.append("/home/[your_name]/qiita")
import funcs

import psycopg2
import time
import os
import requests
import sys
import math
import datetime

url = 'https://qiita.com/api/v2/items'

start = "2018-10-20"
end = "2018-11-1"

# in order not to over the limitation accessing 1000 times in one hour
sleep_sec = 3.6
```

さて，以下が指定された期間の記事データを取得するコードです．
最初に`with`句でデータベースの接続を行って，記事データの取得とデータベースへの挿入を行っています．
1回実行するたびに，データベース`value_list`に保存された前回実行日を取得し，
取得した日から5日間の間に投稿された記事データをデータベースに登録します．

QiitaAPIでは，1回のrequestで取得できる記事の件数が100件しかないため，
responseのヘッダの`Total-Count`より記事の総数を取得し，
すべての記事データを取得できるようにループを回します．

なお，取得した記事データには，ユーザの情報も付属しているため，
ユーザの情報も同時にデータベースへ登録を行います（登録は，`funcs.insert_item()`内でしています）．

最後に，取得した日をデータベースに登録するまでが一連のプログラムの流れです．

```~/qiita/get_new_item_backwards.py
# import_s

with psycopg2.connect("host=localhost dbname=qiita  user=postgres password=***") as psgr:
	with psgr.cursor() as cur:
		# get start day
		cur.execute("select * from value_list where key = 'get_item_start'")
		row = cur.fetchall()
		for r in row:
			end = r[1]

		new_end_day = datetime.datetime.strptime(end, "%Y-%m-%d")
		new_start_day = new_end_day - datetime.timedelta(days=5)

		start = str(new_start_day.year) + "-" + str(new_start_day.month) + "-" + str(new_start_day.day)
		end = str(new_end_day.year) + "-" + str(new_end_day.month) + "-" + str(new_end_day.day)

		p = {
			'per_page': 100,
			'page':1,
			'query': 'created:>{} created:<{}'.format(start, end)
		}

		r = requests.get(url, params=p, headers=funcs.my_header())
		# item count in the whole day
		total_count = int(r.headers['Total-Count'])
		# to get all items
		loop_count = math.ceil((total_count - 100)/100)

		funcs.insert_item(psgr, cur, r.json())
		for i in range(loop_count):
            # for access limitation
			time.sleep(sleep_sec)  
			print("loop : "+str(i+2))
			p['page'] = i + 2
			r = requests.get(url, params=p, headers=funcs.my_header())
			print("remaining : " + str(r.headers['Rate-Remaining']))
			funcs.insert_item(psgr, cur, r.json())

		cur.execute("update value_list set value = %s where key = %s ", (start, 'get_item_start'))
		cur.execute("update value_list set value = %s where key = %s ", (end, 'get_item_end'))
		psgr.commit()
```

ここまで，来たら`cron`で定期実行できるようにします．
ただ，`pipenv`環境のため，`shellscript`を書いて簡単に実行できるようにします．

```~/qiita/sh_get_new_item_backwards.sh
cd /home/[your_name]/qiita
/home/[your_name]/pyenv/shims/pipenv run python get_new_item_backwards.py
```

`shellscript`を書いたら，`cron`に登録します．
今回は10分間隔での実行にしましたが，もう少し早くデータを取得し終えたい場合は5分間隔でもいいかなと思います．
ちなみに，10分間隔だと，1日でおよそ2年分の記事データを取得でき，
Qiitaのサービスが開始した2011年からの記事データをおよそ4日で取得できます．

```crontab
*/10 * * * * sh /home/[your_name]/qiita/sh_get_new_item_backwards.sh
```


## タグの共起関係の取得

ある程度，データがたまったら，タグの出現回数の計算とタグの共起関係を取得していきます．
まずは，タグの出現回数のプログラムから．
このプログラムは，取得した記事データの全期間のタグ情報を取得し，出現回数をカウントしています．
（実際の可視化の際には，期間を指定した方が面白くなりそうではあるので，今後実装していきたいです．）

内容としては，データベースに登録されたタグ情報はカンマ(`,`)区切りで保存したので，
それを`split`して1つ1つの要素にした後，
`pandas`の`value_counnts()`関数を呼ぶだけで，同じ文字列の出現回数を計算しています．（`pandas`が優秀すぎる）
あとは，1行ごとにデータベースへ登録を行っているだけです．

```~/qiita/insert_tag_count.py
import sys
sys.path.append("/home/[your_name]/qiita")
import funcs

import pandas as pd
import psycopg2
import sys
import datetime

# datetime
today = datetime.datetime.now()
date = str(today.year) + "-" + str(today.month) + "-" + str(today.day)

# https://stackoverflow.com/questions/37224002/split-pandas-series-into-dataframe-by-delimiter
item_df = pd.read_sql(sql="select permanent_id,tags_str from item_list", con=funcs.get_connection())

tag_df = item_df['tags_str'].str.split(',')
tmp = []
for tag in tag_df:
	tmp.append([e.strip() for e in tag])

tag_df = pd.DataFrame(tmp)

tag_all = pd.concat([tag_df[0],tag_df[1],tag_df[2],tag_df[3],tag_df[4]]).dropna()
tag_df = tag_all.value_counts()
with psycopg2.connect("host=localhost dbname=qiita  user=postgres password=***") as psgr:
	with psgr.cursor() as cur:
		for i,t in tag_df.iteritems():
			cur.execute("insert into tag_appear_count (tag_name, calc_date, period, count) values (%s, %s, %s, %s)", (i, date, 'whole', str(t)))
```

さて，最後のプログラムはタグ同士の共起関係を取得，計算するプログラムです．
その前に，import群と補助関数を定義します．
この`get_related_tags_with_search_word()`は，
記事データのうちタグに引数`search`を含む記事のタグを取得する関数です．
返り値は，`search`ワードと同じ記事中に書かれたタグと，そのタグの出現回数を返します．
今回は，出現回数が200回以上のタグのみに絞りました．
また，`search`を含むタグはヒットさせたくないので，正規表現で検索結果から除外するようにします．

```~/qiita/calc_tag_relationship.py
import sys
sys.path.append("/home/[your_name]/qiita")
import funcs

import pandas as pd
import psycopg2
import sys
import datetime

import re

def get_related_tags_with_search_word(search='python'):
	# compile
	try:
		pattern = r'%s' % (search)
		repattern = re.compile(pattern)
	except Exception as e:
		return pd.DataFrame()

	# get DataFrame
	item_df = pd.read_sql(sql="select tags_str from item_list where tags_str like %s", params=['%{}%'.format(search)],con=funcs.get_connection())

	# split tags_str
	tag_df = item_df['tags_str'].str.split(',')
	tmp = []
	for tag in tag_df:
		tmp.append([e.strip() for e in tag])

	tag_df = pd.DataFrame(tmp)
	tag_all = pd.concat([tag_df[0],tag_df[1],tag_df[2],tag_df[3],tag_df[4]]).dropna()

	# get tags_str and count
	df = pd.DataFrame()
	for i,t in tag_all.value_counts().iteritems():
		if not repattern.match(i) and t > 200:
			df = pd.concat([df, pd.DataFrame({'count':t}, index=[i])])
	if df.empty:
		return pd.DataFrame(columns=[search])
	df.columns=[search]
	return df
```

さて，補助関数を定義したので，共起関係を取得する部分を書いていきます．
ただ，補助関数のおかげで`search`に共起関係を探したいキーワードを入れるだけです．


```~/qiita/calc_tag_relationship.py
# import_s
# get_related_tags_with_search_word()

p_df = get_related_tags_with_search_word(search='Python')
print(p_df)
```

以下が`search=Python`とした実行結果です．
やはりPythonでは，`DeepLearning`や`機械学習`と言ったキーワードと共起関係にあるようです．

```:result
                 Python
機械学習               1617
DeepLearning        999
Django              980
TensorFlow          738
pandas              724
numpy               608
matplotlib          570
MachineLearning     540
Jupyter             528
OpenCV              503
AWS                 493
Keras               483
RaspberryPi         420
Anaconda            400
Flask               363
scikit-learn        360
初心者                 343
自然言語処理              335
docker              323
Chainer             315
Mac                 298
python2.7           290
lambda              286
Ruby                254
数学                  249
pyenv               247
pip                 238
Twitter             231
データ分析               225
Windows             225
Selenium            222
JavaScript          219
スクレイピング             210
画像処理                205

```

以下が`search=JavaScript`とした実行結果です．
`Node.js`や`vue.js`といったフレームワークと共起している関係が見て取れます．
また，私は`gulp`というキーワードをこの共起関係を使って知ることができました．

```:result
                  JavaScript
Node.js                 1648
jQuery                  1340
HTML                    1068
vue.js                   826
HTML5                    818
CSS                      716
React                    694
reactjs                  615
es6                      526
TypeScript               507
webpack                  399
AngularJS                390
PHP                      338
Chrome                   290
Ruby                     279
babel                    270
npm                      246
redux                    240
CoffeeScript             234
Rails                    228
GoogleAppsScript         217
es2015                   207
Electron                 204
gulp                     204
```



# おわりに


今回は記事データの取得と，タグの共起関係を計算するだけに終わってしまいましたが，
本番はこれを可視化するところなので，今後はD3.jsを使っていい感じに可視化したいと思っています．

Qiitaの記事データを眺めるだけでも面白いので，みなさんもぜひやってみてください～
