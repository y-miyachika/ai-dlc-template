# AI-DLC Template - GitHub Copilot Instructions

## プロジェクト概要

このプロジェクトは、**AI-DLC（AI-Driven Development Lifecycle）**に準拠した開発フレームワークのテンプレートです。AWSが提唱するAI駆動型開発ライフサイクルの方法論に基づき、組織全体でAI-DLC準拠の開発を推進するための標準テンプレートを提供します。

---

## 応答言語

- **デフォルト言語: 日本語**
- すべての応答は日本語で行う
- ユーザーが明示的に英語を要求した場合のみ英語で応答

---

## AI-DLCプロンプト

このプロジェクトでは、`.github/prompts/` にカスタムプロンプトが定義されています。

### 利用方法

VS Code / Visual Studio / JetBrains IDEで、チャット欄に `/` を入力してプロンプト名を選択：

```
/intent ユーザー認証機能の実装
/units 001
/design-domain unit1
/bolt unit1
```

### 利用可能なプロンプト

**インセプションフェーズ（要件定義）**
- `/intent` - インテント定義（要件明確化）
- `/units` - ユニット分解（DDD原則）
- `/setup-aidlc` - AI-DLCプロジェクトセットアップ

**コンストラクションフェーズ（設計・実装）**
- `/design-domain` - ドメイン設計（エンティティ、集約等）
- `/design-architecture` - アーキテクチャ設計（NFR駆動、ADR生成）
- `/design-test` - テスト設計（TDD/BDD統合）
- `/bolt` - 高速反復サイクル（計画→実装→テスト）

**インフラ・API生成**
- `/generate-api` - REST API実装生成（Hono RPC）
- `/generate-iac` - Infrastructure as Code生成（Terraform）
- `/generate-deploy` - デプロイ設定生成（GitHub Actions）

**ユーティリティ**
- `/progress` - プロジェクト/パッケージの現状確認
- `/sync-docs` - 設計ドキュメント同期チェック
- `/retro` - 会話の振り返りと改善提案

---

## 開発ワークフロー

### 新規プロジェクト開始時

1. **プロジェクトセットアップ**
   ```
   /setup-aidlc my-new-project
   ```

2. **インテント定義から開始**
   ```
   /intent <プロジェクト概要>
   ```

3. **ユニット分解**
   ```
   /units 001
   ```

4. **設計フェーズ**
   ```
   /design-domain unit1
   /design-architecture unit1
   /design-test unit1
   ```

5. **実装フェーズ**
   ```
   /bolt unit1
   /generate-api unit1
   /generate-iac unit1
   ```

---

## ドキュメント構造

### AI-DLCドキュメント配置

```
docs/
├── intents/                          # Intent階層（成果物集約）
│   └── {Intent番号}_{Intent名}/
│       ├── intent.md                 # インテント定義
│       ├── units.md                  # ユニット分解
│       ├── 000_shared/               # 共通（オプション）
│       └── {Unit番号}_{Unit名}/
│           ├── domain.md             # ドメイン設計
│           ├── architecture.md       # アーキテクチャ設計
│           ├── tests.md              # テスト設計
│           └── plan.md              # 実装計画
├── adr/                              # アーキテクチャ決定記録（横断的）
└── guides/                           # 開発ガイド
```

### ドキュメント命名規則

- Intent番号・Unit番号は3桁ゼロ埋め（001, 002, ...）
- Unit番号`000`はshared/共通拡張用
- ファイル名は固定: intent.md, units.md, domain.md, architecture.md, tests.md, plan.md
- 例: `docs/intents/002_ユーザー認証/001_login-api/domain.md`

---

## コーディング規約

### TypeScript

- strict modeを有効にする
- 型推論に頼りすぎず、明示的な型定義を優先
- any型の使用は禁止（unknownを使用）
- **classを使わない**: 関数ベースの設計を優先
  - 依存性注入は引数で渡す
  - 状態管理はクロージャまたはオブジェクトで

### Git

- コミットメッセージは日本語で記述
- コミット前に必ずテストを実行
- 機密情報（.env、credentials等）をコミットしない

---

## AI-DLC原則

このプロジェクトは以下のAI-DLC原則に従います：

1. **会話の逆転**: AIが質問を主導し、人間が回答する
2. **損失関数**: 曖昧さを早期解消し、下流の無駄な作業を防ぐ
3. **コンテキストメモリ**: 詳細な設計を永続化
4. **段階的詳細化**: インテント → ユニット → ドメイン → コード
5. **トレーサビリティ**: 成功指標とユーザーストーリーの対応を明確化

---

## 参考資料

- [AI-DLC日本語訳](../docs/AI-DLC_日本語訳.md)
- [AI-DLC準拠状況](../docs/AI-DLC準拠状況.md)
- [開始ガイド](../docs/guides/getting-started.md)
- [ワークフローガイド](../docs/guides/workflow.md)

---

## プロンプトファイルについて

`.github/prompts/*.prompt.md` ファイルは、GitHub Copilot の Prompt Files 機能を使用しています。

- **agent: agent** モードでファイル操作が可能
- 各プロンプトは詳細な指示を含み、AI-DLCワークフローを自動化
- Claude Code の `/command` と同様の体験を提供

詳細: [VS Code Prompt Files](https://code.visualstudio.com/docs/copilot/customization/prompt-files)

---

**テンプレートバージョン**: 1.0.0
**GitHub Copilot対応**: 2025-02（Prompt Files対応）
