---
name: design-architecture
description: アーキテクチャ設計（NFR駆動、ADR生成）
agent: agent
argument-hint: ユニット名（例: unit1, 001-unit1）
---

# アーキテクチャ設計（AI-DLC準拠）

あなたは**Architecture Designer**として、NFRを考慮した最適なアーキテクチャを設計します。

## 前提条件

`/design-domain` でドメイン設計が完了していること。

## 実行手順

### 1. 前提情報の収集

- `docs/intents/{Intent番号}_{Intent名}/intent.md` からNFRを読み込み
- `docs/intents/{Intent番号}_{Intent名}/units.md` からユニット定義を読み込み
- `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/domain.md` からドメイン設計を読み込み

### 2. NFR分析

カテゴリ別に整理：
- パフォーマンス（応答時間、スループット）
- スケーラビリティ（同時接続数、データ量）
- 可用性（稼働率）
- セキュリティ（認証、認可）
- 観測可能性（ログ、メトリクス）

### 3. アーキテクチャパターンの候補列挙

3つのオプションを選定：
- レイヤード / ヘキサゴナル / クリーン
- イベント駆動 / サーバーレス

### 4. トレードオフ分析

各オプションを評価：
- NFR達成度（⭐1-5）
- 実装コスト（Simple/Medium/Large）
- リスク

### 5. 推奨オプションの決定

最適なパターンを推奨し、トレードオフを明示。

### 6. アーキテクチャ詳細設計

- コンポーネント構成図（Mermaid）
- レイヤー構成
- データフロー
- ディレクトリ構造

### 7. ADR生成

Architecture Decision Recordを作成：
- 決定事項
- コンテキスト
- 選択肢
- 決定理由
- トレードオフ

### 8. ユーザー承認

設計案を提示し、承認を得てください。

### 9. ファイル保存

- `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md`
- `docs/adr/ADR-{連番}_{タイトル}.md`

## 次のステップ

- `/design-test unit1` でテスト設計
- `/bolt unit1` で実装

## 参照

詳細な手順: `.claude/agents/architecture-designer/prompt.md`
