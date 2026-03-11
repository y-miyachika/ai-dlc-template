---
name: iac-generator
description: "Terraform/Terragrunt IaC生成。アーキテクチャ設計からセキュアで再利用可能なTerraformモジュールと環境別設定を自動生成する。引数: ユニット名（例: unit1）"
---

# IaC Generator Skill

あなたはインフラストラクチャ設計とTerraform/Terragruntの専門家です。アーキテクチャ設計からセキュアで再利用可能なTerraformモジュールを自動生成します。

## 目的

`/design-architecture`で設計されたアーキテクチャをTerraform/Terragruntモジュールとして実装し、unit単位で管理可能なIaCを生成します。

**技術スタック**: Terraform + Terragrunt + AWS Provider

---

## 設定

出力パスは `.claude/skill-config.json` でカスタマイズ可能です。
詳細は [Skill共通設定ガイド](../config.md) を参照してください。

**デフォルト設定**:
```json
{
  "iac-generator": {
    "outputDir": "terraform",
    "modulesDir": "modules",
    "environmentsDir": "environments"
  }
}
```

設定ファイルが存在しない場合は、プロジェクト構成を自動判定します（ステップ0参照）。

---

## 入力

### 必須
- **ユニット名**: 対象のユニット（例: `unit1`, `001-unit1`）
- **アーキテクチャ設計**: `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md`

### オプション
- **NFR定義**: パフォーマンス、セキュリティ、可用性要件
- **出力先**: 設定ファイル → 自動判定の優先順位で決定

---

## 処理フロー

### ステップ0: プロジェクト構成の判定

**判定ロジック**:
1. `packages/` 存在 → **モノレポ** → `packages/infrastructure/`
2. それ以外 → **アプリ単体** → `infrastructure/`

---

### ステップ1: アーキテクチャ設計の読み込み

architecture.mdから以下を抽出：
- インフラコンポーネント（RDS, S3, Lambda, VPC等）
- NFR（パフォーマンス、セキュリティ、可用性）
- データフロー、unit間依存関係、環境要件

---

### ステップ2: Terraformモジュール設計

unit単位でモジュール化。unit間の依存は `module.unit1.vpc_id` のようにoutput参照で解決。

### 環境別デフォルト設定

```hcl
variable "enable_xray" {
  type    = bool
  default = false
}

variable "enable_pitr" {
  type    = bool
  default = false
}

variable "enable_multi_az" {
  type    = bool
  default = false
}

variable "backup_retention_days" {
  type    = number
  default = 0
}
```

| 環境 | X-Ray | PITR | Multi-AZ | バックアップ |
|-----|-------|------|----------|------------|
| dev | 無効 | 無効 | 無効 | 無効 |
| staging | 無効 | 有効 | 無効 | 3日 |
| production | 有効 | 有効 | 有効 | 7日 |

---

### ステップ3〜9: 生成

詳細テンプレートは `README.md` を参照。

**出力**:
- `{root}/terraform/modules/{unit}/` — main.tf, variables.tf, outputs.tf, versions.tf, README.md
- `{root}/environments/{env}/` — main.tf, variables.tf, outputs.tf, terraform.tfvars.example, versions.tf
- `{root}/docs/{unit}_IaC設計.md`
- `{root}/.gitignore`
- `packages/infrastructure/package.json`（モノレポのみ）

---

## セキュリティベストプラクティス

1. **暗号化**: S3デフォルト暗号化、RDS storage_encrypted、EBS encrypted
2. **バックアップ**: 本番7日保持、dev無効、S3バージョニング
3. **IAM最小権限**: Lambda実行ロール最小限
4. **ネットワーク**: Security Group最小限ポート開放
5. **タグ戦略**: Environment, Project, Unit タグ

---

## エラーハンドリング

- アーキテクチャ設計未作成 → エラー（`/design-architecture`を先に実行）
- 既存モジュール → 確認してから上書き
- 既存環境設定 → 新しいmodule呼び出しを追加
