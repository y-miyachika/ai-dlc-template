# AI-DLC Template - Claude Code 設定

## 🚀 AI-DLC統合（フレームワーク版）

このプロジェクトは、AWSが提唱する**AI-DLC（AI-Driven Development Lifecycle）**に準拠した開発フレームワークのテンプレートです。

**目的**: 組織全体でAI-DLC準拠の開発を推進するための標準テンプレート

### テンプレート構成

このテンプレートは、AI-DLC方法論、スラッシュコマンド、再利用可能なSkill/SubAgentを提供します：

```
ai-dlc-template/
├── .claude/
│   ├── commands/          # AI-DLCスラッシュコマンド（15個）
│   ├── agents/            # SubAgents（深い思考・対話型）
│   │   ├── intent-definer/
│   │   ├── units-decomposer/
│   │   ├── domain-designer/
│   │   ├── architecture-designer/
│   │   └── test-designer/
│   └── skills/            # Skills（コード生成型）
│       ├── api-generator/
│       ├── iac-generator/
│       └── deploy-generator/
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

### Skill/SubAgentアーキテクチャ

各コマンドは、**Skill**（コード生成）または**SubAgent**（深い思考・対話）を使用します：

**Skills（コード生成型）**:
- `api-generator`: REST API実装生成（Hono RPC）
- `iac-generator`: Infrastructure as Code生成（Terraform/Terragrunt）
- `deploy-generator`: デプロイ設定生成（GitHub Actions、スクリプト）

**SubAgents（深い思考・対話型）**:
- `intent-definer`: インテント定義（要件明確化）
- `units-decomposer`: ユニット分解（DDD原則）
- `domain-designer`: ドメイン設計（エンティティ、集約等）
- `architecture-designer`: アーキテクチャ設計（NFR駆動、ADR生成）
- `test-designer`: テスト設計（TDD/BDD統合）

詳細は各Skill/SubAgentの `README.md` を参照してください。

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

**`/intent`** → `intent-definer` SubAgent
- 用途: AI-DLC準拠のインテント定義（要件明確化）
- 引数: タスクの概要（自由記述）
- 出力: docs/intents/に連番付きMarkdownファイル
- 例: `/intent ユーザー認証機能の実装`
- 詳細: `.claude/agents/intent-definer/README.md`

**`/units`** → `units-decomposer` SubAgent
- 用途: インテント/Backlogをユニットに分解（疎結合・高凝集）
- 引数: Intent番号（省略時は最新）
- 出力: docs/units/に分解結果を保存
- 例: `/units 001` または `/units`
- 詳細: `.claude/agents/units-decomposer/README.md`

### コンストラクションフェーズ

**`/design-domain`** → `domain-designer` SubAgent
- 用途: ユニットのドメイン設計（DDD戦術的設計パターン）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 出力: docs/design-artifacts/domain/にドメインモデルを保存
- 例: `/design-domain unit1`
- 詳細: `.claude/agents/domain-designer/README.md`

**`/design-architecture`** → `architecture-designer` SubAgent
- 用途: NFR考慮のアーキテクチャ設計（NFR駆動、ADR生成）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 出力: docs/design-artifacts/architecture/にアーキテクチャ設計、docs/design-artifacts/adr/にADR
- 例: `/design-architecture unit1`
- 詳細: `.claude/agents/architecture-designer/README.md`

**`/design-test`** → `test-designer` SubAgent
- 用途: テスト設計（TDD/BDD統合、テストピラミッド構築）
- 引数: ユニット名（例: `unit1`, `001-unit1`）
- 出力: docs/design-artifacts/tests/にテスト設計を保存
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

### インフラ・API生成

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

### オペレーションフェーズ（未実装）

**`/operate`**（未実装）
- 用途: 運用フェーズのサポート
- 引数: なし
- 動作: テレメトリ分析、インシデント管理、改善提案
- 出力: docs/operations/に運用レポート
- 例: `/operate`

## 詳細ガイド

より詳しい情報は以下のガイドを参照してください：

| ガイド | 内容 |
|-------|------|
| [ワークフローDAG](docs/guides/workflow-dag.md) | コマンド間の依存関係図、並列実行可能な組み合わせ |
| [実装スコープ](docs/guides/implementation-scope.md) | 各コマンドの完了条件チェックリスト |
| [E2Eテストフロー](docs/guides/e2e-testing-flow.md) | テスト実行タイミング、P0/P1/P2優先度分類 |
| [外部API制約](docs/guides/external-api-constraints.md) | レート制限、クォータ、SLA対応パターン |
| [プロジェクトテンプレート](docs/guides/project-templates.md) | モノレポ/シングルアプリ/マイクロサービス別ガイド |
| [品質チェックリスト](.claude/agents/quality-checklist.md) | SubAgent/Skill出力の品質基準 |
| [ワークフロー](docs/guides/workflow.md) | AI-DLC 3フェーズの詳細 |
| [開始ガイド](docs/guides/getting-started.md) | 最初のステップ |
| [ベストプラクティス](docs/guides/best-practices.md) | 推奨パターン |

## 開発ワークフロー

### 新規プロジェクト開始時

1. **テンプレートから作成**
   ```bash
   git clone https://github.com/y-miyachika/ai-dlc-template.git my-new-project
   cd my-new-project
   rm -rf .git && git init  # 新規リポジトリとして初期化
   ```
   または、GitHub上で「Use this template」ボタンから新規リポジトリを作成

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
docs/                      # AI-DLC成果物はルートに集約
├── intents/              # インテント定義
├── units/                # ユニット分解
├── design-artifacts/     # 全ユニットの設計ドキュメント
│   ├── domain/           # 001〜
│   ├── architecture/     # 001〜
│   ├── tests/            # 001〜
│   └── adr/              # ADR
└── plans/                # 実装計画
```

