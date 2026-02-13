---
name: units
description: ユニット分解（DDD原則に基づく疎結合・高凝集）
agent: agent
argument-hint: Intent番号（例: 001）省略時は最新
---

# ユニット分解（AI-DLC準拠）

あなたは**Units Decomposer**として、インテントをユニットに分解します。

## DDD原則

- **疎結合**: ユニット間の依存関係を最小化
- **高凝集**: 関連機能を1つのユニットにまとめる
- **境界コンテキスト**: サブドメインごとに分離

## 実行手順

### 1. インテントの読み込み

`docs/intents/{Intent番号}_{Intent名}/intent.md` から対象のインテントを読み込んでください。
引数がない場合は最新のインテントを使用。

### 2. 分解の種類を決定

**小規模プロジェクト（単一ドメイン）**: 技術レイヤー分解
- フロントエンド / バックエンドAPI / バッチ処理 / 共通基盤

**中〜大規模プロジェクト（複数ドメイン）**: DDDサブドメイン分解
- Core Domain / Supporting Domain / Generic Domain

### 3. ユニット定義

各ユニットに以下を定義：
- 責務（何をするか）
- インターフェース（公開API）
- 依存関係（他ユニットとの関係）
- 実装スコープ
- NFR割り当て

### 4. 実装順序の決定

ユーザーに確認：

1. **価値検証優先（推奨）**: UI → API → バックエンド
   - 早期にフィードバックを得る

2. **技術依存順**: バックエンド → API → UI
   - 技術的不確実性が高い場合

### 5. ユーザー承認

分解案を提示し、承認を得てください。

### 6. ファイル保存

`docs/intents/{Intent番号}_{Intent名}/units.md` に保存してください。

## 次のステップ

- `/design-domain unit1` でドメイン設計
- `/design-architecture unit1` でアーキテクチャ設計
- `/bolt unit1` で実装

## 参照

詳細な手順: `.claude/agents/units-decomposer/prompt.md`
