# Test Designer SubAgent

TDD（Test-Driven Development）とBDD（Behavior-Driven Development）を統合したテスト設計を実行するSubAgent

## 概要

BDD受入基準をテストケースに変換し、テストピラミッドを構築し、TDDサイクルで実装を駆動するテスト設計を作成します。

## 使い方

### 前提条件

- ユニット分解が完了している（`/units` 実行済み）
- ドメイン設計が完了している（`/design-domain` 実行済み）
- アーキテクチャ設計が完了している（`/design-architecture` 実行済み）
- `docs/intents/` または `docs/backlog/` に受入基準（Given/When/Then形式）が定義されている

### 実行

```bash
# AI-DLCテンプレートのスラッシュコマンドから
/design-test unit1

# または、Backlog番号 + ユニット番号
/design-test 046-unit1
```

### 生成されるファイル

```
docs/design-artifacts/tests/
└── {Backlog番号}_{ユニット名}_test_design.md
```

**例**: `docs/design-artifacts/tests/046_order-management_test_design.md`

## TDD/BDD統合の流れ

### 1. BDD受入基準をテストケースに変換

**受入基準（Given/When/Then）**:
```gherkin
Scenario: 注文の作成
  Given 顧客がログインしている
  And カートに商品が1つ以上ある
  When 注文を確定する
  Then 注文が作成される
  And 在庫が減少する
```

**変換後のテストケース**:
```typescript
it("Scenario: 注文の作成", async () => {
  // Given: 顧客がログインしている
  const customer = createCustomer({ id: "user123" });
  const cart = createCart({ items: [{ productId: "prod456", quantity: 2 }] });

  // When: 注文を確定する
  const order = await orderService.createOrder({
    customerId: customer.id,
    items: cart.items
  });

  // Then: 注文が作成される
  expect(order.id).toBeDefined();

  // And: 在庫が減少する
  const inventory = await inventoryRepository.findByProductId("prod456");
  expect(inventory.quantity).toBe(8); // 10 → 8
});
```

### 2. テストレベルの分類

| レベル | 目的 | 実行環境 | 対象環境 |
|--------|------|---------|---------|
| **Unit Test** | 単一関数・メソッドの動作検証 | ローカル、CI/CD | なし（モック） |
| **Integration Test** | コンポーネント間の連携検証 | ローカル、CI/CD | LocalStack/Testcontainers |
| **E2E Test** | エンドツーエンドのユーザーシナリオ検証 | **ローカル、CI/CD** | **デプロイ先環境（dev/staging）** |

**重要**: E2Eテストは**デプロイ先環境に対して実行**（ローカルからでもCI/CDからでもOK）

### 3. テストピラミッド

**理想的な比率**:
```
    E2E (10%)
   ────────
  Integration (20%)
  ──────────────────
      Unit (70%)
──────────────────────────
```

### 4. TDDサイクル

**Red → Green → Refactor**:

1. **Red（失敗するテストを書く）**:
```typescript
it("注文が作成される", async () => {
  const order = await orderService.createOrder({ /* ... */ });
  expect(order.id).toBeDefined();
});
```

2. **Green（最小限の実装で通す）**:
```typescript
class OrderService {
  async createOrder(params: CreateOrderParams): Promise<Order> {
    return { id: generateId() }; // 最小実装
  }
}
```

3. **Refactor（リファクタリング）**:
```typescript
class OrderService {
  async createOrder(params: CreateOrderParams): Promise<Order> {
    // バリデーション
    if (!params.items || params.items.length === 0) {
      throw new Error("Order must have at least one item");
    }

    // ビジネスロジック
    const order = new Order(generateId(), params.customerId, params.items);
    await this.orderRepository.save(order);

    // ドメインイベント発行
    await this.eventBus.publish(new OrderCreated(order.id));

    return order;
  }
}
```

## アーキテクチャ別のテスト戦略

### Lambda / Event-Driven

**Integration Test**:
- LocalStack DynamoDB/SQS + Lambda関数をローカル実行

**E2E Test**:
- デプロイ済みのLambda関数を実AWS APIで呼び出し
```typescript
await lambdaClient.invoke({
  FunctionName: 'my-app-dev-orchestrator'
})
```

### REST API

**Integration Test**:
- ローカルサーバー起動 + In-Memory DB

**E2E Test**:
- デプロイ済みのAPIエンドポイントにHTTPリクエスト
```typescript
await fetch('https://dev-api.example.com/users')
```

### Web Application

**Integration Test**:
- ローカルサーバー + Playwright（localhost）

**E2E Test**:
- デプロイ済みのサイトにPlaywrightでアクセス
```typescript
await page.goto('https://dev.example.com')
```

### Batch / CLI

**Integration Test**:
- ローカル実行 + Testcontainers

**E2E Test**:
- デプロイ済みのバッチジョブを実S3/DBで実行

### GraphQL API

**Integration Test**:
- ローカルApolloサーバー + In-Memory DB

**E2E Test**:
- デプロイ済みのGraphQL Endpointにクエリ

### Microservices

**Integration Test**:
- ローカルQueue Emulator + 各サービス

**E2E Test**:
- デプロイ済みの全サービス + 実Message Queue

