# AI-DLC Template - Claude Code 設定

## 🚀 AI-DLC統合（フレームワーク版）

このプロジェクトは、AWSが提唱する**AI-DLC（AI-Driven Development Lifecycle）**に準拠した開発フレームワークのテンプレートです。

**目的**: 組織全体でAI-DLC準拠の開発を推進するための標準テンプレート

### テンプレート構成

このテンプレートは、AI-DLC方法論とスラッシュコマンドを提供します：

```
ai-dlc-template/
├── .claude/commands/      # AI-DLCスラッシュコマンド（9個）
├── docs/
│   ├── guides/           # AI-DLC開発ガイド
│   ├── AI-DLC_日本語訳.md
│   └── AI-DLC準拠状況.md
├── .gitignore
├── CLAUDE.md             # このファイル
└── README.md             # プロジェクト概要
```

**注**: `/setup-aidlc` 実行後に以下が自動生成されます：
- package.json（pnpm workspace設定）
- pnpm-workspace.yaml
- apps/ または packages/ ディレクトリ
- プロジェクト固有のCLAUDE.md、README.md

## AI-DLCスラッシュコマンド

### セットアップ

**`/setup-aidlc`**
- 用途: 新規プロジェクトのAI-DLC環境セットアップ
- 引数: プロジェクト名（省略時は対話形式）
- 動作:
  1. プロジェクト情報の収集（対話形式）
  2. package.json、pnpm-workspace.yamlの生成
  3. apps/ または packages/ ディレクトリの作成
  4. docs/配下のAI-DLC成果物用ディレクトリ作成
  5. CLAUDE.md、README.md、.gitignoreの生成
- 例: `/setup-aidlc my-new-app`

### インセプションフェーズ

**`/intent`**
- 用途: AI-DLC準拠のインテント定義（要件明確化）
- 引数: タスクの概要（自由記述）
- 動作: AIが4つの質問で明確化 → ユーザーストーリー、NFR、リスク定義
- 出力: docs/intents/に連番付きMarkdownファイル
- 例: `/intent ユーザー認証機能の実装`

**`/units`**
- 用途: インテント/Backlogをユニットに分解（疎結合・高凝集）
- 引数: Intent番号（省略時は最新）
- 動作: DDDのサブドメイン概念でユニット分解、依存関係図作成
- 出力: docs/units/に分解結果を保存
- 例: `/units 001` または `/units`

### コンストラクションフェーズ

**`/design-domain`**
- 用途: ユニットのドメイン設計（DDD原則）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 動作: エンティティ、集約、値オブジェクト、ドメインイベント等を設計
- 出力: docs/design-artifacts/domain/にドメインモデルを保存
- 例: `/design-domain unit1`

**`/design-architecture`**
- 用途: NFR考慮のアーキテクチャ設計（論理設計）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 動作: NFR分析 → アーキテクチャパターン選択 → トレードオフ分析 → ADR生成
- 出力: docs/design-artifacts/architecture/にアーキテクチャ設計、docs/design-artifacts/adr/にADRを保存
- 例: `/design-architecture unit1`

**`/design-test`**
- 用途: テスト設計（TDD/BDD統合）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 動作: BDD受入基準 → TDDテストケース → テスト実装計画
- 出力: docs/design-artifacts/tests/にテスト設計を保存
- 例: `/design-test unit1`

**`/bolt`**
- 用途: 高速反復サイクル（計画→実装を1サイクルで）
- 引数: ユニット名（例: `unit1`）
- 動作: 計画がなければ作成 → 承認 → TDDサイクルで実装 → テスト → 次ステップ提案
- 例: `/bolt unit1`

### インフラ・API生成

**`/generate-api`**
- 用途: REST API実装生成（Hono RPC）
- 引数: ユニット名（例: `unit1`）
- 前提条件: `/bolt` でservices層が実装済みであること
- 動作:
  1. 既存のservices層を読み込み（packages/api/src/services/）
  2. HTTP層を生成（routes/, schemas/, index.ts, types/）
  3. OpenAPI仕様を生成（docs/api/openapi.yaml）
- 出力:
  - packages/api/src/routes/ - Honoルート定義
  - packages/api/src/schemas/ - Zodスキーマ
  - packages/api/src/index.ts - Honoアプリケーション
  - packages/api/src/types/index.ts - 型エクスポート
  - docs/api/openapi.yaml - OpenAPI仕様
  - docs/api/{unit}_API設計.md - API設計ドキュメント
