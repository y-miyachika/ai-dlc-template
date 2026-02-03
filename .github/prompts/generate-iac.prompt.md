---
name: generate-iac
description: Infrastructure as Code生成（Terraform/Terragrunt）
agent: agent
argument-hint: ユニット名（例: unit1）
---

# Infrastructure as Code生成（Terraform）

あなたは**IaC Generator**として、Terraform/Terragruntモジュールを自動生成します。

## 前提条件

`/design-architecture` でアーキテクチャ設計が完了していること。

## 技術スタック

Terraform + Terragrunt + AWS Provider

## 実行手順

### 1. プロジェクト構成の判定

- `packages/` 存在 → モノレポ → `packages/infrastructure/`
- `apps/` 存在 → アプリ単体 → `infrastructure/`

### 2. アーキテクチャ設計の読み込み

`docs/design-artifacts/architecture/` から設計ドキュメントを読み込み。

### 3. Terraformモジュール生成

unit単位でモジュール化：
- `terraform/modules/{unit}/main.tf`
- `terraform/modules/{unit}/variables.tf`
- `terraform/modules/{unit}/outputs.tf`
- `terraform/modules/{unit}/versions.tf`
- `terraform/modules/{unit}/README.md`

### 4. 環境設定生成

dev/staging/production の設定ファイル生成：
- `environments/dev/main.tf` - 全unitをここで呼び出し
- `environments/dev/variables.tf`
- `environments/dev/terraform.tfvars.example`

## 環境別デフォルト設定

**dev（開発）**:
- X-Ray: 無効（コスト削減）
- バックアップ: 無効
- インスタンス: 最小

**production（本番）**:
- X-Ray: 有効（監視）
- Multi-AZ: 有効（高可用性）
- バックアップ: 7日

## 次のステップ

```bash
cd infrastructure/environments/dev
terraform init
terraform plan
terraform apply
```

## 参照

詳細な手順: `.claude/skills/iac-generator/prompt.md`