**理由**: 横断的な参照が容易、ユニット間の依存関係を把握しやすい

### AI-DLCドキュメント命名規則

**Intent.Unit形式を使用**（例: `002-001-strivo-effect-visualizer.md`）

| 種類 | 命名パターン | 例 |
|------|-------------|-----|
| Intent | `{Intent番号}_タイトル.md` | `002_ユーザー認証.md` |
| Units | `{Intent番号}_ユニット分解.md` | `002_ユニット分解.md` |
| design-artifacts | `{Intent番号}-{Unit番号}-名前.md` | `002-001-login-api.md` |
| plans | `{Intent番号}-{Unit番号}-名前.md` | `002-001-login-api.md` |

- Intent番号・Unit番号は3桁ゼロ埋め
- Unit番号`000`はshared/共通拡張用（例: `002-000-shared-domain.md`）

### Unit完了チェックリスト

Unit実装完了時（コミット前）に確認：

- [ ] テスト全パス
- [ ] `docs/plans/{Intent}-{Unit}-*.md` 作成・更新済み
- [ ] `docs/design-artifacts/` の関連ドキュメント確認
- [ ] 型エラーなし

### ローカル開発環境（プロジェクト固有）

プロジェクトでDynamoDB等のAWSリソースを使う場合：

```bash
# packages/api/.env.example を .env にコピーして設定
cp packages/api/.env.example packages/api/.env

# 環境変数例
DYNAMODB_TABLE_NAME=your-table-name
AWS_REGION=ap-northeast-1
```

**注意**: `.env`は`.gitignore`に含める

## コーディング規約

### TypeScript

- strict modeを有効にする
- 型推論に頼りすぎず、明示的な型定義を優先
- any型の使用は禁止（unknownを使用）
- **classを使わない**: 関数ベースの設計を優先
  - 依存性注入は引数で渡す
  - 状態管理はクロージャまたはオブジェクトで

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
