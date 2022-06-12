# 前準備

## clone

```bash
git clone https://github.com/ng3rdstmadgke/snap-station.git

cd snap-station
```

## slsインストール

```bash
# nvm install
# https://github.com/nvm-sh/nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

# node.jsの最新のltsをインストール
nvm install --lts
nvm use --lts
node -v
npm -v

# npm update
npm update -g npm

# serverless install
npm install
```

## frontビルド

```bash
(cd front && npm install && npm run generate)
```

## pythonライブラリインストール

```bash
python3.8 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

# 単体テスト

※ テストの実行にはfrontのビルドが必要

```bash
# 開発用イメージビルド
./bin/build.sh

# 開発用データベース起動
./bin/run-mysql.sh -d

# テスト
./bin/test.sh
```

# 開発環境

```bash
# 開発用イメージビルド
./bin/build.sh

# 開発用データベース起動
./bin/run-mysql.sh -d

# オペレーションshell起動
./bin/shell.sh -e local.env

# マイグレーション実行 (オペレーションshell内での操作)
app@ip-* $ ./bin/lib/create-database.sh
app@ip-* $ ./bin/lib/alembic.sh upgrade head

# 初期データ登録 (オペレーションshell内での操作)
# スーパーユーザー作成
app@ip-* $ ./bin/lib/manage.sh create_user admin --superuser
# 通常ユーザー作成
app@ip-* $ ./bin/lib/manage.sh create_user user1
app@ip-* $ ./bin/lib/manage.sh create_user user2
# ロール作成
app@ip-* $ ./bin/lib/manage.sh create_role ItemAdminRole
# ロールのアタッチ
app@ip-* $ ./bin/lib/manage.sh attach_role user1 ItemAdminRole

# オペレーションshellからログアウト
app@ip-* $ exit

# アプリ起動
./bin/run.sh -e local.env

# アクセス
# http://localhost:3000/
# http://localhost:8000/api/docs
```

# 本番環境デプロイ

```bash
STAGE_NAME=mi1

# 環境変数ファイル作成 (オペレーションshellログイン用 (本番環境では利用しない))
cp local.env ${STAGE_NAME}.env
vim ${STAGE_NAME}.env

# オペレーションshell起動
./bin/shell.sh -e ${STAGE_NAME}.env

# マイグレーション実行 (オペレーションshell内での操作)
app@ip-* $ ./bin/lib/create-database.sh
app@ip-* $ ./bin/lib/alembic.sh upgrade head

# 初期データ登録 (オペレーションshell内での操作)
# スーパーユーザー作成
app@ip-* $ ./bin/lib/manage.sh create_user admin --superuser
# 通常ユーザー作成
app@ip-* $ ./bin/lib/manage.sh create_user user1
app@ip-* $ ./bin/lib/manage.sh create_user user2
# ロール作成
app@ip-* $ ./bin/lib/manage.sh create_role ItemAdminRole
# ロールのアタッチ
app@ip-* $ ./bin/lib/manage.sh attach_role user1 ItemAdminRole

# オペレーションshellからログアウト
app@ip-* $ exit

# プロファイル作成
cp ./profile/sample.yml ./profile/${STAGE_NAME}.yml
vim ./profile/${STAGE_NAME}.yml

# lambda用イメージのビルド
./bin/build-lambda.sh -s ${STAGE_NAME}

# ECRにリポジトリを作成 (TODO: IaC化)
# snap-station/lambda/${STAGE_NAME}

# ECRにイメージをアップロード
./bin/push-lambda.sh -s ${STAGE_NAME}

# デプロイ
./bin/deploy.sh --stage mi1

# 削除
sls remove --stage ${STAGE_NAME}
```


# オペレーションshell

## オペレーションshell起動

```bash
# 開発用データベース起動
./bin/run-mysql.sh -d

# オペレーション用shell起動
./bin/shell.sh -e <ENV_PATH>

```

## マイグレーション(オペレーションshell内での操作)

```bash
# DB作成
./bin/lib/create-database.sh

# マイグレーション: 履歴確認
./bin/lib/alembic.sh history -v

# マイグレーション: 最新バージョンにアップグレード
./bin/lib/alembic.sh upgrade head

# マイグレーション: 次のバージョンにアップグレード
./bin/lib/alembic.sh upgrade +1

# マイグレーション: 最初のバージョンにダウングレード
./bin/lib/alembic.sh downgrade base

# マイグレーション: 次のバージョンにダウングレード
./bin/lib/alembic.sh downgrade -1

# マイグレーション: マイグレーションファイル生成
./bin/lib/alembic.sh revision --autogenerate -m "message"
```

## DBログイン(オペレーションshell内での操作)

```bash
# DB作成
./bin/lib/create-database.sh

# mysql ログイン
./bin/lib/mysql.sh
```

## マネジメントコマンド(オペレーションshell内での操作)

```bash
# ヘルプ
./bin/lib/manage.sh

# スーパーユーザー作成
./bin/lib/manage.sh create_user admin --superuser

# 通常ユーザー作成
./bin/lib/manage.sh create_user user1
./bin/lib/manage.sh create_user user2

# ロール作成
./bin/lib/manage.sh create_role ItemAdminRole

# ロールのアタッチ
./bin/lib/manage.sh attach_role user1 ItemAdminRole

# ロールのデタッチ
./bin/lib/manage.sh detach_role user1 ItemAdminRole
```