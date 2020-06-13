# はじめに

Java で半角と全角が混ざった状態で String#format を使うと文字の位置がずれてしまうことが起きた。
あまり調べても簡単に解決する方法がなかったので，パッと関数を組んでみた。

## 解決策

早速だけれど，以下が解決策
（とりあえず以下の`format()`と`getByteLength()`だけコピー&ペーストすれば解決します）

```java:main
    public static void main(String[] args){
        //ずれるformat文
        System.out.println(String.format("%-20s", "あああああ")+" | ");
        System.out.println(String.format("%-20s", "aaaaa")+" | ");
        //ずれないformat文
        System.out.println(format("あああああ", 20)+" | ");
        System.out.println(format("aaaaa", 20)+" | ");
    }

    private static String format(String target, int length){
        int byteDiff = (getByteLength(target, Charset.forName("UTF-8"))-target.length())/2;
        return String.format("%-"+(length-byteDiff)+"s", target);
    }

    private static int getByteLength(String string, Charset charset) {
        return string.getBytes(charset).length;
    }
```

```markdown:result
あああああ                | 
aaaaa                | 
あああああ            | 
aaaaa                | 
```

(Qiita へコピー&ペーストした時に`|`がずれたのは内緒...)
以下余談です。

## ずれる理由と対策法

そもそもなぜ半角と全角でずれるのかというと，`UTF-8` などで文字を表現すると`byte` の数が以下のように異なるからです。

* 日本語 : 3byte
* 英数字 : 1byte

なので，`main`中の`int byteDiff`で調整しています。
また，以下の関数より、日本語は 3byte で、英語は 1byte であることが分かります。

```java:analyze
    public static void main(String[] args){
        analyze("あああああ");
        analyze("aaaaa");
    }

    private static void analyze(String target){
        System.out.println("\n"+target);
        int length=20;
        System.out.println(format("length()", length)+":"+target.length());
        System.out.println(format("UTF-8 length()", length)+":"+getByteLength(target, Charset.forName("UTF-8")));
        System.out.println(format("diff", length)+":"+(getByteLength(target, Charset.forName("UTF-8"))-target.length())/2);
    }

```

```markdown:result
あああああ
length()            :5
UTF-8 length()      :15
diff                :5

aaaaa
length()            :5
UTF-8 length()      :5
diff                :0
```


# 参考

* [Java：ゼロ埋め、半角スペース埋めする方法](http://write-remember.com/program/java/format/)
* [[Java] Stringのバイト数を取得する](https://qiita.com/hecateball/items/d0edb6ff65d8f6b18ec4)
