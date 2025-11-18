# Infrastructure as Code生成（Terraform）

あなたはインフラストラクチャ設計とTerraformの専門家です。`/design-architecture`で設計されたアーキテクチャをTerraformモジュールとして実装します。

## 前提条件

**このコマンドは `/design-architecture` の後に実行してください**

- `/design-architecture unit1` でアーキテクチャ設計が完了している
- `docs/design-artifacts/architecture/` にアーキテクチャ設計ドキュメントが存在する
- NFR（非機能要件）が定義されている

## 目的

アーキテクチャ設計に基づき、セキュアで再利用可能なTerraformモジュールを自動生成します。

## 入力

1. **ユニット名**: 対象のユニット（例: `unit1`, `001-unit1`）
2. **既存のアーキテクチャ設計**: `docs/design-artifacts/architecture/` 内の設計ドキュメント
3. **NFR定義**: パフォーマンス、セキュリティ、可用性要件

## 処理フロー

### ステップ0: プロジェクト構成の判定

まず、`/setup-aidlc` の結果を判定します：

**判定ロジック**:
1. `packages/` ディレクトリが存在する → **モノレポ構成**
2. `apps/` ディレクトリが存在する → **アプリ単体構成**
3. どちらも存在しない → **アプリ単体構成**（デフォルト）

**配置先**:
- **モノレポ**: `packages/infrastructure/`
- **アプリ単体**: `infrastructure/`

以降、`{infrastructure-root}` はこの判定結果に基づいたパスを指します。

### ステップ1: アーキテクチャ設計の読み込み

`docs/design-artifacts/architecture/` 配下の設計ドキュメントを読み込みます：

以下を抽出：
- **インフラコンポーネント**: 例: RDS, S3, Lambda, VPC
- **NFR**: パフォーマンス、セキュリティ、可用性、スケーラビリティ
- **データフロー**: コンポーネント間の接続
- **依存関係**: unit間の依存（例: unit2がunit1のVPCを使う）
- **環境要件**: dev, staging, production

### ステップ2: Terraformモジュール設計

unit単位でモジュール化します：

| コンポーネント | Terraformリソース | セキュリティ考慮 |
|-------------|------------------|----------------|
| データベース | `aws_db_instance`, `aws_dynamodb_table` | 暗号化、バックアップ |
| ストレージ | `aws_s3_bucket` | バージョニング、暗号化 |
| コンピュート | `aws_lambda_function`, `aws_ecs_service` | IAMロール最小権限 |
| ネットワーク | `aws_vpc`, `aws_security_group` | セキュリティグループ最小化 |

**依存関係の解決**:
- unit間の依存関係は、環境ごとの `main.tf` で module の output を参照
- 例: `module.unit1.vpc_id` を `module.unit2` に渡す

### ステップ3: ディレクトリ構造生成

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
    │   │   └── ...
    │   └── production/
    │       └── ...
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
│   │   ├── main.tf              # 全unitをここで呼び出し
│   │   └── ...
│   ├── staging/
│   └── production/
└── .gitignore
```

**重要**:
- モジュールは `{infrastructure-root}/terraform/modules/{unit}/` に生成
- **環境ごとの `main.tf` で全unitをまとめて呼び出す**
- unit間の依存関係は module の output で解決
- モノレポの場合は `package.json` も生成

### ステップ4: Terraformモジュール生成

#### terraform/modules/{unit}/versions.tf

```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

#### terraform/modules/{unit}/variables.tf

```hcl
variable "environment" {
  description = "Environment name (dev, staging, production)"
  type        = string
}

variable "project_name" {
  description = "Project name for resource naming"
  type        = string
}

# unit間の依存関係（例: unit1のVPCを受け取る）
variable "vpc_id" {
  description = "VPC ID from unit1"
  type        = string
  default     = null  # unit1の場合はnull
}

# NFRに基づく変数
variable "db_instance_class" {
  description = "RDS instance class"
  type        = string
  default     = "db.t3.micro"
}

variable "enable_backup" {
  description = "Enable automated backups"
  type        = bool
  default     = true
}

# タグ戦略
variable "tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default     = {}
}
```

#### terraform/modules/{unit}/main.tf（例: RDS + S3）

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

# S3 Bucket Versioning
resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id

  versioning_configuration {
    status = "Enabled"
  }
}

