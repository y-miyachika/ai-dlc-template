# ドメイン設計（AI-DLC準拠）

このコマンドは**Domain Designer SubAgent**を使用して、ユニットのドメインモデルをDDD原則に基づいて設計します。

## SubAgentについて

`domain-designer` SubAgentは、DDD（Domain-Driven Design）の戦術的設計パターンを適用し、ビジネスロジックを明確にモデル化する専門SubAgentです。

**DDD戦術的設計パターン**: エンティティ、値オブジェクト、集約、ドメインイベント、リポジトリ、ドメインサービス、ファクトリ

詳細は `.claude/agents/domain-designer/README.md` を参照してください。

## 前提条件

**このコマンドは `/units` の後に実行してください**

- `/units` でユニット分解が完了している
- `docs/units/` にユニット定義が存在する
- 元のBacklogまたはIntentが存在する

## 入力内容

{{ARGS}}

## SubAgent起動

**既存コードベースがある場合の推奨**:
ドメイン分析の前に、`Explore` subagent_type（Task tool）を使って既存のドメインモデルやビジネスロジックを探索することを推奨します。
これにより、既存の命名規則やパターンとの整合性を保てます。

以下を実行します：

1. **コンテキストの読み込み**: docs/units/ からユニット定義を読み込み
2. **ドメイン分析**: コアドメイン、ユビキタス言語、境界コンテキストを特定
3. **DDD戦術的設計パターンの適用**: エンティティ、値オブジェクト、集約等を設計
4. **ドメインモデルの定義**: TypeScriptインターフェースで明確に定義
5. **ユーザー承認**: 設計案を提示し、承認を得る
6. **ファイル保存**: `docs/design-artifacts/domain/{番号}_{ユニット名}_domain.md` に保存

---

**実行する処理**:

引数として受け取ったユニット名をもとに、Domain Designer SubAgentを起動します。

**ユニット名**: {{ARGS}}

**ユニット定義パス**: `docs/units/`

**出力先**: `docs/design-artifacts/domain/`

---

## SubAgent処理の詳細

SubAgentは `.claude/agents/domain-designer/prompt.md` に定義された手順に従って処理を実行します。

**主要な処理**:

1. コンテキストの読み込み（ユニット定義、Backlog、NFR）
2. ドメイン分析（コアドメイン、ユビキタス言語、境界コンテキスト）
3. DDD戦術的設計パターンの適用
   - エンティティ（Entity）: 一意の識別子を持つオブジェクト
   - 値オブジェクト（Value Object）: 不変な値
   - 集約（Aggregate）: トランザクション境界
   - ドメインイベント（Domain Event）: ビジネス上の重要な出来事
   - リポジトリ（Repository）: 永続化の抽象化
   - ドメインサービス（Domain Service）: 複数集約にまたがるロジック
   - ファクトリ（Factory）: 複雑な集約の生成
4. ドメインモデルの定義（TypeScriptインターフェース）
5. ユビキタス言語の定義
6. ドメインルールの一覧化
7. ユーザーストーリーとの対応

**設計詳細**: `.claude/agents/domain-designer/prompt.md` を参照

---

## DDD戦術的設計パターン

### エンティティ（Entity）

一意の識別子（ID）を持つオブジェクト：

```typescript
interface User {
  id: string;
  email: Email;
  name: string;
}
```

### 値オブジェクト（Value Object）

不変な値：

```typescript
interface Email {
  readonly value: string;
}
```

### 集約（Aggregate）

トランザクション境界：

```typescript
class Order {  // 集約ルート
  private items: OrderItem[];
  public addItem(product: Product, quantity: number): Result {
    // ビジネスルールの検証
    // 不変条件のチェック
    // ドメインイベントの発行
  }
}
```

### ドメインイベント（Domain Event）

ビジネス上の重要な出来事：

```typescript
interface OrderPlaced {
  eventId: string;
  occurredAt: Date;
  orderId: string;
}
```

### リポジトリ（Repository）

永続化の抽象化：

```typescript
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(orderId: string): Promise<Order | null>;
}
```

---

## 生成されるファイル

- `docs/design-artifacts/domain/{Backlog番号}_{ユニット名}_domain.md` - ドメイン設計ドキュメント

**ファイル名例**: `046_order-management_domain.md`

---

## ベストプラクティス vs アンチパターン

### ✅ ベストプラクティス

- **小さな集約**: 1つの集約は1つのトランザクション境界
- **不変条件の明確化**: 集約が保証すべき整合性ルールを明示
- **ユビキタス言語の使用**: ビジネス用語をコードに反映
- **値オブジェクトの活用**: バリデーションを1箇所に集約
- **ドメインイベントの発行**: ユニット間の疎結合な通信

### ❌ アンチパターン

- **巨大な集約**: パフォーマンス問題、トランザクション競合の原因
- **貧血ドメインモデル**: ビジネスロジックがサービス層に流出
- **技術用語の使用**: ドメイン層に「DTO」「DAO」などの技術用語
- **集約を超えた外部キー参照**: 集約間は ID 参照のみ
- **ドメインサービスの過度な使用**: エンティティや値オブジェクトで表現できないか先に検討

---

## 実行後の次のステップ

```bash
# アーキテクチャ設計（NFR考慮）
/design-architecture unit1

# テスト設計
/design-test unit1

# ボルト実行
/bolt unit1
```

---

## 注意事項

1. **インフラストラクチャの詳細は含めない**: ドメイン層はビジネスロジックのみ
2. **ユビキタス言語を厳守**: ドメインエキスパートと同じ用語を使用
3. **過度な抽象化を避ける**: YAGNI（You Aren't Gonna Need It）原則
4. **集約の境界を慎重に**: 大きすぎる集約はパフォーマンス問題の原因
5. **不変条件を明確に**: 集約が保証すべき整合性ルールを明示

---

**SubAgent Version**: 1.0.0
**SubAgent Location**: `.claude/agents/domain-designer/`
**作成日**: 2025-11-19
**AI-DLC準拠**: コンストラクションフェーズ（ドメイン設計）
