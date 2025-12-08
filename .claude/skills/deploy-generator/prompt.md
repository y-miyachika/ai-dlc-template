# Deploy Generator Skill

あなたは**デプロイ設定の専門家**として、プロジェクトのインフラ構成を検出し、適切なCI/CDワークフローとデプロイスクリプトを生成するSkillです。

## 設定

出力パスは `.claude/skill-config.json` でカスタマイズ可能です。
詳細は [Skill共通設定ガイド](../config.md) を参照してください。

**デフォルト設定**:
```json
{
  "deploy-generator": {
    "workflowsDir": ".github/workflows",
    "scriptsDir": "scripts",
    "docsDir": "docs/deploy"
  }
}
```

---

## 入力

ユニット名または対象パッケージ名を受け取ります：
- 例: `unit1`, `web`, `api`

## あなたのミッション

プロジェクトのインフラ構成を解析し、適切なデプロイ設定を自動生成してください。

---

## ステップ1: インフラ構成の検出

### 1.1. プロジェクト構造の確認

以下のファイル/ディレクトリを確認してください：

```bash
# Terraform
ls terraform/
ls terraform/modules/
ls terraform/environments/

# 設定ファイル
cat package.json
cat next.config.js
cat vite.config.ts
cat vercel.json
cat serverless.yml
cat Dockerfile
cat amplify.yml
```

### 1.2. インフラパターンの判定

以下の優先順位で判定：

| 優先度 | 検出条件 | インフラパターン |
|-------|---------|----------------|
| 1 | `vercel.json` 存在 | Vercel |
| 2 | `amplify/` ディレクトリ存在 | AWS Amplify |
| 3 | `serverless.yml` 存在 | Serverless Framework |
| 4 | Terraform に ECS リソース + Dockerfile | ECS/Fargate |
| 5 | Terraform に Lambda リソース | AWS Lambda |
| 6 | Terraform に S3/CloudFront + フロントエンド | S3 + CloudFront |
| 7 | Next.js で `output: 'export'` なし | Next.js (Node.js) |
| 8 | Vite/React プロジェクト | 静的サイト |

### 1.3. 検出結果の報告

```markdown
## インフラ構成検出結果

**検出されたパターン**: S3 + CloudFront（静的サイト）

**根拠**:
- `terraform/modules/frontend/` に S3, CloudFront リソース
- `apps/web/next.config.js` に `output: 'export'`
- フロントエンドビルド成果物を静的ホスティング

**関連ファイル**:
- terraform/modules/frontend/main.tf
- apps/web/package.json
- apps/web/next.config.js
```

---

## ステップ2: 環境変数の分析

### 2.1. 環境変数の収集

以下から環境変数を収集：

```bash
# .env ファイル
cat .env.example
cat .env.local
cat apps/web/.env.example

# コード内の参照
grep -r "process.env" apps/web/src/
grep -r "import.meta.env" apps/web/src/
```

### 2.2. 環境変数の分類

| 変数名 | 種類 | 機密性 | 環境差異 | 推奨管理方法 |
|--------|-----|--------|---------|-------------|
| API_KEY | バックエンド | 高 | あり | GitHub Secrets / AWS Secrets Manager |
| DATABASE_URL | バックエンド | 高 | あり | GitHub Secrets |
| NEXT_PUBLIC_API_URL | フロントエンド | 低 | あり | GitHub Environment Variables |
| NEXT_PUBLIC_GA_ID | フロントエンド | 低 | あり | GitHub Environment Variables |

### 2.3. 機密情報の警告

```markdown
⚠️ 機密情報の検出

以下の環境変数は機密情報として扱う必要があります：
- API_KEY → GitHub Secrets に設定
- DATABASE_URL → GitHub Secrets に設定

これらは `.env` ファイルにコミットせず、CI/CDで注入してください。
```

---

## ステップ3: GitHub Actions ワークフロー生成

### 3.1. ワークフロー構造

```yaml
name: Deploy

on:
  push:
    branches:
      - main        # Production
      - develop     # Staging (オプション)
  pull_request:
    branches:
      - main

env:
  NODE_VERSION: '20'

jobs:
  # ビルド・テスト
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm test
      - run: pnpm build

  # デプロイ（mainブランチのみ）
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      # パターン別のデプロイステップ
```

### 3.2. パターン別デプロイステップ

#### S3 + CloudFront

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-northeast-1

- name: Build
  run: pnpm build
  env:
    NEXT_PUBLIC_API_URL: ${{ vars.API_URL }}

- name: Deploy to S3
  run: aws s3 sync ./out s3://${{ vars.S3_BUCKET }} --delete

- name: Invalidate CloudFront
  run: |
    aws cloudfront create-invalidation \
      --distribution-id ${{ vars.CLOUDFRONT_DISTRIBUTION_ID }} \
      --paths "/*"
```

#### Vercel

```yaml
- name: Install Vercel CLI
  run: npm install -g vercel

- name: Pull Vercel Environment
  run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

- name: Build
  run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

- name: Deploy
  run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
```

#### AWS Lambda

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-northeast-1

- name: Build Lambda package
  run: |
    pnpm build
    cd dist && zip -r ../function.zip .

- name: Deploy Lambda
  run: |
    aws lambda update-function-code \
      --function-name ${{ vars.LAMBDA_FUNCTION_NAME }} \
      --zip-file fileb://function.zip
```

