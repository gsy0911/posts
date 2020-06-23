# はじめに

普段私が`Lambda`を利用する際は、`chalice`を利用して`Lambda`関数の作成とデプロイをしています。

`chalice`は便利なのですが、デプロイ時に毎回`pip install`して必要な`package`を集めているみたいです。
そのため、`pandas`などの重めの`package`を利用している場合は、デプロイに時間がかかります。

そこで、あらかじめ利用する`package`を登録しておく`Lambda`のサービスの`LambdaLayers`を活用します。


## 前提

`LambdaLayers`を作成する際の前提条件などです。

1. `LambdaLayers`は`packages`を`zip`で固めたファイルを利用して、あらかじめ`Lambda`が利用できる状態にすること[参考](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/configuration-layers.html)
1. `package`の配置などフォルダ構成も決まっている
1. `pandas`などの一部の`package`は`AmazonLinux`上で`pip install`する必要がある[参考](https://qiita.com/thimi0412/items/4c725ec2b26aef59e5bd)
1. `EC2`の上でやるのが簡単だけれど、`VPC`などの環境がない場合は作成するのが無駄である（`VPC`そのものに料金がかかる）。
1. `Lambda`には`/tmp`というプログラムがアクセスできる領域がある。

この、 (3) の条件が地味にきつくて、ローカルなどで作成した`zip`ファイルではうまく`Lambda`が認識してくれないことがありました。
また (4) の`EC2`上でやっても良いのですが、サーバーレスしか利用していないプライベート開発だと、
お金をかけたく無いためなるべく`Lambda`のみで完結したいと言う（自分の）要望がありました。

# 実装

## 今回の実装の流れ

上記の`前提`を踏まえて今回は以下の方法で`LambdaLayers`に必要な`zip`ファイルを作成します。

`Lambda`の`/tmp`領域に`package`を`install`して、`zip`で固めて`S3`にアップロードする
実際の`LambdaLayers`の登録は手作業で行う。

## ソースコード

 `Lambda`で実行するプログラムは以下の通りです。

```python:lambda_function.py
import json
import subprocess
import zipfile
import boto3
from pathlib import Path


def lambda_handler(event, context):
    # 引数の確認
    if "package_list" not in event:
        return {"error": "[package_list]が存在しません"}
    package_list = event['package_list']
    if type(package_list) is not list:
        return {"error": "[package_list]がlistではありません"}    
    if "s3_bucket" not in event:
        return {"error": "[s3_bucket]が存在しません"}
    if "s3_key" not in event:
        return {"error": "[s3_key]が存在しません"}
    
    # pythonと言うディレクトリを作成し、そこにpackageをインストール
    mkdir_cmd = "mkdir -p /tmp/python"
    pip_install_cmd = f"pip install -t /tmp/python {' '.join(package_list)}"
    process = (subprocess.Popen(f"{mkdir_cmd} && {pip_install_cmd}", stdout=subprocess.PIPE, shell=True).communicate()[0]).decode('utf-8')
    print('commands...\n'+process)

    # Pathオブジェクトを生成
    p = Path("/tmp/python")

    # zipにinstallしたpackageを追加    
    with zipfile.ZipFile('/tmp/python_lib.zip', 'w', compression=zipfile.ZIP_DEFLATED) as new_zip:
        for f in list(p.glob("**/*")):
            new_zip.write(f, arcname=str(f).replace("/tmp/", ""))

    #S3オブジェクトを取得しupload
    s3 = boto3.resource('s3')
    bucket = s3.Bucket(event['s3_bucket'])
    bucket.upload_file('/tmp/python_lib.zip', event['s3_key'])
    return {
        "success": f"s3://{event['s3_bucket']}/{event['s3_key']}に出力しました"
    }

```

## `Lambda`設定

インストールする`package`に合わせて適宜変えてください。

* メモリ： `512[MB]`
* timeout： `5[min]`
* Role： アップロードする`S3`にアクセス権限を付与。

## 引数例

実行する際に、以下の引数を渡すと`package_list`に含まれる`package`をインストールします。
また、`s3_bucket`と`s3_key`に指定した箇所へ`zip`ファイルをアップロードしてくれます。

```json:event
{
  "package_list": ["pandas", "lxml", "requests", "beautifulsoup4"],
  "s3_bucket": "<s3のバケット名>",
  "s3_key": "lambda_layer/python_lib.zip"
}
```

上記の例を利用して`zip`ファイルを作成した場合は、およそ`43.1[MB]`程度の`zip`ファイルが作成されます。

# おわりに

ということで、上記のソースコードを`Lambda`にコピーして実行すれば、
`LambdaLayers`の登録に必要な`zip`を作成できるようになりました。

注意：今回は仕方なしに`Python`から`shell`を実行しています。
基本的に、`Python`から`shell`を呼び出すことはよくないと思うので、
利用するのはどうしても仕方がない場合にしましょう。


