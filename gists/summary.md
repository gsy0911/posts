# このページについて

ここは以下のことを共有するページです。
* Eclipse での WebApp 開発に関すること
* 講師からいただいた有益な情報
* その他



# 開発演習

開発演習での気づきとかを共有します。

## サーバー`http://localhost:8080/[ProjectName]/`にアクセスしたときに任意のページを表示させる

今現在`http://localhost:8080/[ProjectName]/init`にアクセスするか、`.jps` ファイルを直接実行して起動しているはずです。
そこで、初期ページは`http://localhost:8080/[ProjectName]/` にしたい、という場合はこの方法をするとよいです。

そもそも、なぜ初期ページ`http://localhost:8080/[ProjectName]/`にうまくアクセスできないかというと、あるファイルにそういった記述がないからです。
そのあるファイルとは`/WebContent/WEB-INF/web.xml`です。
このファイルの上部には、`http://localhost:8080/[ProjectName]/`へアクセスがあった場合に表示するページのリストが以下のように記述されています。

```
<welcom-file-list>
  <welcom-file> *** </welcom-file>
  <welcom-file> *** </welcom-file>
  <welcom-file> *** </welcom-file>
</welcom-file-list>
```

おそらく、初期設定の場合は６個のファイル（index.html など）があるはずです。
これらのファイルは、上から順番に存在が確認され、存在すれば読み込まれてレスポンスとして返されるはずです。
なので、任意のページを表示したい場合は、記述の一番上に追記すればよいです。
例えば、`/init` ページを最初に表示したい場合は以下の方法で行けるはずです。

```
<welcom-file-list>
  <welcom-file> init </welcom-file>
  <welcom-file> *** </welcom-file>
  <welcom-file> *** </welcom-file>
  <welcom-file> *** </welcom-file>
</welcom-file-list>
```


ただ、実際はこの`/init`のマッピングは`/init -> init` であることが必須です。

## \[ポート番号が既に使われています\]的なエラーメッセージが表示される。

以下の手順で解決できます。
1. Eclipse の「サーバー」タブを選択
1. 作成した TomCat サーバーをダブルクリック
1. サーバーの詳細設定画面が上部に表示される
1. 表示された画面の右側にあるポート番号をすべて１ずつ加算する
1. 例えば`8080`は`8081`へ。

## コメントアウト

コメントアウトとは、コメントを書いたり、プログラムを消したりせず実行しないようにする記述方法のことです。
以下の方法で用いられます。
```
// コメント一行

/*
コメントアウト複数行
コメントアウト複数行
*/
```

んで、Eclipse では、コメントアウトが`Ctrl + /`のコマンドで簡単にある行の先頭に`//`付与できます。
また、複数行にわたって`//`を付与できます。
コメントがある行で再度`Ctrl + /`を押すとコメントアウトを外すことができます、

# 講師「知っておくべき技術用語など」

講師からいただいたキーワードとそれに関する説明と参考にしたリンクを1つ貼っています。
説明は、間違っているかもしれないですし、そもそも説明不足であると思います（まとめサイト自体情報が絞られているうえに、それを要約するという「上澄み液をろ過した」ような文章なので）。
そこら付近を考慮したうえで、ご覧ください。
分類は、主に講師が書いた順番に準拠しています。
誤字脱字、内容に間違いがあった場合は連絡してください。


## プログラミング補助

