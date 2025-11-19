# IaC Generator Skill

Terraform / Terragrunt ベースの Infrastructure as Code 生成 Skill

## 概要

アーキテクチャ設計から、セキュアで再利用可能なTerraform/Terragruntモジュールを自動生成します。

## 使い方

### 前提条件

- アーキテクチャ設計が完了している（`/design-architecture`実行済み）
- `docs/design-artifacts/architecture/` に設計ドキュメントが存在する
- NFR（非機能要件）が定義されている

### 実行

```bash
# AI-DLCテンプレートのスラッシュコマンドから
/generate-iac unit1

# または、Skillを直接呼び出し
# （他プロジェクトでも使用可能）
```

### 生成されるファイル

#### モノレポの場合

```
packages/
└── infrastructure/
    ├── package.json             # npm scripts（Terraform実行用）
    ├── terraform/
    │   └── modules/
    │       ├── unit1/           # unit単位でモジュール化
    │       │   ├── main.tf
    │       │   ├── variables.tf
    │       │   ├── outputs.tf
    │       │   ├── versions.tf
    │       │   └── README.md
    │       └── unit2/
    │           └── ...
    ├── environments/
    │   ├── dev/
    │   │   ├── main.tf          # 全unitをここで呼び出し
    │   │   ├── variables.tf
    │   │   ├── terraform.tfvars.example
    │   │   └── versions.tf
    │   ├── staging/
    │   └── production/
    └── .gitignore
```

#### アプリ単体の場合

```
infrastructure/
├── terraform/
│   └── modules/
│       ├── unit1/
│       └── unit2/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── production/
└── .gitignore
```

## 技術スタック

- **Terraform**: Infrastructure as Code
- **Terragrunt**: Terraform wrapper（DRY、依存管理）
- **AWS Provider**: AWS リソース管理
- **環境分離**: dev / staging / production

## 特徴

### 1. unit単位のモジュール化

各unitを独立したTerraformモジュールとして生成：

- **再利用性**: 他の環境でも利用可能
- **独立性**: unit単位でデプロイ可能
- **保守性**: 変更の影響範囲が明確

### 2. 環境別デフォルト設定

コスト最適化とセキュリティのバランス：

**POC/開発環境（dev）**:
- X-Ray: **無効** （コスト削減）
- Point-in-Time Recovery: **無効**
- Multi-AZ: **無効**
- バックアップ: **無効**
- インスタンスサイズ: **最小**

**ステージング環境（staging）**:
- X-Ray: **無効**
- Point-in-Time Recovery: **有効**
- Multi-AZ: **無効**
- バックアップ: **3日**

**本番環境（production）**:
- X-Ray: **有効** （パフォーマンス監視）
- Point-in-Time Recovery: **有効**
- Multi-AZ: **有効** （高可用性）
- バックアップ: **7日**

### 3. unit間の依存関係解決

module の output を参照：

```hcl
# environments/dev/main.tf
module "unit1" {
  source = "../../terraform/modules/unit1"
  # ...
}

module "unit2" {
  source = "../../terraform/modules/unit2"

  # unit1のVPCを参照
  vpc_id = module.unit1.vpc_id
}
```

### 4. セキュリティベストプラクティス

- S3暗号化、バージョニング
- IAMロール最小権限
- セキュリティグループ最小化
- RDSバックアップ、暗号化

### 5. Terragruntサポート

DRY原則でインフラコード管理：

```hcl
# environments/dev/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

terraform {
  source = "../../terraform/modules/unit1"
}

inputs = {
  environment = "dev"
  enable_xray = false
}
```

## 出力例

### terraform/modules/unit1/main.tf

```hcl
# S3 Bucket
resource "aws_s3_bucket" "data" {
  bucket = "${var.project_name}-${var.environment}-${local.unit_name}-data"

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-${var.environment}-data"
      Unit = local.unit_name
    }
  )
}

# S3 Bucket Encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

### environments/dev/main.tf

```hcl
terraform {
  required_version = ">= 1.0"

  backend "s3" {
    bucket = "my-project-terraform-state"
    key    = "dev/terraform.tfstate"
    region = "ap-northeast-1"
  }
}

provider "aws" {
  region = "ap-northeast-1"
}

module "unit1" {
  source = "../../terraform/modules/unit1"

  environment  = "dev"
  project_name = "my-project"

  tags = {
    Environment = "dev"
    ManagedBy   = "Terraform"
  }
}
```

## カスタマイズ

### NFRに基づく変数調整

```hcl
# environments/production/terraform.tfvars
db_instance_class = "db.r6g.xlarge"  # NFRから決定
enable_multi_az = true
backup_retention_days = 30
```

### 独自リソースの追加

```hcl
# terraform/modules/unit1/main.tf
resource "aws_cloudwatch_log_group" "app" {
  name              = "/aws/lambda/${var.project_name}-${var.environment}-app"
  retention_in_days = var.log_retention_days
}
```

## 他プロジェクトでの利用

このSkillは、AI-DLCテンプレート以外のプロジェクトでも利用可能です：

1. `.claude/skills/iac-generator/` をコピー
2. プロジェクトに配置
3. Skillを呼び出し

```bash
# 他プロジェクトで
claude skill iac-generator --unit unit1
```

## バージョン履歴

- **1.0.0** (2025-11-19): 初版リリース
  - Terraform/Terragruntモジュール生成
  - 環境別デフォルト設定
  - unit間依存関係解決
  - セキュリティベストプラクティス統合

## ライセンス

MIT
