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
│   ├── backlog/            # タスク定義（シンプル版）
│   ├── intents/            # インテント定義（AI-DLC準拠版）
│   ├── requirements/       # 要件定義
│   ├── units/             # ユニット分解
│   ├── design-artifacts/  # 設計ドキュメント
│   │   ├── domain/        # ドメイン設計
│   │   ├── architecture/  # アーキテクチャ設計
│   │   ├── tests/         # テスト設計
│   │   └── adr/          # アーキテクチャ決定記録
│   └── plans/            # 実装計画
├── src/                   # ソースコード
├── tests/                 # テストコード
└── .claude/
    └── commands/          # プロジェクト固有のスラッシュコマンド
```

**モノレポの場合（monorepo）:**

```
<project-name>/
├── docs/
│   ├── guides/           # 共通開発ガイド
│   └── README.md         # プロジェクト概要
├── apps/
│   └── <app-name>/
│       └── docs/         # アプリ固有のAI-DLC成果物
├── packages/
│   └── <package-name>/
│       └── docs/         # パッケージ固有のAI-DLC成果物
└── .claude/
    └── commands/          # プロジェクト共通のスラッシュコマンド
```

### 3. 設定ファイルの作成

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
# AI-DLC成果物はコミットする（チーム共有）
# docs/intents/
# docs/units/
# docs/design-artifacts/

# 一時ファイルは除外
*.tmp
*.bak
```

### 4. 初期ガイドドキュメントの作成

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

### 5. セットアップ完了の確認

以下の項目を確認し、ユーザーに報告します：

- [ ] ディレクトリ構造が作成されている
- [ ] CLAUDE.mdが配置されている
- [ ] README.mdが配置されている
- [ ] .gitignoreが適切に設定されている
- [ ] 初期ガイドドキュメントが作成されている
- [ ] .gitkeepファイルで空ディレクトリが保持されている

## 実行フロー

1. **プロジェクト情報の収集**
   - 対話形式でプロジェクト情報を収集
   - 既存ディレクトリの確認

2. **ディレクトリ構造の作成**
   - プロジェクトタイプに応じた構造を作成
   - 既存ディレクトリとの統合を考慮

3. **設定ファイルの作成**
   - CLAUDE.md, README.md, .gitignoreを作成
   - 既存ファイルがある場合は確認してから上書き

4. **初期ガイドの作成**
   - getting-started.md, workflow.mdを作成

5. **完了報告**
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

- docs/intents/ - インテント定義
- docs/units/ - ユニット分解
- docs/design-artifacts/ - 設計ドキュメント
- CLAUDE.md - Claude Code設定
- README.md - プロジェクト概要
- docs/guides/getting-started.md - 開始ガイド
- docs/guides/workflow.md - ワークフローガイド

## 次のステップ

1. インテント定義から開始:
   `/intent <プロジェクトの目的や機能概要>`

2. または、既存のBacklogから開始:
   `/backlog <タスク概要>`

3. ガイドを参照:
   - docs/guides/getting-started.md
   - docs/guides/workflow.md
```
