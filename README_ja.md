# SIEM on Amazon Elasticsearch Service

[In English](README.md)

SIEM on Amazon Elasticsearch Service (Amazon ES) は、セキュリティインシデントを調査するためのソリューションです。AWS のマルチアカウント環境下で、複数種類のログを収集し、ログの相関分析や可視化をすることができます。デプロイは、AWS CloudFormation または AWS Cloud Development Kit (AWS CDK) で行います。20分程度でデプロイは終わります。AWS サービスのログを Simple Storage Service (Amazon S3) のバケットに PUT すると、自動的に ETL 処理を行い、SIEM on Amazon ES に取り込まれます。ログを取り込んだ後は、ダッシュボードによる可視化や、複数ログの相関分析ができるようになります。

![Sample dashboard](./docs/images/dashboard-sample.jpg)

## アーキテクチャ

![Architecture](./docs/images/aes-siem-architecture.png)

## 対応ログ

SIEM on Amazon ES は以下のログを取り込むことができます。

|AWS Service|Log|
|-----------|---|
|AWS CloudTrail|CloudTrail Log Event|
|Amazon Virtual Private Cloud (Amazon VPC)|VPC Flow Logs|
|Amazon GuardDuty|GuardDuty finding|
|AWS WAF|AWS WAF Web ACL traffic information<br>AWS WAF Classic Web ACL traffic information|
|Elastic Load Balancing|Application Load Balancer access logs<br>Network Load Balancer access logs<br>Classic Load Balancer access logs|
|Amazon CloudFront|Standard access log<br>Real-time log|
|Amazon Simple Storage Service (Amazon S3)|access log|
|Amazon Route 53 Resolver|VPC DNS query log|

