# AI-DLC Template - Claude Code 設定

AWSが提唱する**AI-DLC（AI-Driven Development Lifecycle）**に準拠した開発フレームワークのテンプレート。
組織全体でAI-DLC準拠の開発を推進するための標準テンプレートとして使用する。

## テンプレート構成

```
ai-dlc-template/
├── .claude/
│   ├── commands/          # AI-DLCスラッシュコマンド（13個）
│   ├── agents/            # SubAgents（深い思考・対話型）
│   ├── skills/            # Skills（コード生成型）
│   └── rules/             # パス別コーディング規約
├── docs/
│   ├── guides/            # AI-DLC開発ガイド
│   ├── AI-DLC_日本語訳.md
│   └── AI-DLC準拠状況.md
├── CLAUDE.md              # このファイル
└── README.md
```

`/setup-aidlc` 実行後に package.json、pnpm-workspace.yaml、apps/packages/ 等が自動生成される。

## コマンド概要

| フェーズ | コマンド | 種別 | 概要 |
|---------|---------|------|------|
| セットアップ | `/setup-aidlc` | - | プロジェクト初期化 |
| インセプション | `/intent` | SubAgent | インテント定義（要件明確化） |
| | `/units` | SubAgent | ユニット分解（DDD原則） |
| コンストラクション | `/design-domain` | SubAgent | ドメイン設計 |
| | `/design-architecture` | SubAgent | アーキテクチャ設計（NFR駆動） |
| | `/design-test` | SubAgent | テスト設計（TDD/BDD） |
| | `/bolt` | - | 高速反復（計画→TDD実装） |
| 生成 | `/generate-api` | Skill | REST API生成（Hono RPC） |
| | `/generate-iac` | Skill | Terraform/Terragrunt生成 |
| | `/generate-deploy` | Skill | CI/CDデプロイ設定生成 |
| ユーティリティ | `/progress` | - | 進捗確認 |
| | `/retro` | - | 振り返り・改善提案 |
| | `/sync-docs` | - | ドキュメント同期チェック |

詳細は @docs/guides/command-reference.md を参照。

## 開発ワークフロー

```
/intent → /units → /design-domain → /design-architecture → /design-test → /bolt → /generate-*
```

詳細は @docs/guides/getting-started.md 、@docs/guides/workflow.md を参照。

## ドキュメント構造・命名規則

@docs/guides/document-structure.md を参照。

## Unit完了チェックリスト

Unit実装完了時（コミット前）に確認：

- [ ] テスト全パス
- [ ] `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/plan.md` 作成・更新済み
- [ ] 関連ドキュメント確認
- [ ] 型エラーなし

## 詳細ガイド

| ガイド | 内容 |
|-------|------|
| @docs/guides/workflow-dag.md | コマンド間の依存関係図 |
| @docs/guides/implementation-scope.md | 各コマンドの完了条件 |
| @docs/guides/e2e-testing-flow.md | テスト実行タイミング |
| @docs/guides/external-api-constraints.md | レート制限・SLA対応 |
| @docs/guides/project-templates.md | プロジェクトタイプ別ガイド |
| @.claude/agents/quality-checklist.md | 品質基準 |
| @docs/guides/best-practices.md | 推奨パターン |

## 応答言語

- **デフォルト言語: 日本語**
- すべての応答は日本語で行う
- ユーザーが明示的に英語を要求した場合のみ英語で応答

## gitコミット規約

**重要: ユーザーの明示的な指示があるまでcommitしない**

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
