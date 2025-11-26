# Deploy Generator Skill

インフラ構成を検出し、適切なデプロイ設定を自動生成するSkill

## 概要

プロジェクトのインフラ構成（Terraform、package.json、設定ファイル等）を解析し、CI/CDワークフローとデプロイスクリプトを自動生成します。

## 使い方

### 前提条件

- アーキテクチャ設計が完了している（`/design-architecture` 実行済み）
- インフラ構成が決定している

### 実行

```bash
# AI-DLCテンプレートのスラッシュコマンドから
/generate-deploy unit1

# または、Skillを直接呼び出し
```

### 生成されるファイル

```
.github/
└── workflows/
    └── deploy.yml          # GitHub Actions ワークフロー

scripts/
└── deploy.sh               # デプロイスクリプト

docs/deploy/
└── README.md               # デプロイ手順書
```

## 対応インフラパターン

### 1. S3 + CloudFront（静的サイト）

**検出条件**:
- `terraform/modules/` に S3/CloudFront リソースがある
- `next.config.js` に `output: 'export'` がある
- Vite/React のビルド設定

**生成内容**:
- S3へのビルド成果物アップロード
- CloudFrontキャッシュ無効化
- 環境変数の注入

### 2. Vercel

**検出条件**:
- `vercel.json` が存在
- Next.js プロジェクト

**生成内容**:
- Vercel CLI でのデプロイ
- Preview/Production環境の分岐
- 環境変数設定ガイド

### 3. AWS Lambda（サーバーレス）

**検出条件**:
- `terraform/modules/` に Lambda リソースがある
- `serverless.yml` が存在

**生成内容**:
- Lambda 関数のデプロイ
- API Gateway 設定
- 環境変数・シークレット管理

### 4. ECS/Fargate

**検出条件**:
- `Dockerfile` が存在
- `terraform/modules/` に ECS リソースがある

**生成内容**:
- Docker イメージビルド・プッシュ
- ECS サービス更新
- ヘルスチェック待機

### 5. Amplify

**検出条件**:
- `amplify/` ディレクトリが存在
- `amplify.yml` が存在

**生成内容**:
- Amplify Console連携
- ブランチベースデプロイ

## 環境変数の扱い

### 検出と提案

Skillは以下を検出し、適切な管理方法を提案します：

| 種類 | 例 | 推奨管理方法 |
|-----|---|-------------|
| 機密情報 | API_KEY, DB_PASSWORD | GitHub Secrets, AWS Secrets Manager |
| 環境依存 | API_URL, STAGE | GitHub Environment Variables |
| ビルド時 | NEXT_PUBLIC_* | CI/CDで注入 |

### 環境別設定

```yaml
# .github/workflows/deploy.yml 例
jobs:
  deploy:
    environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
    env:
      API_URL: ${{ vars.API_URL }}
      API_KEY: ${{ secrets.API_KEY }}
```

## 使用例

### 基本的な使い方

```bash
/generate-deploy unit1
```

### 特定のインフラパターンを指定

```bash
/generate-deploy unit1 --target s3-cloudfront
/generate-deploy unit1 --target vercel
/generate-deploy unit1 --target lambda
```

## 生成されるワークフロー例

### S3 + CloudFront

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Build
        run: pnpm build
        env:
          NEXT_PUBLIC_API_URL: ${{ vars.API_URL }}

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - name: Deploy to S3
        run: aws s3 sync ./out s3://${{ vars.S3_BUCKET }} --delete

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ vars.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

## デプロイ前チェックリスト

生成されるドキュメントには、以下のチェックリストが含まれます：

```markdown
## デプロイ前チェックリスト

### 初回セットアップ
- [ ] GitHub Secrets の設定
  - [ ] AWS_ACCESS_KEY_ID
  - [ ] AWS_SECRET_ACCESS_KEY
- [ ] GitHub Environment Variables の設定
  - [ ] API_URL (環境ごと)
  - [ ] S3_BUCKET (環境ごと)
- [ ] IAMロールの権限確認

### 毎回のデプロイ
- [ ] テストが全て通過
- [ ] ビルドが成功
- [ ] 環境変数の変更がないか確認
```

## 他プロジェクトでの利用

このSkillは、AI-DLCテンプレート以外のプロジェクトでも利用可能です：

1. `.claude/skills/deploy-generator/` をコピー
2. プロジェクトに配置
3. Skillを呼び出し

## バージョン履歴

- **1.0.0** (2025-11-26): 初版リリース
  - S3+CloudFront、Vercel、Lambda、ECS、Amplify対応
  - 環境変数検出・提案機能
  - デプロイ前チェックリスト生成

## ライセンス

MIT
