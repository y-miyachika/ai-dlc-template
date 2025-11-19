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
- **Skill/SubAgentアーキテクチャ**: 再利用可能なSkill（コード生成）とSubAgent（深い思考・対話）
  - 2つのSkills: `api-generator`, `iac-generator`
  - 5つのSubAgents: `intent-definer`, `units-decomposer`, `domain-designer`, `architecture-designer`, `test-designer`
- **スラッシュコマンド**: Claude Code用の9個の開発支援コマンド（+1個未実装）
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
# CodeCommitの場合
git clone codecommit::ap-northeast-1://devops@ai-dlc-template my-new-project
cd my-new-project

# ローカルの場合
cp -r ai-dlc-template my-new-project
cd my-new-project
git init
```

### 2. AI-DLC環境のセットアップ

```bash
# Claude Codeで /setup-aidlc を実行
/setup-aidlc my-new-project
```

このコマンドで以下が自動作成されます：
- package.json（pnpm workspace設定）
- pnpm-workspace.yaml
- apps/ または packages/ ディレクトリ
- docs/ 配下のAI-DLC成果物用ディレクトリ
- CLAUDE.md、README.md、.gitignore

### 3. 依存関係のインストール

```bash
pnpm install
```

### 4. 開発開始

```bash
# インテント定義から始める
/intent <プロジェクト概要>

# ユニット分解
/units

# ドメイン設計
/design-domain unit1

# アーキテクチャ設計
/design-architecture unit1

# テスト設計
/design-test unit1

# 実装
/bolt unit1

# API生成（必要な場合）
/generate-api unit1

# インフラ生成（必要な場合）
/generate-iac unit1
```

## 📁 テンプレート構造

```
ai-dlc-template/
├── .claude/
│   ├── commands/               # AI-DLCスラッシュコマンド（9個）
│   │   ├── setup-aidlc.md
│   │   ├── intent.md
│   │   ├── units.md
│   │   ├── design-domain.md
│   │   ├── design-architecture.md
│   │   ├── design-test.md
│   │   ├── bolt.md
│   │   ├── generate-api.md
│   │   └── generate-iac.md
│   ├── agents/                 # SubAgents（深い思考・対話型）
│   │   ├── intent-definer/     # インテント定義
│   │   ├── units-decomposer/   # ユニット分解
│   │   ├── domain-designer/    # ドメイン設計
│   │   ├── architecture-designer/  # アーキテクチャ設計
│   │   └── test-designer/      # テスト設計
│   └── skills/                 # Skills（コード生成型）
│       ├── api-generator/      # REST API生成
│       └── iac-generator/      # IaC生成
├── docs/                       # AI-DLC方法論ドキュメント
│   ├── AI-DLC_日本語訳.md      # 論文完全翻訳
│   ├── AI-DLC準拠状況.md       # 準拠率分析・実装状況
│   └── guides/                 # 開発ガイド
│       ├── getting-started.md
│       └── workflow.md
├── .gitignore
├── CLAUDE.md                   # AI-DLC開発ガイド
└── README.md                   # このファイル
```

**注**: `apps/`, `packages/`, `package.json`, `pnpm-workspace.yaml` は `/setup-aidlc` 実行時に自動生成されます。

### Skill/SubAgentアーキテクチャ

各コマンドは、再利用可能な**Skill**（コード生成）または**SubAgent**（深い思考・対話）を使用：

**Skills（コード生成型）**:
- `api-generator`: REST API実装生成（Hono RPC、OpenAPI）
- `iac-generator`: Infrastructure as Code生成（Terraform/Terragrunt）

**SubAgents（深い思考・対話型）**:
- `intent-definer`: インテント定義（要件明確化）
- `units-decomposer`: ユニット分解（DDD原則）
- `domain-designer`: ドメイン設計（エンティティ、集約等）
- `architecture-designer`: アーキテクチャ設計（NFR駆動、ADR生成）
- `test-designer`: テスト設計（TDD/BDD統合）

詳細は各Skill/SubAgentの `README.md` を参照してください。

## 📖 利用可能なスラッシュコマンド

### セットアップ
- `/setup-aidlc` - プロジェクトのAI-DLC環境セットアップ

### インセプションフェーズ（要件定義）
- `/intent` → `intent-definer` SubAgent - インテント定義（要件の明確化）
- `/units` → `units-decomposer` SubAgent - ユニット分解（疎結合・高凝集）

### コンストラクションフェーズ（設計・実装）
- `/design-domain` → `domain-designer` SubAgent - ドメイン設計（DDD戦術的設計）
- `/design-architecture` → `architecture-designer` SubAgent - アーキテクチャ設計（NFR駆動、ADR生成）
- `/design-test` → `test-designer` SubAgent - テスト設計（TDD/BDD統合）
- `/bolt` - 実装（高速反復サイクル）

### インフラ・API生成
- `/generate-api` → `api-generator` Skill - REST API実装生成（Hono RPC、OpenAPI）
- `/generate-iac` → `iac-generator` Skill - Infrastructure as Code生成（Terraform/Terragrunt）

### オペレーションフェーズ（未実装）
- `/operate` - 運用サポート（テレメトリ分析等）

詳細は [CLAUDE.md](CLAUDE.md) を参照してください。

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