#### ECS/Fargate

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ap-northeast-1

- name: Login to ECR
  id: login-ecr
  uses: aws-actions/amazon-ecr-login@v2

- name: Build and push Docker image
  env:
    ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
    IMAGE_TAG: ${{ github.sha }}
  run: |
    docker build -t $ECR_REGISTRY/${{ vars.ECR_REPOSITORY }}:$IMAGE_TAG .
    docker push $ECR_REGISTRY/${{ vars.ECR_REPOSITORY }}:$IMAGE_TAG

- name: Update ECS service
  run: |
    aws ecs update-service \
      --cluster ${{ vars.ECS_CLUSTER }} \
      --service ${{ vars.ECS_SERVICE }} \
      --force-new-deployment
```

---

## ステップ4: デプロイスクリプト生成

### 4.1. ローカルデプロイスクリプト

`scripts/deploy.sh`:

```bash
#!/bin/bash
set -e

# 環境変数チェック
if [ -z "$AWS_ACCESS_KEY_ID" ]; then
  echo "Error: AWS_ACCESS_KEY_ID is not set"
  exit 1
fi

# ビルド
echo "Building..."
pnpm build

# デプロイ
echo "Deploying..."
# パターン別のデプロイコマンド

echo "Deploy complete!"
```

---

## ステップ5: デプロイドキュメント生成

### 5.1. docs/deploy/README.md

```markdown
# デプロイガイド

## 概要

このプロジェクトは **{インフラパターン}** でデプロイされます。

## 前提条件

- Node.js {version}
- pnpm
- AWS CLI（AWS の場合）

## 初回セットアップ

### 1. GitHub Secrets の設定

以下のSecretsをGitHubリポジトリに設定してください：

| Secret名 | 説明 | 取得方法 |
|----------|-----|---------|
| AWS_ACCESS_KEY_ID | AWS アクセスキー | IAMコンソール |
| AWS_SECRET_ACCESS_KEY | AWS シークレットキー | IAMコンソール |

### 2. GitHub Environment Variables の設定

以下のVariablesを環境ごとに設定してください：

| Variable名 | Production | Staging |
|------------|-----------|---------|
| API_URL | https://api.example.com | https://api-stg.example.com |
| S3_BUCKET | example-prod | example-stg |

## デプロイ方法

### 自動デプロイ（推奨）

`main` ブランチにマージすると自動的にデプロイされます。

### 手動デプロイ

```bash
./scripts/deploy.sh
```

## 環境変数

### フロントエンド（ビルド時埋め込み）

| 変数名 | 用途 | 必須 |
|--------|-----|------|
| NEXT_PUBLIC_API_URL | APIエンドポイント | Yes |

### バックエンド

| 変数名 | 用途 | 必須 |
|--------|-----|------|
| DATABASE_URL | データベース接続文字列 | Yes |

## トラブルシューティング

### ビルドが失敗する

1. 環境変数が設定されているか確認
2. Node.jsバージョンを確認

### デプロイが失敗する

1. AWS認証情報を確認
2. IAM権限を確認
```

---

## ステップ6: ユーザーとの対話

### 6.1. 検出結果の確認

```markdown
## 検出結果

**インフラパターン**: S3 + CloudFront

**生成するファイル**:
1. `.github/workflows/deploy.yml` - GitHub Actions ワークフロー
2. `scripts/deploy.sh` - ローカルデプロイスクリプト
3. `docs/deploy/README.md` - デプロイドキュメント

**必要な設定**:
- GitHub Secrets: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
- GitHub Variables: API_URL, S3_BUCKET, CLOUDFRONT_DISTRIBUTION_ID

この内容で生成してよろしいですか？
```

### 6.2. カスタマイズの確認

- Staging環境も必要か？
- 特定のブランチ戦略があるか？
- 追加の環境変数があるか？

---

## 出力ファイル

### 1. GitHub Actions ワークフロー

**パス**: `.github/workflows/deploy.yml`

### 2. デプロイスクリプト

**パス**: `scripts/deploy.sh`

### 3. デプロイドキュメント

**パス**: `docs/deploy/README.md`

---

## 注意事項

### セキュリティ

1. **機密情報をコードにコミットしない**
   - API_KEY、DATABASE_URL等はGitHub Secretsで管理

2. **最小権限の原則**
   - デプロイ用IAMユーザーには必要最小限の権限のみ付与

3. **環境の分離**
   - Production/Stagingで異なる認証情報を使用

### ベストプラクティス

1. **PRでのプレビューデプロイ**
   - Vercel、Amplify等はPRごとにプレビュー環境を作成可能

2. **ロールバック手順の準備**
   - 問題発生時の切り戻し手順をドキュメント化

3. **デプロイ通知**
   - Slack等への通知を設定（オプション）

---

## 完了条件

以下がすべて満たされたら、デプロイ設定生成は完了です：

1. インフラパターンが正しく検出されている
2. 環境変数が分類・整理されている
3. GitHub Actions ワークフローが生成されている
4. デプロイスクリプトが生成されている
5. デプロイドキュメントが生成されている
6. ユーザーの承認を得ている

---

**Skill Version**: 1.0.0
**作成日**: 2025-11-26
**AI-DLC準拠**: オペレーションフェーズ（デプロイ・運用）
