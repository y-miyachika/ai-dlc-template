# AI-DLC Template - Claude Code 設定

## 🚀 AI-DLC統合（フレームワーク版）

このプロジェクトは、AWSが提唱する**AI-DLC（AI-Driven Development Lifecycle）**に準拠した開発フレームワークのテンプレートです。

**目的**: 組織全体でAI-DLC準拠の開発を推進するための標準テンプレート

### プロジェクト構成

このテンプレートはpnpm workspaceによるモノレポ構成です：

```
ai-dlc-template/
├── .claude/
│   └── commands/          # AI-DLCスラッシュコマンド
├── docs/
│   ├── guides/           # AI-DLC開発ガイド
│   ├── AI-DLC_日本語訳.md
│   └── AI-DLC付録A対比.md
├── templates/            # 各種テンプレートファイル
├── apps/                 # アプリケーション（Greenfield例）
│   └── my-app/
│       └── docs/         # アプリ固有のAI-DLC成果物
│           ├── backlog/
│           ├── intents/
│           ├── units/
│           ├── design-artifacts/
│           └── plans/
└── packages/             # 共通パッケージ
```

## AI-DLCスラッシュコマンド

### セットアップ

**`/setup-aidlc`**
- 用途: 新規プロジェクトのAI-DLC環境セットアップ
- 引数: プロジェクト名（省略時は対話形式）
- 動作: docs/ディレクトリ構造作成、CLAUDE.md生成、初期設定
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

### インフラ・API生成（未実装）

**`/generate-iac`**（未実装）
- 用途: Infrastructure as Code生成（Terraform/CDK）
- 引数: ユニット名
- 動作: アーキテクチャ設計からIaCコード生成、セキュリティベストプラクティス適用
- 出力: infrastructure/ディレクトリにTerraform/CDKコード
- 例: `/generate-iac unit1`

**`/generate-api`**（未実装）
- 用途: REST API実装生成
- 引数: ユニット名
- 動作: ドメインモデルからAPI仕様生成（OpenAPI）、実装コード生成
- 出力: packages/api/にAPIコード、docs/api/にOpenAPI仕様
- 例: `/generate-api unit1`

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

2. **セットアップ**
   ```bash
   pnpm install
   /setup-aidlc my-new-project
   ```

3. **AI-DLCサイクル開始**
   ```bash
   /intent <プロジェクト概要>
   /units
   /design-domain unit1
   /design-architecture unit1
   /design-test unit1
   /bolt unit1
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
   ```

## ドキュメント構造

### ルートdocs/ - AI-DLCフレームワーク共通

```
docs/
├── guides/              # AI-DLC開発ガイド
├── AI-DLC_日本語訳.md   # AI-DLC論文日本語訳
└── AI-DLC付録A対比.md   # 実装と論文の対比
```

### アプリ内docs/ - アプリ固有のAI-DLC成果物

```
apps/my-app/docs/
├── backlog/            # タスク定義（従来版・シンプル）
├── intents/            # インテント定義（AI-DLC準拠版）
├── requirements/       # 要件定義
├── units/             # ユニット分解
├── design-artifacts/  # 設計ドキュメント
│   ├── domain/        # ドメイン設計
│   ├── architecture/  # アーキテクチャ設計
│   ├── tests/         # テスト設計
│   └── adr/          # アーキテクチャ決定記録
└── plans/            # 実装計画
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
