# Infrastructure as Code生成（Terraform）

このコマンドは**iac-generator Skill**を使用して、Terraform/Terragruntモジュールを自動生成します。

## Skillについて

`iac-generator` Skillは、アーキテクチャ設計からセキュアで再利用可能なTerraform/Terragruntモジュールを生成する再利用可能なSkillです。

**技術スタック**: Terraform + Terragrunt + AWS Provider

詳細は `.claude/skills/iac-generator/README.md` を参照してください。

## 前提条件

**このコマンドは `/design-architecture` の後に実行してください**

- `/design-architecture unit1` でアーキテクチャ設計が完了している
- `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md` にアーキテクチャ設計ドキュメントが存在する
- NFR（非機能要件）が定義されている

## 入力内容

{{ARGS}}

## Skill起動

### EnterPlanMode統合

インフラコード生成の前に **`EnterPlanMode` で Plan Mode に入り、生成計画を立てる**。

**Plan Mode内で実行する内容：**

1. **プロジェクト構成の判定**（Glob/Read）
   - packages/ または apps/ の存在を確認し、配置先を決定
   - 既存のTerraformモジュール/環境設定があれば確認
2. **アーキテクチャ設計の読み込み**
   - `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md` から設計ドキュメントを読み込み
   - 必要なAWSリソース、NFR要件を抽出
3. **Terraformモジュール構成計画**
   - unit単位のモジュール分割方針
   - 環境別（dev/staging/production）のデフォルト設定方針
   - 既存モジュールとの依存関係・統合方針
4. **生成ファイル一覧の提示**
   - 新規作成/上書きされるファイルのリスト
   - 既存の `environments/dev/main.tf` への追加内容

**`ExitPlanMode` で生成計画の承認を得る** → 承認後にコード生成を実行。

---

**実行する処理**:

承認後、引数として受け取ったユニット名をもとに、iac-generator Skillを起動します。

**ユニット名**: {{ARGS}}

**アーキテクチャ設計パス**: `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md`

**出力先**: 自動判定（packages/infrastructure/ または infrastructure/）

---

## Skill処理の詳細

Skillは `.claude/skills/iac-generator/SKILL.md` に定義された手順に従って処理を実行します。

**主要な処理**:

1. プロジェクト構成の判定（モノレポ vs アプリ単体）
2. アーキテクチャ設計の読み込み（コンポーネント、NFR、依存関係抽出）
3. Terraformモジュール設計（unit単位でモジュール化）
4. 環境別デフォルト設定（dev/staging/production）
5. Terraformモジュール生成（main.tf, variables.tf, outputs.tf等）
6. 環境設定ファイル生成（全unitをまとめて呼び出し）
7. package.json生成（モノレポの場合のみ）
8. セキュリティベストプラクティス適用（暗号化、バックアップ、IAM最小権限等）
9. IaC設計ドキュメント生成

**コード生成テンプレート詳細**: `.claude/skills/iac-generator/SKILL.md` を参照

---

## 環境別デフォルト設定

コスト最適化とセキュリティのバランス：

**POC/開発環境（dev）**:
- X-Ray: **無効** （コスト削減）
- バックアップ: **無効**
- インスタンスサイズ: **最小**

**本番環境（production）**:
- X-Ray: **有効** （パフォーマンス監視）
- Multi-AZ: **有効** （高可用性）
- バックアップ: **7日**

---

## 生成されるファイル

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

### 3. package.json（モノレポの場合のみ）

- `packages/infrastructure/package.json`

### 4. ドキュメント（新規生成）

- `{infrastructure-root}/docs/{unit}_IaC設計.md`

### 5. .gitignore（存在しなければ生成）

- `{infrastructure-root}/.gitignore`

---

## ディレクトリ構造

### モノレポの場合

```
packages/
└── infrastructure/
    ├── package.json
    ├── terraform/
    │   └── modules/
    │       ├── unit1/
    │       └── unit2/
    ├── environments/
    │   ├── dev/
    │   ├── staging/
    │   └── production/
    ├── docs/
    └── .gitignore
```

### アプリ単体の場合

```
infrastructure/
├── terraform/
├── environments/
├── docs/
└── .gitignore
```

---

## 実行後の次のステップ

```bash
# モノレポの場合
pnpm --filter @{project}/infrastructure tf:init
pnpm --filter @{project}/infrastructure tf:plan
pnpm --filter @{project}/infrastructure tf:apply

# アプリ単体の場合
cd infrastructure/environments/dev
terraform init
terraform plan
terraform apply
```

---

## プロジェクト構成の判定

まず、プロジェクト構成を自動判定します：

**判定ロジック**:
1. `packages/` ディレクトリが存在する → **モノレポ構成**
2. `apps/` ディレクトリが存在する → **アプリ単体構成**

**判定結果**:
- **モノレポ**: `packages/infrastructure/`
- **アプリ単体**: `infrastructure/`

---

## 注意事項

- **前提条件**: `/design-architecture {unit}` が実行済みであること
- **構成判定**: `packages/` または `apps/` の存在で自動判定
- **環境ごとに全unitをまとめる**: `environments/dev/main.tf` で全unitを呼び出し
- **依存関係の解決**: module の output を使って unit間の依存を解決
- **既存環境への追加**: 既に `environments/dev/main.tf` がある場合は module を追加
- **NFRに基づく設計**: アーキテクチャ設計のNFRを反映
- **セキュリティファースト**: デフォルトでセキュアな設定

---

**Skill Version**: 1.0.0
**Skill Location**: `.claude/skills/iac-generator/`