## モック・フィクスチャ設計

### モック戦略

| 依存 | モック方法 | 理由 |
|-----|----------|------|
| HTTP API | `vi.mock()` でモック | 実APIは遅い、レート制限あり |
| データベース | LocalStack/Testcontainers | 実DBとの統合テストが必要 |
| ファイルシステム | テスト用ディレクトリ | 実ファイル操作が必要 |
| 環境変数 | `vi.stubEnv()` | 環境に依存しないテスト |
| 時刻 | `vi.setSystemTime()` | 決定的なテストのため |

### フィクスチャファイル

```
tests/
├── fixtures/
│   ├── valid-order.json       # 正常な注文データ
│   ├── invalid-order.json     # 不正な注文データ
│   └── customer-with-cart.json # 顧客+カートデータ
└── helpers/
    └── loadFixture.ts         # フィクスチャ読み込みヘルパー
```

## テストカバレッジ目標

| 指標 | 目標値 | 理由 |
|-----|--------|------|
| **Line Coverage** | 80%以上 | 主要ロジックをカバー |
| **Branch Coverage** | 70%以上 | 条件分岐を網羅 |
| **Function Coverage** | 90%以上 | すべての関数をテスト |

**未カバーを許容する箇所**:
- ロギング処理
- エラーハンドリングの一部（実現困難なケース）
- デバッグ用コード

## E2EテストのCI/CD統合

E2Eテストはデプロイ後に実行：

```yaml
# GitHub Actions
jobs:
  unit-and-integration:
    runs-on: ubuntu-latest
    steps:
      - run: pnpm test:unit
      - run: pnpm test:integration

  deploy-dev:
    needs: unit-and-integration
    steps:
      - run: terraform apply

  e2e-dev:
    needs: deploy-dev
    env:
      TEST_ENV: dev
      API_URL: https://dev-api.example.com
    steps:
      - run: until curl -f $API_URL/health; do sleep 5; done
      - run: pnpm test:e2e:dev
```

## ベストプラクティス vs アンチパターン

### ✅ ベストプラクティス

1. **Given/When/Then コメント**:
```typescript
it("Scenario: 注文の作成", async () => {
  // Given: 顧客がログインしている
  // When: 注文を確定する
  // Then: 注文が作成される
});
```

2. **Arrange-Act-Assert パターン**:
```typescript
it("注文が作成される", async () => {
  // Arrange（Given）
  const customer = createCustomer();

  // Act（When）
  const order = await orderService.createOrder({ customerId: customer.id });

  // Assert（Then）
  expect(order.id).toBeDefined();
});
```

3. **テストケース名は具体的に**:
- ✅ "100階層のノードを5000トークン以内で取得できる"
- ❌ "test1", "動作確認"

### ❌ アンチパターン

1. **テスト後に実装**:
- 実装が先だとテストが形骸化
- TDDの利点（設計の改善）を失う

2. **E2Eテストの過剰使用**:
- E2Eテストは遅い、コスト高
- テストピラミッドを守る（70% Unit, 20% Integration, 10% E2E）

3. **モックの乱用**:
- 何でもモックすると、実際の連携をテストしない
- Integration テストで実DB/APIを使う

## 出力例

### テスト設計ドキュメント

**ファイル**: `docs/design-artifacts/tests/046_order-management_test_design.md`

```markdown
# テスト設計: order-management

**元のユニット定義**: `docs/units/046_units.md`
**ドメイン設計**: `docs/design-artifacts/domain/046_order-management_domain.md`
**アーキテクチャ設計**: `docs/design-artifacts/architecture/046_order-management_architecture.md`

---

## アーキテクチャタイプ

**判定結果**: REST API

---

## テストケース一覧

### テストケース1: 注文の作成

**前提条件（Setup）**:
- 顧客がログインしている（userId: "user123"）
- カートに商品が1つ以上ある（productId: "prod456", quantity: 2）

**実行（Exercise）**:
- `OrderService.createOrder()` を実行

**検証（Verify）**:
- 注文が作成される
- 在庫が減少する

**テストレベル**: E2E Test（デプロイ先実行）

（以下略）
```

## AI-DLC原則との対応

| 原則 | 実装方法 |
|-----|---------|
| **損失関数** | テストで早期に欠陥を検出（実装前に） |
| **段階的詳細化** | BDD受入基準 → テストケース → 実装 |
| **コンテキストメモリ** | テスト設計ドキュメントを永続化 |
| **トレーサビリティ** | 受入基準 → テストケース → 実装コードの紐づけ |
| **会話の逆転** | テストが仕様を定義（実装が後追い） |

## 他プロジェクトでの利用

このSubAgentは、AI-DLCテンプレート以外のプロジェクトでも利用可能です：

1. `.claude/agents/test-designer/` をコピー
2. プロジェクトに配置
3. SubAgentを呼び出し

## バージョン履歴

- **1.0.0** (2025-11-19): 初版リリース
  - BDD受入基準のテストケース変換
  - アーキテクチャタイプ別テスト戦略（6パターン対応）
  - テストピラミッド構築
  - TDDサイクル計画
  - E2EテストのCI/CD統合

## ライセンス

MIT
