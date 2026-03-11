# AI-DLCスラッシュコマンド リファレンス

## セットアップ

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

## インセプションフェーズ

**`/intent`** → `intent-definer` SubAgent
- 用途: AI-DLC準拠のインテント定義（要件明確化）
- 引数: タスクの概要（自由記述）
- 出力: docs/intents/{Intent番号}_{Intent名}/intent.md
- 例: `/intent ユーザー認証機能の実装`
- 詳細: `.claude/agents/intent-definer/README.md`

**`/units`** → `units-decomposer` SubAgent
- 用途: インテント/Backlogをユニットに分解（疎結合・高凝集）
- 引数: Intent番号（省略時は最新）
- 出力: docs/intents/{Intent番号}_{Intent名}/units.md
- 例: `/units 001` または `/units`
- 詳細: `.claude/agents/units-decomposer/README.md`

## コンストラクションフェーズ

**`/design-domain`** → `domain-designer` SubAgent
- 用途: ユニットのドメイン設計（DDD戦術的設計パターン）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 出力: docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/domain.md
- 例: `/design-domain unit1`
- 詳細: `.claude/agents/domain-designer/README.md`

**`/design-architecture`** → `architecture-designer` SubAgent
- 用途: NFR考慮のアーキテクチャ設計（NFR駆動、ADR生成）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 出力: docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md、docs/adr/にADR
- 例: `/design-architecture unit1`
- 詳細: `.claude/agents/architecture-designer/README.md`

**`/design-test`** → `test-designer` SubAgent
- 用途: テスト設計（TDD/BDD統合、テストピラミッド構築）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 出力: docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/tests.md
- 例: `/design-test unit1`
- 詳細: `.claude/agents/test-designer/README.md`

**`/bolt`**
- 用途: 高速反復サイクル（計画→実装を1サイクルで）
- 引数: ユニット名（例: `unit1`）
- 動作: 計画がなければ作成 → 承認 → TDDサイクルで実装 → テスト → 次ステップ提案
- 例: `/bolt unit1`

**`/retro`**（Retrospective）
- 用途: 会話の振り返りと改善提案
- 引数: なし
- 動作: CLAUDE.mdへの追記提案、新規コマンド提案、既存コマンド改善提案
- 例: `/retro`

**`/progress`**
- 用途: プロジェクト/パッケージの現状確認
- 引数: パッケージ名（省略時は全体）
- 動作: Git状況 + 実装進捗 + ブロッカー確認
- 例: `/progress collector`

## インフラ・API生成

**`/generate-api`** → `api-generator` Skill
- 用途: REST API実装生成（Hono RPC、OpenAPI）
- 引数: ユニット名（例: `unit1`）
- 前提条件: `/bolt` でservices層が実装済みであること
- 出力: packages/api/src/ にHTTP層、docs/api/ にOpenAPI仕様
- 例: `/generate-api unit1`
- 詳細: `.claude/skills/api-generator/README.md`

**`/generate-iac`** → `iac-generator` Skill
- 用途: Infrastructure as Code生成（Terraform/Terragrunt）
- 引数: ユニット名（例: `unit1`）
- 前提条件: `/design-architecture` でアーキテクチャ設計が完了していること
- 出力: terraform/modules/{unit}/ にモジュール、environments/ に環境設定
- 例: `/generate-iac unit1`
- 詳細: `.claude/skills/iac-generator/README.md`

**`/generate-deploy`** → `deploy-generator` Skill
- 用途: デプロイ設定生成（GitHub Actions、デプロイスクリプト）
- 引数: ユニット名（例: `unit1`）
- 前提条件: `/design-architecture` でアーキテクチャ設計が完了していること
- 出力: .github/workflows/deploy.yml、scripts/deploy.sh、docs/deploy/README.md
- 例: `/generate-deploy unit1`
- 詳細: `.claude/skills/deploy-generator/README.md`

**`/sync-docs`**
- 用途: 設計ドキュメントと実装の同期チェック
- 引数: パッケージ名またはUnit番号（省略時は全体）
- 動作: 変更ファイル検出 → 関連ドキュメント特定 → 乖離チェック → 更新提案
- 例: `/sync-docs` または `/sync-docs api`

## オペレーションフェーズ（未実装）

**`/operate`**（未実装）
- 用途: 運用フェーズのサポート
- 引数: なし
- 動作: テレメトリ分析、インシデント管理、改善提案
- 出力: docs/operations/に運用レポート
- 例: `/operate`

## 並列実行パターン

独立したコマンドは並列起動が可能：

| 並列パターン | 説明 |
|---|---|
| `/generate-api` + `/generate-iac` + `/generate-deploy` | 生成系は完全に独立 |
| `/design-domain`（unit A） + `/design-domain`（unit B） | 異なるユニットの設計は独立 |

**注意**: `/design-domain` → `/design-architecture` → `/design-test` は順序あり

## SubAgentタイプの使い分け

| タイプ | 推奨場面 | 対応コマンド |
|---|---|---|
| `Plan` | NFR駆動のアーキテクチャ設計 | `/design-architecture` |
| `Explore` | 既存コードベースの探索・分析 | `/design-domain`（前段階）, `/sync-docs` |
| `general-purpose` | 対話型の設計・生成 | `/intent`, `/units`, `/bolt` |
