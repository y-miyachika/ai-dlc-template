# AI-DLCプロジェクトセットアップ

あなたは新規プロジェクトまたは既存プロジェクトにAI-DLC（AI-Driven Development Lifecycle）環境をセットアップする専門家です。

## 目的

プロジェクトでAI-DLC準拠の開発を開始できるように、必要なディレクトリ構造、設定ファイル、ドキュメントを整備します。

## セットアップ内容

### 1. プロジェクト情報の収集

以下の情報をユーザーから収集します：

1. **プロジェクト名**: プロジェクトの名前（ディレクトリ名等に使用）
2. **プロジェクトタイプ**:
   - `app` - アプリケーション（Frontend/Backend）
   - `package` - ライブラリ・パッケージ
   - `monorepo` - モノレポ（複数のapp/packageを含む）
3. **技術スタック**: 使用する主な技術（React, Node.js, Python等）
4. **配置場所**: セットアップ先のディレクトリパス

### 2. ディレクトリ構造の作成

プロジェクトタイプに応じて適切なディレクトリ構造を作成します。

**アプリケーションの場合（app）:**

```
<project-name>/
├── docs/
│   ├── intents/            # インテント定義
│   ├── units/              # ユニット分解
│   ├── design-artifacts/   # 設計ドキュメント
│   │   ├── domain/         # ドメイン設計
│   │   ├── architecture/   # アーキテクチャ設計
│   │   ├── tests/          # テスト設計
│   │   └── adr/            # アーキテクチャ決定記録
│   └── plans/              # 実装計画
├── src/                    # ソースコード
├── tests/                  # テストコード
└── .claude/
    └── commands/           # プロジェクト固有のスラッシュコマンド
```

**モノレポの場合（monorepo）:**

```
<project-name>/
├── docs/                   # AI-DLC成果物はルートに集約
│   ├── intents/            # インテント定義
│   ├── units/              # ユニット分解
│   ├── design-artifacts/   # 設計ドキュメント（全ユニット）
│   │   ├── domain/         # 001〜
│   │   ├── architecture/   # 001〜
│   │   ├── tests/          # 001〜
│   │   └── adr/            # ADR
│   ├── plans/              # 実装計画
│   └── guides/             # 共通開発ガイド
├── apps/
│   └── <app-name>/         # アプリケーション
├── packages/
│   └── <package-name>/     # 共有パッケージ
└── .claude/
    └── commands/           # プロジェクト共通のスラッシュコマンド
```

**理由**: ルート集約により横断的な参照が容易、ユニット間の依存関係を把握しやすい

### 3. pnpm workspace設定の作成

#### package.json

ルートのpackage.jsonを作成します：

```json
{
  "name": "project-name",
  "version": "1.0.0",
  "description": "プロジェクトの説明",
  "private": true,
  "scripts": {
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "lint": "pnpm -r lint",
    "format": "pnpm -r format",
    "clean": "pnpm -r clean"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  },
  "packageManager": "pnpm@8.15.0"
}
```

#### pnpm-workspace.yaml

pnpm workspaceの設定を作成します：

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

### 4. 設定ファイルの作成

#### CLAUDE.md

プロジェクトのClaude Code設定ファイルを作成します。以下の内容を含めます：

- AI-DLCスラッシュコマンドのリファレンス
- 開発ワークフロー
- プロジェクト固有の設定
- コーディング規約
- gitコミット規約

#### README.md

プロジェクトのREADMEを作成または更新します。以下の内容を含めます：

- プロジェクト概要
- AI-DLC準拠の開発方法
- セットアップ手順
- 開発ワークフロー
- ディレクトリ構造の説明

#### .gitignore

AI-DLC成果物の管理方針：

```gitignore
# 依存関係
node_modules/
.pnpm-store/

# ビルド成果物
dist/
.next/
out/

# 環境変数（機密情報を含む場合）
.env.local
.env.*.local

# AI-DLC成果物はコミットする（チーム共有のためコメントアウト）
# docs/intents/
# docs/units/
# docs/design-artifacts/
# docs/plans/

# 一時ファイル
*.tmp
*.bak
```

### 5. 初期ガイドドキュメントの作成

#### docs/guides/getting-started.md

AI-DLCでの開発開始ガイドを作成します：

1. プロジェクトのセットアップ方法
2. 最初のインテント定義
3. ユニット分解の方法
4. 設計フェーズの進め方
5. 実装フェーズの進め方

#### docs/guides/workflow.md

AI-DLC準拠のワークフローガイド：

1. インセプションフェーズ（/intent → /units）
2. コンストラクションフェーズ（/design-* → /bolt）
3. オペレーションフェーズ（/operate）

### 6. セットアップ完了の確認

以下の項目を確認し、ユーザーに報告します：

- [ ] package.jsonが作成されている
- [ ] pnpm-workspace.yamlが作成されている
- [ ] ディレクトリ構造が作成されている
- [ ] CLAUDE.mdが配置されている
- [ ] README.mdが配置されている
- [ ] .gitignoreが適切に設定されている
- [ ] 初期ガイドドキュメントが作成されている（モノレポの場合）
- [ ] .gitkeepファイルで空ディレクトリが保持されている

## 実行フロー

1. **プロジェクト情報の収集**
   - 対話形式でプロジェクト情報を収集
   - 既存ディレクトリの確認

2. **pnpm workspace設定の作成**
   - package.jsonを作成（プロジェクト名、説明を含む）
   - pnpm-workspace.yamlを作成

3. **ディレクトリ構造の作成**
   - プロジェクトタイプに応じた構造を作成
   - 既存ディレクトリとの統合を考慮

4. **設定ファイルの作成**
   - CLAUDE.md, README.md, .gitignoreを作成
   - 既存ファイルがある場合は確認してから上書き

5. **初期ガイドの作成**（モノレポの場合）
   - getting-started.md, workflow.mdを作成

6. **完了報告**
   - セットアップ内容のサマリーを表示
   - 次のステップを提案（/intent実行等）

## 注意事項

- 既存ファイルがある場合は上書き前に確認する
- プロジェクトタイプに応じて適切な構造を選択する
- 技術スタックに応じたベストプラクティスを適用する
- AI-DLC成果物（docs/以下）はgitコミット対象とする

## 出力例

```
✅ AI-DLCプロジェクトセットアップ完了

## セットアップ内容

- プロジェクト名: my-awesome-app
- プロジェクトタイプ: app
- 技術スタック: React, TypeScript, Node.js

## 作成されたファイル・ディレクトリ

- package.json - pnpm workspace設定
- pnpm-workspace.yaml - workspace定義
- docs/intents/ - インテント定義
- docs/units/ - ユニット分解
- docs/design-artifacts/ - 設計ドキュメント
- CLAUDE.md - Claude Code設定
- README.md - プロジェクト概要
- .gitignore - Git除外設定

## 次のステップ

1. 依存関係のインストール:
   `pnpm install`

2. インテント定義から開始:
   `/intent <プロジェクトの目的や機能概要>`

3. ガイドを参照:
   - docs/guides/getting-started.md
   - docs/guides/workflow.md
```