- 例: `/generate-api unit1`

**`/generate-iac`**
- 用途: Infrastructure as Code生成（Terraform）
- 引数: ユニット名（例: `unit1`）
- 前提条件: `/design-architecture` でアーキテクチャ設計が完了していること
- 動作:
  1. アーキテクチャ設計を読み込み（docs/design-artifacts/architecture/）
  2. unit単位でTerraformモジュールを生成
  3. 環境ごとに全unitをまとめて呼び出し（dev/staging/production）
  4. unit間の依存関係をmodule outputで解決
- 出力:
  - {infrastructure-root}/terraform/modules/{unit}/ - Terraformモジュール
  - {infrastructure-root}/environments/dev/main.tf - 環境設定（全unit呼び出し）
  - docs/infrastructure/{unit}_IaC設計.md - IaC設計ドキュメント
- 例: `/generate-iac unit1`

### オペレーションフェーズ（未実装）

**`/operate`**（未実装）
- 用途: 運用フェーズのサポート
- 引数: なし
- 動作: テレメトリ分析、インシデント管理、改善提案
- 出力: docs/operations/に運用レポート
- 例: `/operate`

## 開発ワークフロー

### 新規プロジェクト開始時

1. **テンプレートから作成**
   ```bash
   git clone codecommit::ap-northeast-1://devops@ai-dlc-template my-new-project
   cd my-new-project
   ```

2. **AI-DLC環境のセットアップ**
   ```bash
   /setup-aidlc my-new-project
   ```

   このコマンドで以下が自動作成されます：
   - package.json、pnpm-workspace.yaml
   - apps/ または packages/ ディレクトリ
   - docs/配下のAI-DLC成果物用ディレクトリ
   - CLAUDE.md、README.md、.gitignore

3. **依存関係のインストール**
   ```bash
   pnpm install
   ```

4. **AI-DLCサイクル開始**
   ```bash
   /intent <プロジェクト概要>
   /units
   /design-domain unit1
   /design-architecture unit1
   /design-test unit1
   /bolt unit1
   /generate-api unit1  # APIが必要な場合
   /generate-iac unit1  # インフラが必要な場合
   ```

### 既存プロジェクトでの活用

1. **インテント定義から開始**
   ```bash
   /intent <新機能の概要>
   ```

2. **設計フェーズ**
   ```bash
   /units
   /design-domain unit1
   /design-architecture unit1
   /design-test unit1
   ```

3. **実装フェーズ**
   ```bash
   /bolt unit1
   /generate-api unit1  # APIが必要な場合
   /generate-iac unit1  # インフラが必要な場合
   ```

## ドキュメント構造

### ルートdocs/ - AI-DLCフレームワーク共通（テンプレートに含まれる）

```
docs/
├── guides/              # AI-DLC開発ガイド
│   ├── getting-started.md
│   └── workflow.md
├── AI-DLC_日本語訳.md   # AI-DLC論文日本語訳
└── AI-DLC準拠状況.md    # 準拠率分析・実装状況
```

### プロジェクトのdocs/ - `/setup-aidlc`実行後に作成される

プロジェクトタイプに応じて、以下のような構造が作成されます：

**appの場合:**
```
<project-name>/docs/
├── intents/            # インテント定義（AI-DLC準拠版）
├── units/             # ユニット分解
├── design-artifacts/  # 設計ドキュメント
│   ├── domain/        # ドメイン設計
│   ├── architecture/  # アーキテクチャ設計
│   ├── tests/         # テスト設計
│   └── adr/          # アーキテクチャ決定記録
└── plans/            # 実装計画
```

**monorepoの場合:**
```
apps/<app-name>/docs/  # 各アプリ固有のAI-DLC成果物
packages/<pkg-name>/docs/  # 各パッケージ固有のAI-DLC成果物
```

## 応答言語

- **デフォルト言語: 日本語**
- すべての応答は日本語で行う
- ユーザーが明示的に英語を要求した場合のみ英語で応答

## gitコミット規約

**重要: ユーザーの明示的な指示があるまでcommitしない**

**コミットメッセージ形式:**
```bash
git commit -m "$(cat <<'EOF'
機能追加: XXXの実装

## 実装内容
- 機能A
- 機能B

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```
