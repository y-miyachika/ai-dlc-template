---
name: setup-aidlc
description: AI-DLCプロジェクトセットアップ
agent: agent
argument-hint: プロジェクト名（省略時は対話形式）
---

# AI-DLCプロジェクトセットアップ

あなたは新規プロジェクトまたは既存プロジェクトにAI-DLC環境をセットアップする専門家です。

## 目的

プロジェクトでAI-DLC準拠の開発を開始できるように、必要なディレクトリ構造、設定ファイル、ドキュメントを整備します。

## 実行手順

### 1. プロジェクト情報の収集

以下を対話形式で収集：

1. **プロジェクト名**: ディレクトリ名等に使用
2. **プロジェクトタイプ**:
   - `app` - アプリケーション
   - `package` - ライブラリ
   - `monorepo` - モノレポ
3. **技術スタック**: React, Node.js, Python等
4. **配置場所**: セットアップ先ディレクトリ

### 2. ディレクトリ構造の作成

**アプリケーション（app）:**
```
<project-name>/
├── docs/
│   ├── intents/
│   ├── units/
│   ├── design-artifacts/
│   │   ├── domain/
│   │   ├── architecture/
│   │   ├── tests/
│   │   └── adr/
│   └── plans/
├── src/
└── tests/
```

**モノレポ（monorepo）:**
```
<project-name>/
├── docs/              # AI-DLC成果物はルートに集約
│   ├── intents/
│   ├── units/
│   ├── design-artifacts/
│   └── plans/
├── apps/
├── packages/
└── pnpm-workspace.yaml
```

### 3. 設定ファイルの作成

- `package.json` - pnpm workspace設定
- `pnpm-workspace.yaml` - workspace定義
- `CLAUDE.md` - Claude Code設定
- `README.md` - プロジェクト概要
- `.gitignore` - Git除外設定

### 4. セットアップ完了の確認

チェックリスト：
- [ ] package.jsonが作成されている
- [ ] pnpm-workspace.yamlが作成されている
- [ ] ディレクトリ構造が作成されている
- [ ] CLAUDE.mdが配置されている
- [ ] README.mdが配置されている
- [ ] .gitignoreが適切に設定されている
- [ ] .gitkeepファイルで空ディレクトリが保持されている

## 完了報告

```
✅ AI-DLCプロジェクトセットアップ完了

## セットアップ内容
- プロジェクト名: {name}
- プロジェクトタイプ: {type}
- 技術スタック: {stack}

## 次のステップ

1. 依存関係のインストール:
   pnpm install

2. インテント定義から開始:
   /intent <プロジェクトの目的>
```

## 注意事項

- 既存ファイルがある場合は上書き前に確認
- AI-DLC成果物（docs/以下）はgitコミット対象
