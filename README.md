# 【AWS】手を動かして学ぶAWS AWS Serverless Application Model(SAM)

## はじめに

この記事ではAWS Serverless Application Model(以下、 本文ではSAM)を使って、サーバーレスアプリケーションを開発するチュートリアルを書きます。
主な内容としては実践したときのメモを中心に書きます。（忘れやすいことなど）
誤りなどがあれば修正していく想定です。

## AWS SAMとは

## ハンズオン

おおまかな手順は以下のとおりです。

- 環境のセットアップ
- SAMの利用
- AWS CLIでリソースをみる

## セットアップ方法

では、まずは環境をセットアップしていきましょう。

### AWS SAM CLI のインストール

[Install the AWS SAM CLI - AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)に従って、セットアップを行います。

AWS SAM CLIをインストールします。以下のコマンドを実行してください。

```bash
wget https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
sudo ./sam-installation/install
rm -rf aws-sam-cli-linux-x86_64.zip sam-installation
```

動作確認を行います。

```bash
sam --version
```

実行結果

```bash
SAM CLI, version 1.146.0
```

## 準備体操：AWS SAMでHello Worldアプリケーションを作成して動作確認

Powertools for AWS Lambda (Python)のチュートリアルに入る前に、AWS SAMでHello Worldアプリケーションを作成して動作確認を行います。
これからのやることのおおまかな流れは以下のとおりです。

- Pythonのバージョンを確認
- AWS SAMでHello Worldアプリケーションを作成
- ローカルでビルドとAPI起動

まずはPythonのバージョンを確認します。

```bash
python3 --version
```

今回はPython 3.12.1を使用します。このバージョンを使ってAWS SAMでHello Worldアプリケーションを作成します。
以下のコマンドを実行して、プロジェクトを作成します。

```bash
sam init --runtime python3.12
```

ディレクトリを移動します。

```bash
cd powertools-quickstart
```

ローカルでビルドとAPI起動を行います。

```bash
sam build && sam local start-api
```

`http://127.0.0.1:3000/hello`でアクセスします。
Web画面上に以下のようなメッセージが表示されたら成功です。

```text
{"message": "hello world"}
```

curlコマンドでアクセスする場合は以下のとおりです。※ターミナルを次の行で表示するために改行が入るようにしています。

```bash
curl http://127.0.0.1:3000/hello && echo ""
```

sam local invokeによる実行も試してみます。以下のコマンドを実行します。
イベントファイルとリソース指定で実行する例を示します。

```bash
sam local invoke HelloWorldFunction -e events/event.json
```

または短縮版で実行することもできます。

```bash
sam local invoke -e events/event.json
```

問題なく動作したら、次にデプロイを行います。以下のコマンドを実行します。

```bash
sam build && sam deploy --guided
```

いくつか質問が表示されるので、以下のように入力して進めてください。

```
Setting default arguments for 'sam deploy'
=========================================
Stack Name [powertools-quickstart]: 
AWS Region [ap-northeast-1]: 
#Shows you resources changes to be deployed and require a 'Y' to initiate deploy
Confirm changes before deploy [Y/n]: Y
#SAM needs permission to be able to create roles to connect to the resources in your template
Allow SAM CLI IAM role creation [Y/n]: Y
#Preserves the state of previously provisioned resources when an operation fails
Disable rollback [y/N]: y
HelloWorldFunction has no authentication. Is this okay? [y/N]: y
Save arguments to configuration file [Y/n]: y
SAM configuration file [samconfig.toml]: 
SAM configuration environment [default]: 
```

デプロイがはじまるのでしばらく待ちます。途中で`changeset`の確認が表示されるので`Y`を入力して進めてください。
デプロイが完了したら、以下のようなメッセージが表示されます。

```
Successfully created/updated stack - powertools-quickstart in ap-northeast-1
```

デプロイが完了したら、samのコマンドでデプロイしたLambda関数のエンドポイントを確認します。

```bash
sam list endpoints --output json
```

実行結果の`CloudEndpoint`という項目でProdとStageのURLが確認できます。

```json
  {
    "LogicalResourceId": "ServerlessRestApi",
    "PhysicalResourceId": "XXXXXXX",
    "CloudEndpoint": [
      "https://{PhysicalResourceId}.execute-api.ap-northeast-1.amazonaws.com/Prod",
      "https://{PhysicalResourceId}.execute-api.ap-northeast-1.amazonaws.com/Stage"
    ],
    "Methods": [
      "/hello['get']"
    ]
  }
```

`CloudEndpoint`のURLに`/hello`を付与してアクセスします。ブラウザまたはcurlコマンドでアクセスしてください。

```bash
curl https://{PhysicalResourceId}.execute-api.ap-northeast-1.amazonaws.com/Prod/hello && echo ""
```

実行結果

```json
{"message": "hello world"}
```

### AWS CLI のインストール

まずはAWS CLIをインストールします。最新版のAWS CLIを公式インストーラーでインストールします。

```bash
# 1. インストーラーをダウンロード
curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip"

# 2. unzipがインストールされていない場合はインストール
sudo apt update && sudo apt install unzip -y  # Ubuntu/Debian系
# または
sudo yum install unzip -y                     # CentOS/RHEL系

# 3. ダウンロードしたファイルを展開
unzip awscliv2.zip

# 4. インストール実行
sudo ./aws/install

# 5. インストール確認
aws --version

# ダウンロードしたzipファイルと展開したディレクトリを削除してクリーンアップします。
rm  "awscliv2.zip"

# 解凍したディレクトリを削除
rm -rf aws
```

