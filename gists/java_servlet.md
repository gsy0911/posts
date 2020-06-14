# はじめに

このページでは特定のプログラムの動作（呼び出し）について書きます。
通常のプログラムと異なり`WebApplication`の場合は呼び出されて（クライアントからのリクエストによって）プログラムは動きます。
そのため、書いたコードがどのように動いているかを想像することは難しいです。
そこで、今回は提供されている`WebApplication`の動きについて、特に初期動作について解説します。

***ちなみに間違っていることもあるので、詳しい動作を知りたい場合は本とかちゃんとしたページの解説をみるべきです***

ただし、この`JSP`と`Servlet`を使った開発方法は古い（＝過去の技術）らしいのでふんわりと動作のイメージだけわかっておけば十分だと思います。
（今後で会わないことを願いながら。）
プログラミングにおいて何よりも大事なのはイメージです。ここでいうイメージというのは、各クラス同士がどのようにして連携し動いているかということです。なので、上述のようにいきなり初学者が`Web`プログラミングというのは、全体の動きが見えないという点で非常に学習しずらいと思います。

まぁこのページも文章ばっかで、イメージが大事というなら図でも描いた方がいいですね。


# WebApplicationの動き

ここでは、実際に Web ページの URL を呼び出したときの挙動（なにがどう呼び出されて動いているのか）を説明します。

## URL[http://localhost:8080/WebApplicationAns/FrontServlet/*](http://localhost:8080/WebApplicationAns/FrontServlet/) を呼び出したとき

この意味は最初にアプリケーションをローカルのサーバで実行したときです。この場合の呼び出しは以下の手順ですすみ、最終的に`MainMenu.jsp`が呼び出されます。

1. /WebContent/web.xmlの`<welcome-file-list></welcom-file-list>`の中の`<welcome-file></welcome-file>`が呼び出されます。
1. 今回は`<welcome-file>FrontServlet</welcome-file>`になっているので、FrontServletが呼ばれます、
1. ここでいう「FrontServlet」とは「/ドキュメント記述子/サーブレットのマッピング」で`/FrontServlet/* -> FrontServlet`になっているのでFrontServlet.javaが呼ばれます。名前が同一なので分かりにくいですが、例えばURLが`http://localhost:8080/WebApplication/A`で、マッピングが`/A/* -> FrontServlet`でも同様のことが起き（ると思い）ます。
1. マッピング（`FrontServlet.java`のクラス定義の上にアノテーションで`@mapping`的なので`/FrontServlet/*`という記述でマッピングを行っています）で`FrontServlet.java`が参照されていることが分かったのでURL`http://localhost:8080/WebApplication/FrontServlet/`へのアクセス方法は（明示的ではないけど）`GETメソッド`（＝URLにパラメータを渡す方法）で呼ばれています。
1. なので`FrontServlet.java`の`doGet()`メソッドを見てみると`doPost()`が呼ばれていることが分かります。
1. `doPost()`では主に「次に表示（遷移）する画面」と「その画面へパラメータを渡す」ことを行います。
1. 「次に表示（遷移）する画面」は`String defaultPage = "MainMenu.jsp"`で定義を行っています。
1. 「その画面へパラメータを渡す」ところは`Command command = getCommand(cId)`にて行われます。ここでは`HttpServletRequest req`から渡された`cId`より動かす`xxxController`が選択されます（初期状態では`xxxCommand`しかないけど）。
1. ただ、今回のURL`http://localhost:8080/WebApplication/FrontServlet/`ではパラメータである`cId`が無いので、`getCommand(String)`における`if文`の`else`節が呼ばれ、特に何も行われません。そして`String nextPage = defaultPage`の記述があるように、最初は`/WebContent/MainMenu.jsp`の画面が表示されます。

このようにして`http://localhost:8080/WebApplication/FrontServlet/`
へのアクセスがあった場合には`MainMenu.jsp`の画面が表示されます。

## 画面遷移が行われた場合

次に画面の繊維の方法について簡単に説明する。例えば、`MainMenu.jsp`から`リンク「商品管理」`が押された場合を考えてみます。

1. `MainMenu.jsp`の`リンク「商品管理」`を見てみると`<a>タグ`が貼られていることが分かります。これは、アンカータグといって別の画面に遷移したり同じページの特定の場所に移動できる（？）htmlタグです。アンカータグの構成は次のようになっています`<a href="遷移先アドレス">「表示名」</a>`。
1. で、`リンク「商品管理」`の`href="遷移先アドレス"`を見てみるとなんか`<% %>`で囲まれた式があり、最後に`?<% cId %>=110`とか書かれてい（ると思い）ます。
1. この`<% %>`の中にある文字列は定数を表しており、`パッケージ「const」の「Consts.java」`とかに具体的な内容が書かれています。
1. `<a>`タグが押されると`http://localhost:8080/WebApplication/FrontServlet?cId=110`というリクエストがクライアントからサーバーへ送られます。このリクエストは`デプロイメント記述子/マッピング「/FrontServlet/* -> FrontServlet」`により`FrontServlet.java`が再び呼ばれます。ここで`/FrontServlet/*`の`*`は`「あらゆる文字列」`を意味しており例えば`http://localhost:8080/WebApplication/FrontServlet/aaaaaaaaa`みたいなURLにアクセスしてもマッピングにより`FrontServlet.java`が呼ばれ（ると思い）ます。
1. で、上記のURLアクセスは`GETメソッド`だが`FrontServlet.java`内では`doGet()`は`doPost()`を呼んでいる。なので最初の場合と同様に`doPost()`の処理が行われます。
1. ここで`getCommand(cId)`が活躍してくる。今回渡された`cId`はURLの`?`以降の文字列を見てみると`110`となっているので、`if文`中の`110`に対応する`command`が`getCommand(cId)`より取得され、戻り値として返されます。ちなみに`cId = commandId`の略語だと思います。
1. 取得した`command`オブジェクトは`execute(req)`メソッドを持っており、`command.execute(req)`で実行されます。この`execute(req)`内で「次の画面に渡すパラメータ」を`req.setAttribute(string, param)`の形で渡しています。`setAttribute()`は複数個ある場合とかない場合もある。そして「次に表示（遷移）する画面」は戻り値`xxx.jsp`が渡されます。
1. 戻り値として`xxx.jsp`を受け取った`doPost()`メソッドはそれを`nextPage = "xxx.jsp"`とし次の画面を表示します。

このようにして、リンクが押された場合の挙動が決まっています。
とりあえず、覚えておくべきは`「すべてのリンクはFrontServletを呼んでいてcIdによって"次に表示（遷移）する画面"と"その画面に渡すパラメータ"を変えている」`ということです。

# DAOとServiceとControllerについて

これらの関係・設計思想としては「呼び出す側は呼び出される側の内部仕様を知らなくても、期待する結果が返ってくる」ことである。なので、それぞれのパーツ（オブジェクト）が変更されても小さな変更で済みます。（的なことは教科書にも書いてあったけ？）

* DAOはデータアクセスだけを行う。今回はMySQLを使っているが別のデータベースに切り替えたい場合はここだけを変えればよい。
* ServiceはDAOを使ってデータの取得を行う。ここでDAOが何をしているかはserviceには関係なく期待する結果だけ返ってくればよい。
* ControllerはServiceなどを使ってデータの取得と画面遷移の役割を担っている。ControllerはServiceは見えるがDAOは見えず、serviceがどこからデータを持ってきているのかは知らなくてもよい。
