# Domain Designer SubAgent

DDD（Domain-Driven Design）原則に基づき、ユニットのドメイン設計を実行するSubAgent

## 概要

ビジネスロジックを明確にモデル化し、エンティティ、値オブジェクト、集約、ドメインイベント、リポジトリ、ドメインサービス、ファクトリを適切に設計します。

## 使い方

### 前提条件

- ユニット分解が完了している（`/units` 実行済み）
- `docs/intents/{Intent番号}_{Intent名}/units.md` にユニット定義が存在する
- 元のBacklogまたはIntentが存在する

### 実行

```bash
# AI-DLCテンプレートのスラッシュコマンドから
/design-domain unit1

# または、Intent番号 + Unit番号
/design-domain 002-001
```

### 生成されるファイル

```
docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/
└── domain.md
```

**例**: `docs/intents/002_ユーザー認証/001_unit1/domain.md`

## DDD戦術的設計パターン

このSubAgentが適用する主要なパターン：

### 1. エンティティ（Entity）

**定義**: 一意の識別子（ID）を持つオブジェクト

**特徴**:
- ライフサイクルを通じて追跡される
- 属性が変わっても、同じエンティティとして扱われる

**例**:
```typescript
interface User {
  id: string;
  email: Email;
  name: string;
  createdAt: Date;
}
```

### 2. 値オブジェクト（Value Object）

**定義**: 識別子を持たず、属性の値で区別される不変オブジェクト

**特徴**:
- 不変（Immutable）
- 等価性は属性の値で判断
- バリデーションを1箇所に集約

**例**:
```typescript
interface Email {
  readonly value: string;
}
```

### 3. 集約（Aggregate）

**定義**: エンティティと値オブジェクトのクラスタ、トランザクション境界を定義

**特徴**:
- 集約ルート（Aggregate Root）がすべての変更を管理
- 不変条件（Invariants）を保証
- 1つの集約 = 1つのトランザクション

**例**:
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

### 4. ドメインイベント（Domain Event）

**定義**: ドメイン内で発生した重要な出来事

**特徴**:
- 過去形で命名（例: `OrderPlaced`, `UserRegistered`）
- 他の集約やユニットへの通知に使用
- イベント駆動アーキテクチャの基盤

**例**:
```typescript
interface OrderPlaced {
  eventId: string;
  occurredAt: Date;
  orderId: string;
  customerId: string;
}
```

### 5. リポジトリ（Repository）

**定義**: 集約の永続化とクエリのインターフェース

**特徴**:
- データアクセスの抽象化
- ドメイン層とインフラ層の境界
- 集約単位で定義

**例**:
```typescript
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(orderId: string): Promise<Order | null>;
}
```

### 6. ドメインサービス（Domain Service）

**定義**: 特定のエンティティや値オブジェクトに属さないビジネスロジック

**特徴**:
- ステートレスな操作
- 複数の集約にまたがるビジネスロジック

**例**:
```typescript
interface InventoryCheckService {
  checkAvailability(productId: string, quantity: number): Promise<boolean>;
}
```

### 7. ファクトリ（Factory）

**定義**: 複雑な集約の生成ロジック

**特徴**:
- 生成時のバリデーション
- 不変条件を満たした状態で生成

**例**:
```typescript
class OrderFactory {
  static create(customerId: string, items: OrderItemInput[]): Order {
    // バリデーション、初期状態設定、不変条件チェック
  }
}
```

## ドメイン設計の原則

### ユビキタス言語（Ubiquitous Language）

**原則**: ドメインエキスパートと開発者が共有する用語を使う

**例**:
- ❌ `data`, `record`, `info`
- ✅ `Order`, `Customer`, `Product`

### 境界コンテキスト（Bounded Context）

**原則**: ドメインモデルの境界を明確にする

**例**:
- `unit1` (注文管理) と `unit2` (在庫管理) は別の境界コンテキスト
- 同じ「Product」という用語でも、意味が異なる可能性がある

### 不変条件（Invariants）

**原則**: 集約が保証すべき整合性ルールを明示する

**例**:
- 注文の合計金額は常に明細の合計と一致する
- 在庫数は負にならない

## ベストプラクティス vs アンチパターン

### ✅ ベストプラクティス

1. **小さな集約**
   - 1つの集約は1つのトランザクション境界
   - パフォーマンス最適化

2. **不変条件の明確化**
   - 集約が保証すべき整合性ルールを明示
   - バグの早期発見

3. **ユビキタス言語の使用**
   - ビジネス用語をコードに反映
   - ドメインエキスパートとのコミュニケーション向上

4. **値オブジェクトの活用**
   - バリデーションを1箇所に集約
   - 不正な値を防ぐ

5. **ドメインイベントの発行**
   - ユニット間の疎結合な通信
   - イベント駆動アーキテクチャ

### ❌ アンチパターン

1. **巨大な集約**
   - パフォーマンス問題の原因
   - トランザクション競合
   - 解決策: 集約を分割、結果整合性を使う

2. **貧血ドメインモデル**
   - ビジネスロジックがサービス層に流出
   - ドメインモデルがデータ構造だけになる
   - 解決策: エンティティに振る舞いを持たせる

3. **技術用語の使用**
   - ドメイン層に「DTO」「DAO」などの技術用語
   - ビジネスの意図が不明確
   - 解決策: ビジネス用語を使う

4. **集約を超えた外部キー参照**
   - 集約の境界が曖昧
   - トランザクション境界の混乱
   - 解決策: 集約間は ID 参照のみ

5. **ドメインサービスの過度な使用**
   - エンティティや値オブジェクトが貧弱になる
   - ビジネスロジックが分散
   - 解決策: エンティティや値オブジェクトで表現できないか先に検討

## 出力例

### ドメイン設計ドキュメント

**ファイル**: `docs/intents/046_order-management/001_order-management/domain.md`

```markdown
# ドメイン設計: order-management

**元のユニット定義**: `docs/intents/046_order-management/units.md`
**対応するユーザーストーリー**: US-001, US-002, US-003

---

## ドメインの概要

このユニットが扱うビジネスドメインの概要を説明：

- **コアドメイン**: 注文管理（ECサイトの中核機能）
- **ユビキタス言語**: Order, OrderItem, Customer, Product
- **境界コンテキスト**: 在庫管理、決済管理とは分離

---

## 1. エンティティ（Entities）

### エンティティ1: Order

**責務**:
- 顧客の注文を表現
- 注文明細の管理
- 注文状態の管理

**属性**:
```typescript
interface Order {
  id: string;
  customerId: string;
  items: OrderItem[];
  status: OrderStatus;
  total: Money;
  createdAt: Date;
  updatedAt: Date;
}
```

（以下略）
```

## 他プロジェクトでの利用

このSubAgentは、AI-DLCテンプレート以外のプロジェクトでも利用可能です：

1. `.claude/agents/domain-designer/` をコピー
2. プロジェクトに配置
3. SubAgentを呼び出し

## バージョン履歴

- **1.0.0** (2025-11-19): 初版リリース
  - DDD戦術的設計パターンの適用
  - エンティティ、値オブジェクト、集約、ドメインイベント、リポジトリ、ドメインサービス、ファクトリの設計
  - ユビキタス言語の定義
  - ベストプラクティス vs アンチパターンのガイド

## 品質基準

出力品質を確保するため、[品質チェックリスト](../quality-checklist.md)を参照してください。

## ライセンス

MIT