| キーワード | 説明 | リンク |
|:--:|:--:|:--:|
| maven | Java用プロジェクト管理ツール。XMLで記述されたライブラリ名とバージョンを指定することで、外部サイトで集中管理されているJARを自動ダウンロードし、ローカルでビルドする。 | [Maven with Eclipse](https://qiita.com/tarosa0001/items/e5667cfa857529900216) |
| Jenkins | プログラミングのテストを補助するツール。定期的にテストを自動実行してくれる(cronのような？)ソフト。 | [Jenkins"だけ"](https://qiita.com/Higemal/items/bb9a0c12b8fca0dd6657), [Jenkins + ](https://ics.media/tutorial-jenkins) |
| git | プログラムのソースコードのバージョンを管理するシステム。 | [よくわかる](https://www.sejuku.net/blog/5756) |

## テストツール

| キーワード | 説明 | リンク |
|:--:|:--:|:--:|
| JUnit | ユニットテスト（単体テスト）の自動化を行うためのフレームワーク（プログラム中に処理したメソッドの戻り値や、引き渡したパラメータに対してチェックを行う） | [使い方](https://qiita.com/takehiro224/items/a5d4265c4a1b36b0919c) |
| DBUnit | DBに更新した値に対してチェックを行うことができる。 | [使い方](https://qiita.com/tarosa0001/items/70a1efa9edac2d83ba1a) |
| Selenium | ブラウザを自動で操作するツール | [Selenium + Eclipse](https://qiita.com/tsukakei/items/41bc7f3827407f8f37e8) |
| JMeter | 負荷検証ツール。 サーバに対して指定した量のリクエストを送り、そのレスポンスを受けることで、パフォーマンス計測することができる。 | [使い方](http://tech-blog.rakus.co.jp/entry/2017/08/24/111332) |
| FindBugs | 実行時エラーとなりそうなところを見つけてくれるツール？ | [使い方](https://qiita.com/opengl-8080/items/796a4b534bc8104aebc3) |
| CheckStyle | Java のソースコードがコーディング規約に即しているかどうか判定するための静的解析ツール | [使い方](https://qiita.com/opengl-8080/items/cb4122a19269e8e683a4) |

## データベース補助
| キーワード | 説明 | リンク |
|:--:|:--:|:--:|
| O/Rマッピング | データベースとオブジェクト指向プログラミング言語の間の非互換なデータを変換するプログラミング技法のこと。 | [O/RMとは](http://www.atmarkit.co.jp/ait/articles/0404/13/news075.html) |
| Hibernate | JavaとRDBとのO/RMを（XMLで）実現するライブラリ。SQL文を自動生成する。 | [使い方](http://www.techscore.com/tech/Java/Others/Hibernate/index/) |
| MyBatis | JavaとRDBとのO/RMを（XMLで）実現するライブラリ。長期的な運用を考えるとHibernateより保守性で優れている。 | [Hibernateとの比較](http://bbook.hatenablog.jp/entry/2015/02/15/183946) |
| JPA | JavaとRDBとのO/RMを（主にアノテーションで）実現するライブラリ。アノテーションで記述されるため、entityとRDBの関係がわかりやすい。XMLの場合は離れて記述されてしまう。 | [JPAについて](https://qiita.com/opengl-8080/items/e4840aa3e33b42ae0d6b) |
| JTA | RDBとのトランザクションに関するライブラリ | - |


## Webフレームワーク

| キーワード | 説明 | リンク |
|:--:|:--:|:--:|
| Spring Framework | WEBアプリ作成の支援ツール | [Springについて](https://www.sejuku.net/blog/10456) |
| Spring Core | Spring の根幹をなすモジュールであり、オブジェクトの生成・登録、オブジェクト間の関連づけを行う Bean コンテナの機能を提供する。 | [Spring core/mvc](http://www.techscore.com/tech/Java/Others/Spring/1/) |
| Spring MVC | Model-View-Controllerフレームワークを提供する | [同上](http://www.techscore.com/tech/Java/Others/Spring/1/) |
| JAX-RS | (Java API for RESTful Web Services) | [Servletとの比較スライド](https://backpaper0.github.io/ghosts/jaxrs-getting-started-and-practice.html#24) |
| DI | (Dependency Injection)プログラムの依存関係（関数の呼び出しやインスタンスの生成など）を直接記述せずに実行時に動的に記述するようにする方法。 | [DIとは](https://qiita.com/hshimo/items/1136087e1c6e5c5b0d9f) |
| AOP | (Aspect Oriented Programming；アスペクト指向) オブジェクト指向の弱点を補うプログラミング方法。 | [AOPとは](http://e-words.jp/w/AOP.html) |
| RESTful | (REpresentational State Transfer) 分散型システムにおける複数のソフトウェアを連携させるのに適した設計原則の集合、考え方のこと | [RESTfulとは](https://qiita.com/NagaokaKenichi/items/0647c30ef596cedf4bf2) |


## その他

| キーワード | 説明 | リンク |
|:--:|:--:|:--:|
| HTML5 | Webページの構造を記述する言語。HTML5は2014年に策定された。拡張子は.html | [wikipedia](https://ja.wikipedia.org/wiki/HyperText_Markup_Language) |
| CSS3 | Webページをどのように装飾するか指示する文書。拡張子は.css | [wikipedia](https://ja.wikipedia.org/wiki/Cascading_Style_Sheets) |
| Javascript | 主にクライアント側のウェブページを動的に動かすために用いられる。動的型付けに対応しており、Javaとは全く異なる言語で拡張子は.js。 | [wikipedia](https://ja.wikipedia.org/wiki/JavaScript) |
| typescript | Javascriptに変換可能であり、静的型付けにも対応している。 | [typescriptについて](https://www.buildinsider.net/web/pronamatypescript/01) |
| Ajax | ウェブブラウザ内で非同期通信を行いながらインターフェイスの構築を行うプログラミング手法のこと。 | [wikipedia](https://ja.wikipedia.org/wiki/Ajax) |
| DOM | (Document Object Model)xmlやhtmlの各要素、たとえば<p>とか<img>などの要素にアクセスする仕組みのこと | [wikipedia](https://ja.wikipedia.org/wiki/Document_Object_Model) |
| XML | (Extensible Markup Language)拡張可能なマークアップ言語。<TAG></TAG>で記述される。拡張子は.xml | [wikipedia](https://ja.wikipedia.org/wiki/Extensible_Markup_Language) |
| json | (Javascript Object Notation)様々なソフトウェアやプログラミング言語間におけるデータの受け渡しの際に用いられる。 | [wikipedia](https://ja.wikipedia.org/wiki/JavaScript_Object_Notation) |
| Angular JS/Angular | Googleが開発しているJavascriptで書かれたフロントエンドWebアプリケーションのフレームワーク | [wikipedia](https://ja.wikipedia.org/wiki/AngularJS) |
| REACT | Facebookが開発しているJavascriptで書かれたフロントエンドWebアプリケーションのフレームワーク | [REACTとは](https://qiita.com/naruto/items/fdb61bc743395f8d8faf) |
| Vue.js | 利用者が作成するプロジェクトの規模に合わせて、柔軟にプログラミングの方法を切り替えることができる？ | [Vueについて](https://blog.nagisa-inc.jp/archives/2980) |
| Node.js | サーバーサイドのJavascript。 | [Node.jsとは](https://qiita.com/hshimo/items/1ecb7ed1b567aacbe559) |
| Bootstrap | レスポンシブデザイン（スマホやPCの画面サイズに合わせてwebページレイアウトを柔軟に調整する）を標準で提供する。 | [Bootstrapとは](https://techacademy.jp/magazine/6270) |
| tensorflow | googleが提供する深層学習用のライブラリ| [Wikipedia](https://ja.wikipedia.org/wiki/TensorFlow) |

# その他

その他、重要なJavaに関する？キーワードまとめをします。

| キーワード | 説明 | リンク |
|:--:|:--:|:--:|
| POJO | フィールドとアクセサメソッド（sette, getter）が実装されカプセル化が行われているクラス　|  |
| Serializableなクラス | 「生成されたオブジェクトのデータを、ファイルやDBに対して保存・復元出来る」クラスのこと | [Serializableとは](http://blog.livedoor.jp/mochikoastech/archives/01135.html) |
| JavaBeans | 上述のPOJOに加えて、java.io.Serializableが実装されたクラスのこと。 | [JavaBeanとは](https://qiita.com/ukitiyan/items/3b7f1ea7eb3d208ed5fe) |
| Beanを「永続化」する | Serializableを実装することと同義。 | [JavaBeansとJSP](http://www.wakhok.ac.jp/~tomoharu/web2004/text/index_c4.html) |
| フォースタンス理論 |  |  |
| 殺虫剤のパラドックス | 古い部分のテストケースが正常に動作しない。回帰テストをする際に起こりうる。 |  |
| プランニングポーカー |  |  |
| JDBC |  |  |
| デザインパターン |  |  |
| Block Vlock |  |  |
| コメントには設計理由を |  |  |
| ランサーズ |  |  |
