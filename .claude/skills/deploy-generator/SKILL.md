---
name: deploy-generator
description: "デプロイ設定生成。インフラ構成を検出し、GitHub Actionsワークフローとデプロイスクリプトを自動生成する。引数: ユニット名（例: unit1）"
---

# Deploy Generator Skill

あなたは**デプロイ設定の専門家**として、プロジェクトのインフラ構成を検出し、適切なCI/CDワークフローとデプロイスクリプトを生成するSkillです。

## 設定

出力パスは `.claude/skill-config.json` でカスタマイズ可能です。

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

ユニット名または対象パッケージ名（例: `unit1`, `web`, `api`）

---

## ステップ1: インフラ構成の検出

| 優先度 | 検出条件 | インフラパターン |
|-------|---------|----------------|
| 1 | `vercel.json` | Vercel |
| 2 | `amplify/` | AWS Amplify |
| 3 | `serverless.yml` | Serverless Framework |
| 4 | Terraform ECS + Dockerfile | ECS/Fargate |
| 5 | Terraform Lambda | AWS Lambda |
| 6 | Terraform S3/CloudFront + フロントエンド | S3 + CloudFront |
| 7 | Next.js（output: 'export' なし） | Next.js (Node.js) |
| 8 | Vite/React | 静的サイト |

---

## ステップ2: 環境変数の分析

| 種類 | 機密性 | 推奨管理方法 |
|-----|--------|-------------|
| マシンシークレット | 高 | GitHub Secrets / AWS Secrets Manager |
| 環境依存 | 低 | GitHub Environment Variables |
| ビルド時 | 低 | CI/CD injection |

---

## ステップ3: GitHub Actions ワークフロー生成

パターン別デプロイステップ（S3+CloudFront, Vercel, Lambda, ECS/Fargate, Amplify）を含むワークフローを生成。

---

## ステップ4: デプロイスクリプト生成

`scripts/deploy.sh` をローカルデプロイ用に生成。

---

## ステップ5: デプロイドキュメント生成

`docs/deploy/README.md` にセットアップ手順、環境変数一覧、トラブルシューティングを生成。

---

## ステップ6: ユーザー確認

検出結果を報告後、以下を確認してからファイル生成：
- Staging環境の要否
- ブランチ戦略
- 追加の環境変数

---

## 出力ファイル

1. `.github/workflows/deploy.yml`
2. `scripts/deploy.sh`
3. `docs/deploy/README.md`

---

## 完了条件

1. インフラパターンが正しく検出されている
2. 環境変数が分類・整理されている
3. 全出力ファイルが生成されている
4. ユーザーの承認を得ている
