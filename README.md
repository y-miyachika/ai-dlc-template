# AI-DLC Template

**AI駆動型開発ライフサイクル（AI-DLC）** のプロジェクトテンプレート

このテンプレートを使って、AI-DLC方法論に基づいた新規プロジェクトを開始できます。

## 🎯 このテンプレートについて

AI-DLCは、AWSが提唱する**AI駆動型開発ライフサイクル**の方法論です。このテンプレートは、AI-DLCに準拠した開発を組織全体で推進するための標準フレームワークを提供します。

### プロジェクトの由来

このテンプレートは、実践的なAI-DLC開発を経て抽出された汎用テンプレートです：
- TDD/BDD統合、DDD原則に基づく設計フェーズを確立
- 実プロジェクトでの知見を基に、再利用可能なスラッシュコマンドとして体系化
- 2025-11-17に汎用テンプレートとして独立

### 主な特徴

- **AI-DLC準拠率 80%**: AWS論文の付録Aに対する準拠率（IaC生成追加）
- **3つのフェーズをカバー**: Inception（要件定義）、Construction（設計・実装）、Operations（運用）
- **13個のAIエージェント**: コード生成型（3個）と対話・分析型（5個）+ ユーティリティ（5個）
- **デュアルツール対応**: Claude Code と GitHub Copilot の両方で同じ13コマンドが利用可能
- **自動セットアップ**: `/setup-aidlc`で必要な構成を自動生成
- **TDD/BDD統合**: テストファーストの開発サイクル
- **DDD原則**: ドメイン駆動設計に基づく設計フェーズ
- **完全なドキュメント**: AI-DLC論文の日本語訳と詳細ガイド

### AI-DLCの価値

1. **要件の明確化**: AIとの対話で要件を体系的に定義
2. **設計の品質**: NFR（非機能要件）を考慮したアーキテクチャ
3. **実装の高速化**: TDDサイクルで品質を維持しながら高速開発
4. **トレーサビリティ**: インテント → ユニット → 設計 → コード の完全な追跡
5. **継続的改善**: フィードバックループによる改善サイクル

## 🚀 使い方

### 1. このテンプレートから新規プロジェクト作成

```bash
# GitHubからクローン
git clone https://github.com/y-miyachika/ai-dlc-template.git my-new-project
cd my-new-project
rm -rf .git && git init  # 新規リポジトリとして初期化
```

または、GitHub上で「Use this template」ボタンから新規リポジトリを作成できます。

### 2. AI-DLC環境のセットアップ

```bash
# Claude Code
/setup-aidlc my-new-project

# GitHub Copilot
@setup-aidlc my-new-project
```

このコマンドで以下が自動作成されます：
- package.json（pnpm workspace設定）
- pnpm-workspace.yaml
- apps/ または packages/ ディレクトリ
- docs/ 配下のAI-DLC成果物用ディレクトリ
- README.md、.gitignore、AI設定ファイル

### 3. 依存関係のインストール

```bash
pnpm install
```

### 4. 開発開始

Claude Code では `/コマンド名`、GitHub Copilot では `@エージェント名` で呼び出します。

| フェーズ | Claude Code | GitHub Copilot |
|---------|-------------|----------------|
| インテント定義 | `/intent <概要>` | `@intent-definer <概要>` |
| ユニット分解 | `/units` | `@units-decomposer` |
| ドメイン設計 | `/design-domain unit1` | `@domain-designer unit1` |
| アーキテクチャ設計 | `/design-architecture unit1` | `@architecture-designer unit1` |
| テスト設計 | `/design-test unit1` | `@test-designer unit1` |
| 実装 | `/bolt unit1` | `@bolt unit1` |
| API生成 | `/generate-api unit1` | `@api-generator unit1` |
| IaC生成 | `/generate-iac unit1` | `@iac-generator unit1` |
| デプロイ設定 | `/generate-deploy unit1` | `@deploy-generator unit1` |

## 📁 テンプレート構造

```
ai-dlc-template/
├── .claude/                    # Claude Code用
│   ├── commands/               # スラッシュコマンド（13個）
│   ├── agents/                 # 対話・分析型エージェント定義
│   └── skills/                 # コード生成型エージェント定義
├── .github/                    # GitHub Copilot用
│   ├── copilot-instructions.md # プロジェクト全体設定
│   ├── agents/                 # カスタムエージェント（13個）
│   └── instructions/           # タスク別インストラクション
├── docs/                       # AI-DLC方法論ドキュメント
│   ├── AI-DLC_日本語訳.md      # 論文完全翻訳
│   ├── AI-DLC準拠状況.md       # 準拠率分析・実装状況
│   └── guides/                 # 開発ガイド
├── .gitignore
├── CLAUDE.md                   # Claude Code設定
└── README.md                   # このファイル
```

