# デプロイ

## Visual Studio Code から Azure Functions をデプロイする

1. VSCodeのextentionをインストール[Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
1. VScodeの左側に出てきた山にログイン
  * sing in to Azure
  * 事前に何か作成している場合は、登録しているfunction appが表示されるっぽい
1. 一旦目標はdeployなので `Azure Function Core Tools` のインストールは見送り

## ローカルの Functions アプリを作成する


# 参考

* [Visual Studio Code から Azure Functions をデプロイする](https://docs.microsoft.com/ja-jp/azure/javascript/tutorial-vscode-serverless-node-01)

## Azure Blob Storage(Binary Large OBject)

* [基礎知識てきな](https://licensecounter.jp/azure/blog/word/azureblob.html)
* [うえと合わせてまとめたらよさそう](https://www.cloudou.net/storage/blob006/)
### create storage account
* Basics
  * Subscription
  * Resource group
  * storage account name
  * location
  * persormance: (`Standard`, `Premium`)
  * Accound kind: (`StorageV2 (general purpose v2)`, `Storage (general purpose v1)`, `BlobStorage`)
  * Replication: (`Locally-redundant storage (LRS)`, `Geo-redundant storage (GRS)`, `Read-access-geo-redundant storage(RA-GRS)`) LRSでも1つのregionで3回レプリケートをして、イレブンナインを達成する。[参考](https://docs.microsoft.com/ja-jp/azure/storage/common/storage-redundancy)
* Networking
  * Public endpoin (all networks)
  * public endpoin (selected networks)
    * Virtual network subscription
    * Virtual network
  * Private endpoint
    * よくわからんが、[参考](https://docs.microsoft.com/ja-jp/azure/private-link/create-private-endpoint-storage-portal)
* Advanced
  * Secure Transfer required: (`Disabled`, >`Enabled`) [参考](https://docs.microsoft.com/ja-jp/azure/storage/common/storage-require-secure-transfer)
  * Azure Files: required replication is GRS: [参考](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-planning#onboard-to-larger-file-shares-standard-tier)
  * Blob soft delete: 論理削除のみを行う。消されたファイルも存在しているため課金対象になる[参考](https://docs.microsoft.com/ja-jp/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal#pricing-and-billing)
  * Hierarchical namespace: 階層的なフォルダ構造。大規模データを扱う際に有用。料金がちょっと高くなる[参考](https://azure.microsoft.com/en-us/services/storage/data-lake-storage/)

deployment result
* deployment name: Microsoft.StorageAccount-20200306222236
* start time: Start time: 2020/3/6 22:44:01
* Correlation ID: ff40f2a9-677f-4ce8-abb1-9c1d98fdc0f1
* Subscription
* Resource group

## Storage accountで利用できるストレージ形態？
[Pythonを利用したアクセス方法](https://docs.microsoft.com/ja-jp/azure/storage/common/storage-samples-python?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

以下の種類のストレージ形態が選べる
* Containers
* File shares
* Tables
* Queues

* blob serviceって何?

## availability Zones
[Azure の Availability Zones の概要](https://docs.microsoft.com/ja-jp/azure/availability-zones/az-overview)

* [Blob Storageに配置されたオブジェクトをAzure Functionsで処理する](https://tech-lab.sios.jp/archives/7505)
* [AzureBlobStorageをPythonから使う](https://qiita.com/garicchi/items/a07c32df5e3010548736)
