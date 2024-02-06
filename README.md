# vnet-peering-base-env

## はじめに
本 Bicep は VNet Peering 検証環境用 のベース環境を作成するBicepです

> [!NOTE]
> - オンプレ環境に接続するために VPN Gateway を `Onpre-VNet`, `Hub-VNet` に作成する想定ですが、本 Bicep では VPN Gateway は作成されません（必要となる `GatewaySubnet` は作成されます）
> - Spoke間の接続を行うためにパケットフォワーダー (Azure Firewall) を `Hub-VNet` に作成する想定ですが、本 Bicep では Azure Firewall は作成されません（必要となる `AzureFirewallSubnet` は作成されます） 

> [!WARNING]
> 本環境は HUB の踏み台サーバーを経由して Spokeやオンプレ の VM にアクセスするような構成です。NSG で SSH(22) への接続を許可するルールを作成していますが、セキュリティリスクが高いため、あくまでも検証用途としてご利用ください（本来は Azure Bastion や Azure Firewall を利用して踏み台サーバーへアクセスさせるべきですが、費用を下げるため NSG で穴あけを行っています）

## 構成図
![](/images/vnet-peering-base-topology.png)

### 前提条件
ローカルPCでBicepを実行する場合は Azure CLI と Bicep CLI のインストールが必要となります。私はVS Code (Visual Studio Code) を利用してBicepファイルを作成しているのですが、結構使いやすいのでおススメです。以下リンクに VS Code、Azure CLI、Bicep CLI のインストール手順が纏まっています

https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/install

## 使い方
本リポジトリーをローカルPCにクローンし、パラメータファイル (main.prod.bicepparam) を修正してご利用ください

**main.prod.bicepparam**
![](/images/vnet-peering-base-bicepparam.png)

> [!IMPORTANT]
> NSGルール作成用の ***myipaddress*** の修正は必須となります。それ以外のパラメータの修正は任意で実施してください。Azureに接続するクライアントのパブリックIPアドレスが分からない場合は[こちらのサイト](https://www.cman.jp/network/support/go_access.cgi)で確認することができます

※Git を利用できる環境ではない場合はファイルをダウンロードしていただくでも問題ないと思います。その場合は、以下の構成でローカルPCにファイルを設置してください

```
main.bicep
main.prod.bicepparam
∟ modules/
　　∟ hubEnv.bicep
　　∟ onpreEnv.bicep
　　∟ spoke1Env.bicep
　　∟ spoke2Env.bicep
```

## 実行手順 (Git bash)

#### 1. Azureへのログインと利用するサブスクリプションの指定
```
az login
az account set --subscription <利用するサブスクリプション名>
```
> [!NOTE]
> az login を実行するとWebブラウザが起動するので、WebブラウザにてAzureへのログインを行う

#### 2. ディレクトリの移動（main.bicep を設置したディレクトリへ移動）
```
cd <main.bicepを設置したディレクトリ>
```

#### 3. デプロイの実行
```
az deployment sub create --location japaneast -f main.bicep -p main.prod.bicepparam
```

#### 4. Azureからのログアウト
```
az logout
```

## 実行時のエラーについて
デプロイ実行時に以下のエラーが出た場合は、すでに作成された環境を一度削除してから再度デプロイを行ってください
> DNS record hub-jump-centos.japaneast.cloudapp.azure.com is already used by another public IP.

 本エラーは hub の踏み台サーバ (hub-jump-centos) の DNS レコードが重複する場合に発生します。DNSレコードはパラメータファイルで指定した ***hubvmName1*** を利用して作成されるため複数デプロイを行うと DNS レコードが重複する仕様となります。本エラーを解消するためには以下2つのどちらかの対応を実施してください

1. 同一サブスクリプションに本Bicepを利用して複数の環境をデプロイする場合は ***hubvmName1*** を環境毎に異なる名前を指定する

2. ***hubEnv.bicep*** の以下の行を削除する (削除することにより、FQDNによる hub の踏み台サーバへのアクセスはできなくなるが、IPアドレスでのアクセスは引き続き可能)
https://github.com/takutsu001/vnet-peering-base-env/blob/1787ab596d20c58241fc7c631e564bbd75c56bf4/modules/hubEnv.bicep#L120C1-L122C6