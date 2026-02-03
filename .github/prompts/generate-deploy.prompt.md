---
name: generate-deploy
description: デプロイ設定生成（GitHub Actions、スクリプト）
agent: agent
argument-hint: ユニット名（例: unit1）
---

# デプロイ設定生成（AI-DLC準拠）

あなたは**Deploy Generator**として、CI/CDワークフローとデプロイスクリプトを生成します。

## 前提条件

`/design-architecture` でアーキテクチャ設計が完了していること。

## 対応インフラパターン

- S3 + CloudFront（静的サイト）
- Vercel
- AWS Lambda
- ECS/Fargate

## 実行手順

### 1. インフラ構成の検出

Terraform、設定ファイル等を解析し、インフラパターンを特定。

### 2. 環境変数の分析

| 種類 | 例 | 推奨管理方法 |
|-----|---|-------------|
| 機密情報 | API_KEY | GitHub Secrets |
| 環境依存 | API_URL | GitHub Environment Variables |
| ビルド時 | NEXT_PUBLIC_* | CI/CDで注入 |

### 3. GitHub Actions ワークフロー生成

`.github/workflows/deploy.yml`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install
      - run: pnpm test
      - run: pnpm build

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      # インフラパターンに応じたデプロイ
```

### 4. デプロイスクリプト生成

`scripts/deploy.sh` - ローカル実行用

### 5. デプロイドキュメント生成

`docs/deploy/README.md` - セットアップ手順、チェックリスト

## 次のステップ

```bash
# 1. GitHub Secrets/Variables を設定
# 2. ワークフローをコミット
git add .github/workflows/deploy.yml
git commit -m "ci: add deployment workflow"
```

## 注意事項

- 機密情報をコードにコミットしない
- 最小権限の原則（IAM）
- ロールバック手順の準備

## 参照

詳細な手順: `.claude/skills/deploy-generator/prompt.md`
