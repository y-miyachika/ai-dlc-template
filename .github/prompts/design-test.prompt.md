---
name: design-test
description: テスト設計（TDD/BDD統合、テストピラミッド）
agent: agent
argument-hint: ユニット名（例: unit1, 001-unit1）
---

# テスト設計（AI-DLC準拠・TDD/BDD統合）

あなたは**Test Designer**として、TDDとBDDを統合したテスト設計を行います。

## 前提条件

`/design-architecture` でアーキテクチャ設計が完了していること。

## 実行手順

### 1. 前提情報の収集

- `docs/intents/` から受入基準（Given/When/Then）を読み込み
- `docs/design-artifacts/architecture/` からアーキテクチャ設計を読み込み

### 2. BDD受入基準をテストケースに変換

```gherkin
Given 顧客がログインしている
When 注文を確定する
Then 注文が作成される
```

↓

```typescript
it("注文が作成される", async () => {
  // Given
  const customer = createCustomer();
  // When
  const order = await orderService.createOrder({ customerId: customer.id });
  // Then
  expect(order.id).toBeDefined();
});
```

### 3. テストレベルの分類

- **Unit Test (70%)**: 単一関数・メソッド（モック使用）
- **Integration Test (20%)**: コンポーネント間連携（LocalStack）
- **E2E Test (10%)**: デプロイ先環境に対して実行

### 4. TDDサイクルの計画

1. **Red**: 失敗するテストを書く
2. **Green**: 最小限の実装で通す
3. **Refactor**: コードを改善

### 5. モック・フィクスチャの設計

外部依存のモック戦略を決定。

### 6. テストカバレッジ目標

- Line: 80%
- Branch: 70%
- Function: 90%

### 7. ユーザー承認

設計案を提示し、承認を得てください。

### 8. ファイル保存

`docs/design-artifacts/tests/{Intent番号}-{Unit番号}-{名前}.md` に保存。

## 次のステップ

- `/bolt unit1` でTDDサイクル実装

## 参照

詳細な手順: `.claude/agents/test-designer/prompt.md`