### AWS CLI の設定

AWS CLIの設定を行います。今回はAWS IAM Identity Center(旧AWS SSO)を使用してログインします。まずは以下のコマンドを実行して、SSOの設定を行います。

```bash
aws configure sso
```
設定時に以下の情報の入力が求められます：
- **SSO start URL**: 組織のSSO開始URL（例：`https://my-company.awsapps.com/start`）
- **SSO Region**: SSOが設定されているリージョン（例：`us-east-1`）
- **アカウント選択**: 利用可能なAWSアカウントから選択
- **ロール選択**: 選択したアカウントで利用可能なロールから選択
- **CLI default client Region**: デフォルトのAWSリージョン（例：`ap-northeast-1`）
- **CLI default output format**: 出力形式（`json`、`text`、`table`のいずれか）
- **CLI profile name**: プロファイル名（`default`とします。）

SSOの設定が完了したら、以下のコマンドでログインを実行します。

```bash
aws sso login
```

### AWS CLI の動作確認

AWS CLIが正しくインストールされ、SSOでログインできているか確認します。AWS STSで認証情報を確認します。

```bash
aws sts get-caller-identity
```

## まとめ

## AWS CLI インストールと SSO ログイン手順 (Linux環境)

このガイドでは、Linux環境でAWS CLIをインストールし、AWS SSOを使用してログインするまでの手順を説明します。

## 前提条件

- Linux環境（Ubuntu、CentOS、Amazon Linux等）
- インターネット接続
- 管理者権限（sudoが使用可能）
- AWS SSO が組織で設定済み
- Python 3.12.1

## AWS CLI のインストール

### 公式インストーラーを使用（推奨）

最新版のAWS CLI v2を公式インストーラーでインストールします。

```bash
# 1. インストーラーをダウンロード
curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip"

# 2. unzipがインストールされていない場合はインストール
sudo apt update && sudo apt install unzip -y  # Ubuntu/Debian系
# または
sudo yum install unzip -y                     # CentOS/RHEL系

# 3. ダウンロードしたファイルを展開
unzip awscliv2.zip

# 4. インストール実行
sudo ./aws/install

# 5. インストール確認
aws --version

# ダウンロードしたzipファイルと展開したディレクトリを削除してクリーンアップします。
rm  "awscliv2.zip"

# 解凍したディレクトリを削除
rm -rf aws
```

## AWS SSO の設定とログイン

### 1. AWS SSO の設定

AWS SSOを使用するための初期設定を行います。

```bash
aws configure sso
```

設定時に以下の情報の入力が求められます：

- **SSO start URL**: 組織のSSO開始URL（例：`https://my-company.awsapps.com/start`）
- **SSO Region**: SSOが設定されているリージョン（例：`us-east-1`）
- **アカウント選択**: 利用可能なAWSアカウントから選択
- **ロール選択**: 選択したアカウントで利用可能なロールから選択
- **CLI default client Region**: デフォルトのAWSリージョン（例：`ap-northeast-1`）
- **CLI default output format**: 出力形式（`json`、`text`、`table`のいずれか）
- **CLI profile name**: プロファイル名（`default`とします。）

### 2. AWS SSO ログイン

設定完了後、以下のコマンドでログインを実行します。

```bash
aws sso login
```

ログイン時の流れ：
1. コマンド実行後、ブラウザが自動的に開きます
2. AWS SSOのログインページが表示されます
3. 組織のIDプロバイダー（例：Active Directory、Okta等）でログイン
4. 認証が成功すると、ターミナルに成功メッセージが表示されます

### 3. ログイン状態の確認

認証情報を確認します。

```bash
aws sts get-caller-identity
```

正常にログインできている場合、以下のような情報が表示されます：

```json
{
    "UserId": "AROAXXXXXXXXXXXXXX:username@company.com",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/RoleName/username@company.com"
}
```

## トラブルシューティング

### よくある問題と解決方法

#### 1. ブラウザが開かない場合

```bash
# 手動でブラウザを開く場合のURL確認
aws sso login --no-browser
```

表示されたURLを手動でブラウザで開いてください。

#### 2. セッションが期限切れの場合

```bash
# 再ログイン
aws sso login
```

#### 4. プロキシ環境での設定

プロキシ環境の場合、以下の環境変数を設定してください：

```bash
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
export NO_PROXY=localhost,127.0.0.1,.company.com
```

## セキュリティのベストプラクティス

1. **定期的な認証情報の更新**: SSOセッションには有効期限があります。定期的に再ログインを行ってください。

2. **最小権限の原則**: 必要最小限の権限を持つロールを使用してください。

3. **プロファイルの分離**: 本番環境と開発環境で異なるプロファイルを使用してください。

4. **ログアウト**: 作業終了時は適切にログアウトしてください：
   ```bash
   aws sso logout --profile <プロファイル名>
   ```

## 参考リンク

- [AWS CLI ユーザーガイド](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS SSO ユーザーガイド](https://docs.aws.amazon.com/singlesignon/latest/userguide/)
- [AWS CLI インストールガイド](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
