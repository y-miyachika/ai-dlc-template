# デプロイ設定生成（AI-DLC準拠）

このコマンドは**Deploy Generator Skill**を使用して、プロジェクトのインフラ構成に基づいたデプロイ設定を自動生成します。

## Skillについて

`deploy-generator` Skillは、プロジェクトのインフラ構成を検出し、適切なCI/CDワークフローとデプロイスクリプトを生成する専門Skillです。

**対応インフラパターン**: S3+CloudFront、Vercel、Lambda、ECS/Fargate、Amplify

詳細は `.claude/skills/deploy-generator/README.md` を参照してください。

## 前提条件

**このコマンドは `/design-architecture` の後に実行してください**

- `/design-architecture` でアーキテクチャ設計が完了している
- インフラ構成（Terraform、package.json等）が存在する

## 入力内容

{{ARGS}}

## Skill起動

### EnterPlanMode統合

デプロイ設定生成の前に **`EnterPlanMode` で Plan Mode に入り、デプロイ戦略を計画する**。

**Plan Mode内で実行する内容：**

1. **インフラ構成の検出**（Glob/Grep/Read）
   - Terraform設定、Dockerfile、package.json等を解析
   - デプロイ対象のインフラパターンを判定（S3+CloudFront/Vercel/Lambda/ECS等）
   - 既存のGitHub Actionsワークフローがあれば確認
2. **環境変数の分析**
   - 機密情報と公開情報を分類
   - GitHub Secrets/Variablesへの配置計画
3. **デプロイ戦略の提示**
   - 検出されたインフラパターンに基づくCI/CD構成
   - 生成されるファイル一覧（新規/上書き）
   - 環境分岐（dev/staging/production）の方針

**`ExitPlanMode` でデプロイ戦略の承認を得る** → 承認後にコード生成を実行。

---

**実行する処理**:

承認後、引数として受け取ったユニット名をもとに、Deploy Generator Skillを起動します。

**ユニット名**: {{ARGS}}

**アーキテクチャ設計パス**: `docs/design-artifacts/architecture/`

**出力先**:
- `.github/workflows/deploy.yml`
- `scripts/deploy.sh`
- `docs/deploy/README.md`

---

## 対応インフラパターン

### S3 + CloudFront（静的サイト）

**検出条件**:
- Terraform に S3/CloudFront リソース
- Next.js `output: 'export'` または Vite/React

**生成内容**:
- S3 sync によるアップロード
- CloudFront キャッシュ無効化

### Vercel

**検出条件**:
- `vercel.json` が存在
- Next.js プロジェクト

**生成内容**:
- Vercel CLI によるデプロイ
- Preview/Production 環境分岐

### AWS Lambda

**検出条件**:
- Terraform に Lambda リソース
- `serverless.yml` が存在

**生成内容**:
- Lambda 関数のパッケージング
- 関数コードの更新

### ECS/Fargate

**検出条件**:
- `Dockerfile` が存在
- Terraform に ECS リソース

**生成内容**:
- Docker イメージビルド・プッシュ
- ECS サービス更新

---

## 環境変数の扱い

### 分類

| 種類 | 例 | 推奨管理方法 |
|-----|---|-------------|
| 機密情報 | API_KEY, DB_PASSWORD | GitHub Secrets |
| 環境依存 | API_URL, STAGE | GitHub Environment Variables |
| ビルド時 | NEXT_PUBLIC_* | CI/CDで注入 |

### フロントエンド特有の注意

⚠️ フロントエンド環境変数は**ビルド時**に埋め込まれます：
- 本番ビルドには本番環境の変数が必要
- CI/CDで適切に注入すること
- 機密情報はバックエンド経由で取得

---

## 生成されるファイル

### GitHub Actions ワークフロー

- `.github/workflows/deploy.yml` - デプロイワークフロー

**構成**:
```yaml
jobs:
  build:     # ビルド・テスト
  deploy:    # デプロイ（mainブランチのみ）
```

### デプロイスクリプト

- `scripts/deploy.sh` - ローカルデプロイ用

### デプロイドキュメント

- `docs/deploy/README.md` - セットアップ手順、チェックリスト

---

## 実行後の次のステップ

```bash
# 1. GitHub Secrets/Variables を設定
# 2. ワークフローをコミット
git add .github/workflows/deploy.yml
git commit -m "ci: add deployment workflow"

# 3. mainブランチにマージでデプロイ実行
```

---

## 注意事項

1. **機密情報をコードにコミットしない**: GitHub Secrets を使用
2. **最小権限の原則**: デプロイ用IAMには必要最小限の権限
3. **環境の分離**: Production/Staging で異なる認証情報を使用
4. **ロールバック手順の準備**: 問題発生時の切り戻し手順を確認

---

## AI-DLC原則との対応

| 原則 | 実装方法 |
|-----|---------|
| **自動化** | CI/CDパイプラインによる自動デプロイ |
| **トレーサビリティ** | アーキテクチャ設計 → デプロイ設定 |
| **段階的詳細化** | インフラ設計 → CI/CD設定 → 運用 |

---

**Skill Version**: 1.0.0
**Skill Location**: `.claude/skills/deploy-generator/`
**作成日**: 2025-11-26
**AI-DLC準拠**: オペレーションフェーズ（デプロイ・運用）