**注**: `apps/`, `packages/`, `package.json`, `pnpm-workspace.yaml` は `/setup-aidlc` 実行時に自動生成されます。

### エージェントの役割分類

13個のエージェントは、役割によって2種類に分類されます：

**コード生成型**（テンプレートベースで成果物を出力）:
- `api-generator`: REST API実装生成（Hono RPC、OpenAPI）
- `iac-generator`: Infrastructure as Code生成（Terraform/Terragrunt）
- `deploy-generator`: デプロイ設定生成（GitHub Actions）

**対話・分析型**（ユーザーとの対話で設計を深掘り）:
- `intent-definer`: インテント定義（要件明確化）
- `units-decomposer`: ユニット分解（DDD原則）
- `domain-designer`: ドメイン設計（エンティティ、集約等）
- `architecture-designer`: アーキテクチャ設計（NFR駆動、ADR生成）
- `test-designer`: テスト設計（TDD/BDD統合）

詳細は各エージェントの定義ファイルを参照してください。

## 📖 利用可能なコマンド（13個）

Claude Code では `/コマンド名`、GitHub Copilot では `@エージェント名` で呼び出します。

### セットアップ

| 用途 | Claude Code | GitHub Copilot |
|------|-------------|----------------|
| AI-DLC環境セットアップ | `/setup-aidlc` | `@setup-aidlc` |

### インセプションフェーズ（要件定義）

| 用途 | Claude Code | GitHub Copilot |
|------|-------------|----------------|
| インテント定義 | `/intent` | `@intent-definer` |
| ユニット分解 | `/units` | `@units-decomposer` |

### コンストラクションフェーズ（設計・実装）

| 用途 | Claude Code | GitHub Copilot |
|------|-------------|----------------|
| ドメイン設計 | `/design-domain` | `@domain-designer` |
| アーキテクチャ設計 | `/design-architecture` | `@architecture-designer` |
| テスト設計 | `/design-test` | `@test-designer` |
| 実装（TDDサイクル） | `/bolt` | `@bolt` |

### インフラ・API・デプロイ生成

| 用途 | Claude Code | GitHub Copilot |
|------|-------------|----------------|
| REST API生成 | `/generate-api` | `@api-generator` |
| IaC生成 | `/generate-iac` | `@iac-generator` |
| デプロイ設定生成 | `/generate-deploy` | `@deploy-generator` |

### ユーティリティ

| 用途 | Claude Code | GitHub Copilot |
|------|-------------|----------------|
| 進捗確認 | `/progress` | `@progress` |
| 振り返り・改善提案 | `/retro` | `@retro` |
| ドキュメント同期チェック | `/sync-docs` | `@sync-docs` |

### オペレーションフェーズ（未実装）

- `/operate` - 運用サポート（テレメトリ分析等）

詳細は [CLAUDE.md](CLAUDE.md) または [.github/copilot-instructions.md](.github/copilot-instructions.md) を参照してください。

## 🔮 今後の拡張予定

以下のコマンドが未実装です（全機能実装後の目標準拠率: 85%+）：

- `/operate` - 運用サポート（テレメトリ分析、インシデント管理）

詳細な準拠率分析は [AI-DLC準拠状況](docs/AI-DLC準拠状況.md) を参照してください。

## 🎓 AI-DLCについて

### 学習リソース

- [AI-DLC日本語訳](docs/AI-DLC_日本語訳.md) - AWS論文の完全な翻訳
- [AI-DLC準拠状況](docs/AI-DLC準拠状況.md) - 準拠率分析と実装状況
- [開始ガイド](docs/guides/getting-started.md) - 最初のステップ
- [ワークフローガイド](docs/guides/workflow.md) - 開発フローの詳細

### 参考資料

- [AI-DLC公式ブログ（日本語）](https://aws.amazon.com/jp/blogs/news/ai-driven-development-life-cycle/) - AWS公式の方法論紹介
- [AI-DLC公式ブログ（英語）](https://aws.amazon.com/blogs/devops/ai-driven-development-life-cycle/) - 原文
- [AI-DLC公式サイト](https://prod.d13rzhkk8cj2z0.amplifyapp.com/) - 包括的なホワイトペーパー

## 📄 ライセンス

MIT License

---

**作成日**: 2025-11-17
**バージョン**: 1.0.0
