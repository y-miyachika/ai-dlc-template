---
applyTo: "**/*.tf,**/*.tfvars"
---

# Terraform コーディング規約

このファイルは Terraform ファイル（`.tf`, `.tfvars`）に適用されるインストラクションです。

## 基本原則

### モジュール単位の設計

- unit 単位でモジュールを作成
- 再利用可能なモジュール設計
- 環境ごとの設定は `environments/` で管理

### ファイル構成

```
terraform/
├── modules/
│   └── {unit}/
│       ├── main.tf          # メインリソース定義
│       ├── variables.tf     # 入力変数
│       ├── outputs.tf       # 出力値
│       ├── versions.tf      # プロバイダー要件
│       └── README.md        # モジュールドキュメント
└── environments/
    ├── dev/
    ├── staging/
    └── production/
```

## リソース命名規則

### 一貫した命名パターン

```hcl
# パターン: {project}-{environment}-{unit}-{resource_type}
resource "aws_s3_bucket" "data" {
  bucket = "${var.project_name}-${var.environment}-${local.unit_name}-data"
}
```

### ローカル変数での一元管理

```hcl
locals {
  unit_name = "user-management"

  name_prefix = "${var.project_name}-${var.environment}-${local.unit_name}"

  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    Unit        = local.unit_name
    ManagedBy   = "Terraform"
  }
}
```

## 変数定義

### 必須変数

```hcl
variable "environment" {
  description = "Environment name (dev, staging, production)"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "project_name" {
  description = "Project name for resource naming"
  type        = string
}
```

### 環境別デフォルト値

```hcl
variable "enable_xray" {
  description = "Enable AWS X-Ray tracing"
  type        = bool
  default     = false  # dev/staging では無効
}

variable "enable_pitr" {
  description = "Enable Point-in-Time Recovery (DynamoDB)"
  type        = bool
  default     = false
}

variable "enable_multi_az" {
  description = "Enable Multi-AZ deployment (RDS)"
  type        = bool
  default     = false
}

variable "backup_retention_days" {
  description = "Number of days to retain backups"
  type        = number
  default     = 0  # 0 = バックアップ無効
}
```

## セキュリティベストプラクティス

### 暗号化

```hcl
# S3 バケット暗号化
resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# RDS 暗号化
resource "aws_db_instance" "main" {
  storage_encrypted = true
  # ...
}
```

### パブリックアクセスのブロック

```hcl
resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### IAM 最小権限

```hcl
resource "aws_iam_role_policy" "lambda" {
  name = "${local.name_prefix}-lambda-policy"
  role = aws_iam_role.lambda.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:UpdateItem",
          "dynamodb:DeleteItem"
        ]
        Resource = aws_dynamodb_table.main.arn
      }
    ]
  })
}
```

## タグ戦略

### すべてのリソースにタグ付け

```hcl
resource "aws_s3_bucket" "data" {
  bucket = "${local.name_prefix}-data"

  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-data"
    }
  )
}
```

### 必須タグ

| タグ名 | 説明 | 例 |
|-------|------|-----|
| `Project` | プロジェクト名 | `my-app` |
| `Environment` | 環境名 | `dev`, `staging`, `production` |
| `Unit` | ユニット名 | `user-management` |
| `ManagedBy` | 管理方法 | `Terraform` |

## 出力値

### 必要な出力を定義

```hcl
output "s3_bucket_name" {
  description = "S3 bucket name"
  value       = aws_s3_bucket.data.id
}

output "s3_bucket_arn" {
  description = "S3 bucket ARN"
  value       = aws_s3_bucket.data.arn
}

output "db_endpoint" {
  description = "RDS endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}
```

### 機密情報は `sensitive = true`

```hcl
output "db_password" {
  description = "Database password"
  value       = random_password.db.result
  sensitive   = true
}
```

## 環境設定

### environments/dev/main.tf

```hcl
provider "aws" {
  region = var.aws_region
}

module "unit1" {
  source = "../../modules/unit1"

  environment  = "dev"
  project_name = var.project_name

  # dev 環境のデフォルト
  enable_xray             = false
  enable_pitr             = false
  enable_multi_az         = false
  backup_retention_days   = 0

  tags = local.common_tags
}
```

### environments/production/main.tf

```hcl
module "unit1" {
  source = "../../modules/unit1"

  environment  = "production"
  project_name = var.project_name

  # production 環境の設定
  enable_xray             = true
  enable_pitr             = true
  enable_multi_az         = true
  backup_retention_days   = 7

  tags = local.common_tags
}
```

## バックエンド設定

### S3 バックエンド

```hcl
terraform {
  backend "s3" {
    bucket         = "my-project-terraform-state"
    key            = "dev/terraform.tfstate"
    region         = "ap-northeast-1"
    encrypt        = true
    dynamodb_table = "my-project-terraform-lock"
  }
}
```

## フォーマット

### terraform fmt の使用

```bash
# フォーマット
terraform fmt -recursive

# チェックのみ
terraform fmt -check -recursive
```

## バリデーション

### terraform validate の使用

```bash
terraform init
terraform validate
```

## ドキュメント

### モジュール README.md

```markdown
# {Unit} Terraform Module

## 概要
{モジュールの説明}

## 使用方法

module "{unit}" {
  source = "../../modules/{unit}"

  environment  = "dev"
  project_name = "my-project"
}

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| environment | Environment name | string | - | yes |
| project_name | Project name | string | - | yes |

## Outputs

| Name | Description |
|------|-------------|
| s3_bucket_name | S3 bucket name |
```

## .gitignore

```
# Terraform
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
!*.tfvars.example
.terraform.lock.hcl

# Secrets
*.pem
*.key
```

## Checkov によるセキュリティスキャン

### 実行方法

```bash
checkov -d . --framework terraform
```

### 一般的なチェック項目

- S3 バケットの暗号化
- パブリックアクセスのブロック
- IAM ポリシーの最小権限
- VPC セキュリティグループの制限