# RDS Instance
resource "aws_db_instance" "main" {
  identifier     = "${var.project_name}-${var.environment}-${local.unit_name}-db"
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = var.db_instance_class

  allocated_storage     = 20
  max_allocated_storage = 100
  storage_encrypted     = true

  db_name  = var.db_name
  username = var.db_username
  password = var.db_password  # 本番ではSecrets Manager推奨

  # unit1のVPCを使う場合
  vpc_security_group_ids = var.vpc_id != null ? [aws_security_group.db[0].id] : []
  db_subnet_group_name   = var.vpc_id != null ? aws_db_subnet_group.main[0].name : null

  backup_retention_period = var.enable_backup ? 7 : 0
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  skip_final_snapshot = var.environment != "production"

  tags = merge(
    var.tags,
    {
      Name = "${var.project_name}-${var.environment}-db"
      Unit = local.unit_name
    }
  )
}

locals {
  unit_name = "{unit}"
}
```

#### terraform/modules/{unit}/outputs.tf

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

output "db_name" {
  description = "Database name"
  value       = aws_db_instance.main.db_name
}

# unit1の場合: VPC IDを出力（他unitで使う）
output "vpc_id" {
  description = "VPC ID"
  value       = var.vpc_id != null ? var.vpc_id : aws_vpc.main[0].id
}
```

#### terraform/modules/{unit}/README.md

```markdown
# {Unit} Terraform Module

## 概要

{ユニットの説明}

## 依存関係

{unit間の依存関係がある場合、記載}

例:
- unit1のVPC IDを使用

## 使用方法

### 基本的な使用例

\`\`\`hcl
module "{unit}" {
  source = "../../terraform/modules/{unit}"

  environment  = "dev"
  project_name = "my-project"

  # unit間の依存
  vpc_id = module.unit1.vpc_id

  # NFRに基づく設定
  db_instance_class = "db.t3.micro"
  enable_backup     = true

  tags = {
    Project = "my-project"
    ManagedBy = "Terraform"
  }
}
\`\`\`

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| environment | Environment name | string | - | yes |
| project_name | Project name | string | - | yes |
| vpc_id | VPC ID from unit1 | string | null | no |

## Outputs

| Name | Description |
|------|-------------|
| s3_bucket_name | S3 bucket name |
| db_endpoint | RDS endpoint |
| vpc_id | VPC ID |

## セキュリティ考慮事項

- S3暗号化: デフォルト有効
- RDSバックアップ: 本番環境で7日保持
- タグ戦略: すべてのリソースにタグ付け
```

### ステップ5: 環境設定ファイル生成

#### environments/dev/versions.tf

```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "my-project-terraform-state"
    key    = "dev/terraform.tfstate"
    region = "ap-northeast-1"
  }
}
```

#### environments/dev/main.tf（全unitをまとめて呼び出し）

```hcl
provider "aws" {
  region = var.aws_region
}

# Unit1: ネットワーク層（VPC等）
module "unit1" {
  source = "../../terraform/modules/unit1"

  environment  = "dev"
  project_name = var.project_name

  tags = local.common_tags
}

# Unit2: データ層（RDS等）
# unit1のVPCを使用
module "unit2" {
  source = "../../terraform/modules/unit2"

  environment  = "dev"
  project_name = var.project_name

  # unit1の出力を参照
  vpc_id = module.unit1.vpc_id

  db_instance_class = "db.t3.micro"
  enable_backup     = false

  tags = local.common_tags
}

locals {
  common_tags = {
    Environment = "dev"
    Project     = var.project_name
    ManagedBy   = "Terraform"
  }
}
```

**重要**:
- **依存関係の順序**: unit1 → unit2 の順で定義
- **output参照**: `module.unit1.vpc_id` で unit1 の出力を unit2 に渡す
- **共通タグ**: `locals` で定義して再利用

#### environments/dev/variables.tf

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "ap-northeast-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
}
```

#### environments/dev/terraform.tfvars.example

```hcl
aws_region   = "ap-northeast-1"
project_name = "my-project"
```

#### environments/dev/outputs.tf

```hcl
# 各unitのoutputを集約
output "unit1_vpc_id" {
  description = "VPC ID from unit1"
  value       = module.unit1.vpc_id
}

output "unit2_db_endpoint" {
  description = "Database endpoint from unit2"
  value       = module.unit2.db_endpoint
  sensitive   = true
}

