# Skill 共通設定ガイド

このドキュメントでは、Skillの出力パスをプロジェクトに応じてカスタマイズする方法を説明します。

## 設定ファイル

プロジェクトルートに `.claude/skill-config.json` を作成することで、Skillの出力パスをカスタマイズできます。

### デフォルト設定

```json
{
  "api-generator": {
    "outputDir": "packages/api",
    "docsDir": "docs/api",
    "routesDir": "src/routes",
    "schemasDir": "src/schemas",
    "servicesDir": "src/services"
  },
  "iac-generator": {
    "outputDir": "terraform",
    "modulesDir": "modules",
    "environmentsDir": "environments"
  },
  "deploy-generator": {
    "workflowsDir": ".github/workflows",
    "scriptsDir": "scripts",
    "docsDir": "docs/deploy"
  }
}
```

### プロジェクトタイプ別の推奨設定

#### モノレポ（pnpm workspace）

```json
{
  "api-generator": {
    "outputDir": "packages/api",
    "docsDir": "docs/api"
  },
  "iac-generator": {
    "outputDir": "infrastructure/terraform"
  }
}
```

#### シングルアプリ

```json
{
  "api-generator": {
    "outputDir": "src/api",
    "docsDir": "docs/api"
  },
  "iac-generator": {
    "outputDir": "terraform"
  }
}
```

#### apps/配下のアプリ

```json
{
  "api-generator": {
    "outputDir": "apps/api",
    "docsDir": "docs/api"
  }
}
```

## 使用方法

### 1. 設定ファイルの作成

プロジェクトルートに `.claude/skill-config.json` を作成します。

### 2. Skillの実行

Skillは自動的に設定ファイルを読み込み、指定されたパスに出力します。

```bash
/generate-api unit1
# → 設定ファイルの outputDir に出力
```

### 3. 設定の上書き

コマンド実行時に引数で上書きすることも可能です：

```bash
/generate-api unit1 --output apps/backend
```

## 環境変数による設定

設定ファイルの代わりに、環境変数でも設定可能です：

```bash
export AIDLC_API_OUTPUT_DIR=apps/api
export AIDLC_IAC_OUTPUT_DIR=infra/terraform
export AIDLC_DEPLOY_WORKFLOWS_DIR=.github/workflows
```

**優先順位**:
1. コマンド引数
2. 環境変数
3. `.claude/skill-config.json`
4. デフォルト値

## 他プロジェクトへの導入

このテンプレートのSkillを他のプロジェクトで使用する場合：

1. `.claude/skills/` ディレクトリをコピー
2. `.claude/skill-config.json` をプロジェクトに合わせて作成
3. 必要なコマンド（`.claude/commands/`）をコピー

これにより、既存のプロジェクト構造を変更せずにSkillを導入できます。
