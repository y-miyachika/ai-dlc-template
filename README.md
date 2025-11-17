# AI-DLC Template

**AI駆動型開発ライフサイクル（AI-DLC）** のプロジェクトテンプレート

このテンプレートを使って、AI-DLC方法論に基づいた新規プロジェクトを開始できます。

## 🎯 このテンプレートについて

AI-DLCは、AWSが提唱する**AI駆動型開発ライフサイクル**の方法論です。このテンプレートは、AI-DLCに準拠した開発を組織全体で推進するための標準フレームワークを提供します。

### 主な特徴

- **AI-DLC準拠率 65%以上**: AWS論文の付録A（プロンプト）に対する準拠率
- **3つのフェーズをカバー**: Inception（要件定義）、Construction（設計・実装）、Operations（運用）
- **スラッシュコマンド**: Claude Code用の10個以上の開発支援コマンド
- **モノレポ構成**: apps/ + packages/ のモダンな構成（pnpm workspace）
- **マルチツール対応**: Claude Code, GitHub Copilot, Amazon Q Developer
- **TDD/BDD統合**: テストファーストの開発サイクル
- **DDD原則**: ドメイン駆動設計に基づく設計フェーズ

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

### 2. セットアップ

```bash
# 依存関係インストール
pnpm install

# AI-DLCフォルダ構造の初期化
/setup-aidlc
```

### 3. 開発開始

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
```

## 📁 プロジェクト構造

```
ai-dlc-template/
├── README.md                    # このファイル
├── CLAUDE.md                    # AI-DLC開発ガイド
├── .claude/commands/            # スラッシュコマンド群
├── docs/                        # AI-DLCドキュメント
│   ├── AI-DLC_日本語訳.md
│   └── guides/
├── templates/                   # テンプレートファイル
├── apps/                        # アプリケーション（ユーザーが作成）
└── packages/                    # 共通パッケージ（ユーザーが作成）
```

## 📖 利用可能なスラッシュコマンド

### セットアップ
- `/setup-aidlc` - プロジェクトのAI-DLC環境セットアップ

### インセプションフェーズ（要件定義）
- `/intent` - インテント定義（要件の明確化）
- `/units` - ユニット分解（疎結合・高凝集）

### コンストラクションフェーズ（設計・実装）
- `/design-domain` - ドメイン設計（DDD原則）
- `/design-architecture` - アーキテクチャ設計（NFR考慮）
- `/design-test` - テスト設計（TDD/BDD統合）
- `/bolt` - 実装（高速反復サイクル）

### インフラ・API生成（未実装）
- `/generate-iac` - Infrastructure as Code生成
- `/generate-api` - REST API実装生成

### オペレーションフェーズ（未実装）
- `/operate` - 運用サポート（テレメトリ分析等）

詳細は [CLAUDE.md](CLAUDE.md) を参照してください。

## 🎓 AI-DLCについて

### 学習リソース

- [AI-DLC日本語訳](docs/AI-DLC_日本語訳.md) - AWS論文の完全な翻訳
- [AI-DLC付録A対比](docs/AI-DLC付録A対比.md) - プロンプトと実装の対比
- [開始ガイド](docs/guides/getting-started.md) - 最初のステップ
- [ワークフローガイド](docs/guides/workflow.md) - 開発フローの詳細

### 参考資料

- [AI-DLC論文（原文）](https://github.com/aws-samples/ai-driven-development-lifecycle)（英語）

## 📄 ライセンス

MIT License

---

**作成日**: 2025-11-17
**バージョン**: 1.0.0
