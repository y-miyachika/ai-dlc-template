# Terraform コーディング規約

- unit単位でモジュール化（`terraform/modules/{unit}/`）
- 命名パターン: `{project}-{environment}-{unit}-{resource_type}`
- 必須タグ: `Project`, `Environment`, `Unit`, `ManagedBy`
- **暗号化はデフォルト有効**: S3 AES256、RDS storage_encrypted、EBS encrypted
- **パブリックアクセスは明示的に許可された場合のみ**: S3 block_public_acls等
- **IAM最小権限**: ワイルドカード(`*`)禁止、リソースARN指定
- 機密出力は `sensitive = true`
- 環境別設定は `environments/{env}/terraform.tfvars` で上書き
- `terraform fmt -recursive` でフォーマット統一