output "unit2_s3_bucket_name" {
  description = "S3 bucket name from unit2"
  value       = module.unit2.s3_bucket_name
}
```

### ステップ6: package.json生成（モノレポの場合のみ）

**モノレポの場合**、`packages/infrastructure/package.json` を生成：

```json
{
  "name": "@{project}/infrastructure",
  "version": "1.0.0",
  "private": true,
  "description": "Infrastructure as Code for {project}",
  "scripts": {
    "tf:init": "cd environments/dev && terraform init",
    "tf:init:staging": "cd environments/staging && terraform init",
    "tf:init:prod": "cd environments/production && terraform init",
    "tf:plan": "cd environments/dev && terraform plan",
    "tf:plan:staging": "cd environments/staging && terraform plan",
    "tf:plan:prod": "cd environments/production && terraform plan",
    "tf:apply": "cd environments/dev && terraform apply",
    "tf:apply:staging": "cd environments/staging && terraform apply",
    "tf:apply:prod": "cd environments/production && terraform apply",
    "tf:destroy": "cd environments/dev && terraform destroy",
    "tf:output": "cd environments/dev && terraform output",
    "fmt": "terraform fmt -recursive terraform/"
  }
}
```

### ステップ7: セキュリティベストプラクティス適用

以下を自動的に適用：

1. **暗号化**
   - S3: デフォルト暗号化有効
   - RDS: `storage_encrypted = true`
   - EBS: `encrypted = true`

2. **バックアップ**
   - RDS: 本番環境で7日保持、dev環境は無効
   - S3: バージョニング有効

3. **IAM最小権限**
   - Lambda実行ロールは必要最小限
   - S3バケットポリシーで明示的な許可のみ

4. **ネットワークセキュリティ**
   - Security Groupは最小限のポート開放
   - パブリックアクセスは明示的に許可された場合のみ

5. **タグ戦略**
   - すべてのリソースに `Environment`, `Project`, `Unit` タグ
   - コスト配分とリソース管理に活用

### ステップ8: ドキュメント生成

`docs/infrastructure/{unit}_IaC設計.md` を生成：

```markdown
# {Unit} - Infrastructure as Code設計

## 概要

{ユニットの説明}

## インフラコンポーネント

### データベース
- **種類**: PostgreSQL (RDS)
- **NFR**: 可用性99.9%, バックアップ7日保持
- **暗号化**: 有効

### ストレージ
- **種類**: S3
- **NFR**: 99.999999999% 耐久性
- **暗号化**: AES-256

## 依存関係

{unit間の依存関係}

例:
- unit1のVPC IDを使用してRDSを配置

## 環境ごとの設定

| 環境 | RDS Instance | Backup | 用途 |
|-----|--------------|--------|------|
| dev | db.t3.micro | なし | 開発環境 |
| staging | db.t3.small | 3日 | ステージング |
| production | db.r6g.large | 7日 | 本番環境 |

## セキュリティ設計

1. **データ保護**
   - 転送中: TLS 1.2+
   - 保存時: AES-256暗号化

2. **アクセス制御**
   - IAM最小権限の原則
   - Security Group最小化

## デプロイ手順

### モノレポの場合

\`\`\`bash
# 1. ルートから実行
pnpm --filter @{project}/infrastructure tf:init

# 2. プラン確認（全unit一括）
pnpm --filter @{project}/infrastructure tf:plan

# 3. 適用
pnpm --filter @{project}/infrastructure tf:apply

# 4. 出力確認
pnpm --filter @{project}/infrastructure tf:output
\`\`\`

### アプリ単体の場合

\`\`\`bash
# 1. 初期化
cd infrastructure/environments/dev
terraform init

# 2. プラン確認（全unit一括）
terraform plan

# 3. 適用
terraform apply

# 4. 出力確認
terraform output
\`\`\`

## リソース一覧

| リソース | タイプ | 説明 |
|---------|-------|------|
| VPC | aws_vpc | 仮想ネットワーク（unit1） |
| RDS | aws_db_instance | PostgreSQLデータベース |
| S3 | aws_s3_bucket | データストレージ |

## module間の依存関係

\`\`\`mermaid
graph TD
    unit1[Unit1: Network] -->|vpc_id| unit2[Unit2: Data]
    unit2 -->|db_endpoint| unit3[Unit3: Compute]
\`\`\`
```

## 出力

### 1. Terraformモジュール（新規生成）

