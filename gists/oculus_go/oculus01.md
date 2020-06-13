# リンク

| 回数 | 内容 |
|:--:|:--:|
| ここ | セットアップ |
| [第二回](https://gist.github.com/gsy0911/8ebf8b16c54bfa6a7eb39848363ad5ea) | OculusGoのコントローラーからの入力取得と表示 |
| [第三回](https://gist.github.com/gsy0911/58da09800b2a2dc2a0e8f0b44ec5edcb) | VR空間で歩く |

# はじめに

OculusGoをせっかく手に入れたので，自分でVRアプリを作ろうと思う！
ただ，スペックが低いPCばっかりなので少しつらい．．．

# 環境構築

以前使ったLenovo君は容量に関して心配なので，しばらく眠っていたLIVA X君を引っ張り出して開発環境を整えていく．
とりあえず，サイトを参考にして以下の環境を整えた．

* OS : Windows 10 (1803 - buid 17134.81)
* unity : version 2018.1.1f1
* Android Studio : version 3.1.2 (android-studio-ide-173.4720617-windows.exe)
* Java : version 1.8 (jdk8u171-windows-x64.exe)

# 手順

## ステップ1

まずは，Unityでの「Build Settings」でAndroidに変更するところまで行う
1. Unityをインストール
1. Android Studioをインストール（ここでAndroid SDKの置き換えとかが必要な場合があるらしい．その場合は[ここ](http://greety.sakura.ne.jp/redo/2018/05/oculus-go-mirage-solo-unity-android-sdk.html)を参考のこと）
1. Android 7.1.1（Nougat）API Level 25のSDKを「SDK Manager」より追加する
1. JDK 1.8をインストール

この状態で「File」，「Build Setting」を選び，「Android」を選択してエラーが出なければ，「Switch Platform」で変更する．
エラーが出る場合は「Edit」，「Preferences」，「External Tools」の中の「Android SDK」と「JDK」のパスを正しくセットする．

## ステップ2

環境が整ったら，アプリをOculusGoに転送する準備を始める．

1. OculusGoの開発者モードを有効にする([参考](http://mogamitsuchikawa.hatenablog.jp/entry/2018/05/12/123422))
1. OculusGo adb driverをインストール
1. 「File」，「Build Setting」，「Player Settings」を選び，以下の項目を変更する
1. 「Other Settings」-「Package Name」を任意に変更
1. 「Minimum API Level」をAndroid 7.1に変更
1. 「XR Settings」-「Virtual Reality Supported」にチェックを入れ，「Virtual Reality SDKs」をOculusに変更
1. 最後に「File」，「Build Setting」，「Build System」をInternalに変更

ここまでできたら，「File」，「Build & Run」で実行する．
無事にインストール出来たら完了！！

# 参考サイト
* [UnityでOculus Goの開発環境をセットアップしてみた](http://www.yoshidayo.com/entry/2018/05/09/103503)
* [Win10でUnity 2017.4.1f1（OculusGo推奨ver.）向けAndroid SDK環境構築の話 ](http://greety.sakura.ne.jp/redo/2018/05/oculus-go-mirage-solo-unity-android-sdk.html)
* [Oculus Go の開発をやってみる(後編)](http://mogamitsuchikawa.hatenablog.jp/entry/2018/05/12/123422)
* [ZENRIN 3D model Data](http://www.zenrin.co.jp/product/service/3d/asset/)
* [【Unity2018対応】Androidビルドでエラーが出る場合の対処法](http://nn-hokuson.hatenablog.com/entry/2017/09/05/202327)