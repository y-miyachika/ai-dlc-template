# AI-DLC Template - GitHub Copilot Instructions

## プロジェクト概要

このプロジェクトは、**AI-DLC（AI-Driven Development Lifecycle）**に準拠した開発フレームワークのテンプレートです。AWSが提唱するAI駆動型開発ライフサイクルの方法論に基づき、組織全体でAI-DLC準拠の開発を推進するための標準テンプレートを提供します。

---

## 応答言語

- **デフォルト言語: 日本語**
- すべての応答は日本語で行う
- ユーザーが明示的に英語を要求した場合のみ英語で応答

---

## AI-DLCエージェント

このプロジェクトでは、以下のカスタムエージェントが利用可能です。

### 利用可能なエージェント

**インセプションフェーズ（要件定義）**
- `@intent-definer` - インテント定義（要件明確化）
- `@units-decomposer` - ユニット分解（DDD原則）

**コンストラクションフェーズ（設計・実装）**
- `@domain-designer` - ドメイン設計（エンティティ、集約等）
- `@architecture-designer` - アーキテクチャ設計（NFR駆動、ADR生成）
- `@test-designer` - テスト設計（TDD/BDD統合）
- `@bolt` - 高速反復サイクル（計画→実装→テスト）

**インフラ・API生成**
- `@api-generator` - REST API実装生成（Hono RPC）
- `@iac-generator` - Infrastructure as Code生成（Terraform）
- `@deploy-generator` - デプロイ設定生成（GitHub Actions）

---

## 開発ワークフロー

### 新規プロジェクト開始時

1. **インテント定義から開始**
   ```
   @intent-definer <プロジェクト概要>
   ```

2. **ユニット分解**
   ```
   @units-decomposer <Intent番号>
   ```

3. **設計フェーズ**
   ```
   @domain-designer unit1
   @architecture-designer unit1
   @test-designer unit1
   ```

4. **実装フェーズ**
   ```
   @bolt unit1
   @api-generator unit1  # APIが必要な場合
   @iac-generator unit1  # インフラが必要な場合
   ```

---

## ドキュメント構造

### AI-DLCドキュメント配置

```
docs/
├── intents/              # インテント定義
├── units/                # ユニット分解
├── design-artifacts/     # 設計ドキュメント
│   ├── domain/           # ドメイン設計
│   ├── architecture/     # アーキテクチャ設計
│   ├── tests/            # テスト設計
│   └── adr/              # アーキテクチャ決定記録
└── plans/                # 実装計画
```

### ドキュメント命名規則

- Intent: `{Intent番号}_タイトル.md` 例: `002_ユーザー認証.md`
- Units: `{Intent番号}_ユニット分解.md`
- Design Artifacts: `{Intent番号}-{Unit番号}-名前.md` 例: `002-001-login-api.md`

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

**テンプレートバージョン**: 1.0.0
**GitHub Copilot対応**: 2025-02