- `{infrastructure-root}/terraform/modules/{unit}/main.tf`
- `{infrastructure-root}/terraform/modules/{unit}/variables.tf`
- `{infrastructure-root}/terraform/modules/{unit}/outputs.tf`
- `{infrastructure-root}/terraform/modules/{unit}/versions.tf`
- `{infrastructure-root}/terraform/modules/{unit}/README.md`

### 2. 環境設定（存在しなければ生成、既存なら更新）

- `{infrastructure-root}/environments/dev/main.tf` - **全unitをここで呼び出し**
- `{infrastructure-root}/environments/dev/variables.tf`
- `{infrastructure-root}/environments/dev/outputs.tf`
- `{infrastructure-root}/environments/dev/terraform.tfvars.example`
- `{infrastructure-root}/environments/dev/versions.tf`
- `{infrastructure-root}/environments/staging/...`
- `{infrastructure-root}/environments/production/...`

**重要**:
- 既に `environments/dev/main.tf` が存在する場合、新しい unit の module 呼び出しを**追加**する
- 依存関係を考慮した順序で module を配置

### 3. package.json（モノレポの場合のみ、存在しなければ生成）

- `packages/infrastructure/package.json`

### 4. ドキュメント（新規生成）

- `docs/infrastructure/{unit}_IaC設計.md`

### 5. .gitignore（存在しなければ生成）

- `{infrastructure-root}/.gitignore`

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

## 実行例

```bash
# 前提: /design-architecture unit1 と unit2 で設計完了済み

# コマンド実行（unit1）
/generate-iac unit1

# コマンド実行（unit2）
/generate-iac unit2

# 出力例（モノレポの場合）
✅ Infrastructure as Code生成完了

## プロジェクト構成
モノレポ構成を検出しました

## 生成されたファイル

### Terraformモジュール（unit2）
- packages/infrastructure/terraform/modules/unit2/main.tf
- packages/infrastructure/terraform/modules/unit2/variables.tf
- packages/infrastructure/terraform/modules/unit2/outputs.tf
- packages/infrastructure/terraform/modules/unit2/versions.tf
- packages/infrastructure/terraform/modules/unit2/README.md

### 環境設定（unit2を追加）
- packages/infrastructure/environments/dev/main.tf （unit2のmodule呼び出しを追加）
- packages/infrastructure/environments/dev/outputs.tf （unit2のoutputを追加）

### ドキュメント
- docs/infrastructure/002-unit2_IaC設計.md

## 次のステップ

1. 依存関係のインストール（モノレポ、初回のみ）:
   pnpm install

2. Terraform初期化（初回のみ）:
   pnpm --filter @{project}/infrastructure tf:init

3. プラン確認（全unit一括）:
   pnpm --filter @{project}/infrastructure tf:plan

4. 変数設定（初回のみ）:
   cd packages/infrastructure/environments/dev
   cp terraform.tfvars.example terraform.tfvars
   # terraform.tfvarsを編集

5. 適用（全unit一括デプロイ）:
   pnpm --filter @{project}/infrastructure tf:apply
```

## 注意事項

- **前提条件**: `/design-architecture {unit}` が実行済みであること
- **構成判定**: `packages/` または `apps/` の存在で自動判定
- **環境ごとに全unitをまとめる**: `environments/dev/main.tf` で全unitを呼び出し
- **依存関係の解決**: module の output を使って unit間の依存を解決
- **既存環境への追加**: 既に `environments/dev/main.tf` がある場合は module を追加
- **NFRに基づく設計**: アーキテクチャ設計のNFRを反映
- **セキュリティファースト**: デフォルトでセキュアな設定

## エラーハンドリング

- アーキテクチャ設計が見つからない場合はエラー（`/design-architecture`を先に実行してください）
- 不正な設計定義がある場合は警告
- 既存のモジュールがある場合は確認してから上書き
- 既存の環境設定がある場合は、新しい module 呼び出しを追加

## サポートするAWSリソース

### コンピュート
- Lambda
- ECS/Fargate
- EC2（オプション）

### データベース
- RDS（PostgreSQL, MySQL）
- DynamoDB

### ストレージ
- S3

### ネットワーク
- VPC
- Subnet
- Security Group
- ALB/NLB

### 監視
- CloudWatch
- SNS

### その他
- IAM Role/Policy
- Secrets Manager
- Parameter Store