対応ログは、[Elastic Common Schema](https://www.elastic.co/guide/en/ecs/current/index.html) に従って正規化しています。ログのオリジナルと正規化したフィールド名の対応表は [こちら](docs/suppoted_log_type.md) をご参照ください。

## ダッシュボード

[こちら](docs/dashboard_ja.md) をご参照ください

## 開始方法

CloudFormation テンプレートを使って、SIEM on Amazon ES のドメインをパブリックアクセスに作成します。Amazon VPC 内へのデプロイやカスタマイズをする場合の手順は [こちら](docs/deployment_ja.md) をご参照ください。

IP アドレスに国情報や緯度・経度のロケーション情報を付与することができます。ロケーション情報は [MaxMind 社](https://www.maxmind.com)の GeoLite2 Free をダウンロードして活用します。ロケーション情報を付与したい方は MaxMind にて無料ライセンスを取得してください。

注) CloudFormation テンプレートは Amazon ES を **t3.small.elasticsearch インスタンス の最小構成でデプロイします。SIEM は、多くのログを集約して負荷が高くなるため、小さい t2/t3 を避けて、メトリクスを確認しつつ最適なインスタンスを選択してください。** また、インスタンスの変更、ディスクの拡張、UltraWarm の使用等は、AWS マネジメントコンソールから直接行ってください。SIEM on Amaozn ES の CloudFormation テンプレートは Amazon ES に対しては初期デプロイのみで、ノードの変更、削除等の管理はしません

### 1. クイックスタート

SIEM on Amazon ES をデプロイするリージョンを選択してください。ご希望のリージョンがこのリストない場合は、手順に従って CloudFormation のテンプレートを作成してください。

| Region | CloudFormation |
|--------|----------------|
| N. Virginia (us-east-1) |[![Deploy in us-east-1](./docs/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=aes-siem&templateURL=https://aes-siem-us-east-1.s3.amazonaws.com/siem-on-amazon-elasticsearch.template) |
| Oregon (us-west-2) |[![Deploy in us-west-2](./docs/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=aes-siem&templateURL=https://aes-siem-us-west-2.s3.amazonaws.com/siem-on-amazon-elasticsearch.template) |
| Tokyo (ap-northeast-1) |[![Deploy in ap-northeast-1](./docs/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/new?stackName=aes-siem&templateURL=https://aes-siem-ap-northeast-1.s3.amazonaws.com/siem-on-amazon-elasticsearch.template) |
| Frankfurt (eu-central-1) |[![Deploy in eu-central-1](./docs/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=aes-siem&templateURL=https://aes-siem-eu-central-1.s3.amazonaws.com/siem-on-amazon-elasticsearch.template) |
| London(eu-west-2) |[![Deploy in eu-west-2](./docs/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-2#/stacks/new?stackName=aes-siem&templateURL=https://aes-siem-eu-west-2.s3.amazonaws.com/siem-on-amazon-elasticsearch.template) |

### 2. CloudFormation テンプレートの作成

クイックスタートでデプロイされた方は CloudFormation テンプレートの作成はスキップしてください。

#### 2-1. 準備

Amazon Linux 2 を実行している Amazon Elastic Compute Cloud (Amazon EC2) インスタンスを使って CloudFormation テンプレートを作成します

前提の環境)

* Amazon Linux 2 on Amazon EC2
* Python 3.7
* git

Python 3と git がインストールされてない場合は以下を実行

```shell
sudo yum -y install python3 git
```

#### 2-2. SIEM on Amazon ES の clone

GitHub レポジトリからコードを clone します

```shell
git clone https://github.com/aws-samples/siem-on-amazon-elasticsearch.git
```

#### 2-3. 環境変数の設定

```shell
export TEMPLATE_OUTPUT_BUCKET=<YOUR_TEMPLATE_OUTPUT_BUCKET> # Name for the S3 bucket where the template will be located
export SOLUTION_NAME="siem-on-amazon-elasticsearch" # name of the solution
export AWS_REGION=<AWS_REGION> # region where the distributable is deployed
```

##### 注) $TEMPLATE_OUTPUT_BUCKET は S3 バケット名です。事前に作成してください。デプロイ用のファイルの配布に使用します。ファイルはパブリックからアクセスできる必要があります。テンプレート作成時に使用する build-s3-dist.sh は S3 バケットの作成をしません

#### 2-4. AWS Lambda 関数のパッケージングとテンプレートの作成

```shell
cd siem-on-amazon-elasticsearch/deployment/cdk-solution-helper/ && ./step1-build-lambda-pkg.sh && cd ..
chmod +x ./build-s3-dist.sh && ./build-s3-dist.sh $TEMPLATE_OUTPUT_BUCKET $SOLUTION_NAME $VERSION
```

#### 2-5. Amazon S3 バケットへのアップロード

```shell
aws s3 cp ./global-s3-assets s3://$TEMPLATE_OUTPUT_BUCKET/ --recursive --acl bucket-owner-full-control
aws s3 cp ./regional-s3-assets s3://$TEMPLATE_OUTPUT_BUCKET/ --recursive --acl bucket-owner-full-control
```

##### 注) コマンドを実行するために S3 バケットへファイルをアップロードする権限を付与し、アップロードしたファイルに適切なアクセスポリシーを設定してください

#### 2-6. SIEM on Amazon ES のデプロイ

コピーしたテンプレートは、`https://s3.amazonaws.com/$TEMPLATE_OUTPUT_BUCKET/siem-on-amazon-elasticsearch.template` にあります。このテンプレートを AWS CloudFormation に指定してデプロイしてください。

### 3. Kibana の設定

約20分で CloudFormation によるデプロイが完了します。次に、Kibana の設定をします。

1. AWS CloudFormation コンソールで、作成したスタックを選択。画面右上のタブメニューから「出力」を選択。Kibana のユーザー名、パスワード、URL を確認できます。この認証情報を使って Kibana にログインしてください
1. Kibana の Dashboard等 のファイルを[**ここ**](https://aes-siem.s3-ap-northeast-1.amazonaws.com/assets/saved_objects.zip) からダウンロードします。ダウンロードしたファイルを解凍してください
1. Kibana のコンソールに移動してください。画面左側に並んでいるアイコンから「Management」 を選択してください、「Saved Objects」、「Import」、「Import」の順に選択をして、先ほど解凍したZIPファイルの中ある「dashboard_v770.ndjson」をインポートしてください
1. インポートした設定ファイルを反映させるために一度ログアウトしてから、再ログインをしてください

### 4. ログの取り込み

S3 バケットの aes-siem-*[AWS アカウント ID]*-log にログを出力してください。ログは自動的に SIEM on Amazon ES に取り込まれて分析ができるようになります。

AWS の各サービスのログを S3 バケットへの出力する方法は、[こちら](docs/configure_aws_service_ja.md) をご参照ください。

## 設定変更

### デプロイ後の Amazon ES ドメインの変更

Amazon ES のアクセスポリシーの変更、インスタンスのスペック変更、AZ の追加と変更、UltraWarm への変更等の Amazon ES ドメイン自体の変更は、AWS マネジメントコンソールから実行してください

### インデックス管理とカスタマイズ

SIEM on Amazon ES はログをインデックスに保存しており、デフォルトでは毎月1回ローテーションをしています。この期間を変更や、AWS 以外のログを取り込みたい方は、[こちら](docs/configure_siem_ja.md) をご参照ください。

## バッチ処理による過去ログの取り込み

Python スクリプトの es-loader をローカル環境で実行することで、すでに S3 バケット に保存されている過去のログを SIEM on Amazon ES に取り込むことができます。

## 作成される AWS リソース

CloudFormation テンプレートで作成される AWS リソースは以下の通りです。AWS Identity and Access Management (IAM) のリソースは AWS マネジメントコンソールから確認してください。

|AWS Resource|Resource Name|目的|
|------------|----|----|
|Amazon ES 7.X|aes-siem|SIEM 本体|
|S3 bucket|aes-siem-[AWS_Account]-log|ログを集約するため|
|S3 bucket|aes-siem-[AWS_Account]-snapshot|Amazon ES の手動スナップショット取得|
|S3 bucket|aes-siem-[AWS_Account]-geo|ダウンロードした GeoIP を保存|
|Lambda function|aes-siem-es-loader|ログを正規化し Amazon ES へロード|
|Lambda function|aes-siem-deploy-aes|Amazon ES のドメイン作成|
|Lambda function|aes-siem-configure-aes|Amazon ES の設定|
|Lambda function|aes-siem-geoip-downloader|GeoIP のダウンロード|
|Lambda function|aes-siem-BucketNotificationsHandler|ログ用 S3 バケットのイベント通知を設定|
|AWS Key Management Service<br>(AWS KMS) CMK & Alias|aes-siem-key|ログの暗号化に使用|
|Amazon SQS Queue|aes-siem-dlq|Amazon ES のログ取り込み失敗用Dead Ltter Queue|
|CloudWatch Events|aes-siem-CwlRuleLambdaGeoipDownloader|aes-siem-geoip-downloaderを毎日実行|
|Amazon SNS Topic|aes-siem-alert|Amazon ES の Alerting の Destinations で選択|
|Amazon SNS Subscription|inputd email|Alert の送信先メールアドレス|

## クリーンアップ

1. AWSマネジメントコンソールの CloudFormation からスタックの aes-siem を削除
1. 手動で次の AWS リソースを削除
    * Amazon ES ドメイン: aes-siem
    * Amazon S3 バケット: aes-siem-[AWS_Account]-log
    * Amaozn S3 バケット: aes-siem-[AWS_Account]-snapshot
    * Amazon S3 バケット: aes-siem-[AWS_Account]-geo
    * AWS KMS カスタマーマネジメントキー: aes-siem-key
        * 削除は注意して行ってください。ログをこのカスタマーマネジメントキーで暗号化していると、キーの削除後はそのログは読み込むことができなくなります。

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

This product uses GeoLite2 data created by MaxMind and licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/), available from [https://www.maxmind.com](https://www.maxmind.com).
