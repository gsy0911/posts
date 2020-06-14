
* [AviUtl](http://spring-fragrance.mints.ne.jp/aviutl/)

* mp4出力

[rigayaの日記兼メモ帳](https://rigaya34589.blog.fc2.com/blog-entry-139.html)
1. `x264guiEx_2.57v3_7zip.7z`内の`auto_setup.exe`を実行

* 

* mp4読み込み


[参考](http://aviutl.info/l-smash-works/)
1. `L-SMASH_Works_r804_plugins.zip`

今回は古いバージョンのを入れたため`/AviUtl/`直下に`exedit.ini`を作成し，以下を追記

```
[extension]
; 拡張子とメディアオブジェクトの種類を関連付けます
.avi=動画ファイル
.avi=音声ファイル
.mpg=動画ファイル
.mpg=音声ファイル
.mp4=動画ファイル
.mp4=音声ファイル
.flv=動画ファイル
.flv=音声ファイル
.dv=動画ファイル
.bmp=画像ファイル
.jpg=画像ファイル
.jpeg=画像ファイル
.png=画像ファイル
.gif=画像ファイル
.wav=音声ファイル
.mp3=音声ファイル
.txt=テキスト

[script]
dll=lua51.dll
```

[プラグイン](https://videoinfo.tenchi.ne.jp/?DirectShow%20File%20Reader%20%A5%D7%A5%E9%A5%B0%A5%A4%A5%F3%20for%20AviUtl)
1. `ds_input026a.lzh`内の`ds_input.aui`を`/AviUtl/Plugins/`以下に配置


* [LargeMemoryAwareを使う](https://indoorotaku.blogspot.com/2014/02/aviutl64bit-os.html)

## 2020/04/03
AviUtlを利用してアップコンバートする[参考](http://maina.world.coocan.jp/garakuta/x264upenc/index.htm)。

* [Lanczos 3-lobed 拡大縮小](http://www.marumo.ne.jp/auf/#lanczos3)
  * lanczos3-0.5.7.lzh
* [WarpSharpMTex](https://w.atwiki.jp/aviutl41991/pages/37.html)
  * warpsharpmt_v133ex6.zip
* [エッジレベル調整](https://rigaya34589.blog.fc2.com/blog-category-11.html)
  * edgelevelIMTv8.zi
