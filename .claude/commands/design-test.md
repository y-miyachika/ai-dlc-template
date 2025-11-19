# テスト設計（AI-DLC準拠・TDD/BDD統合）

このコマンドは**Test Designer SubAgent**を使用して、TDDとBDDを統合したテスト設計を行います。

## SubAgentについて

`test-designer` SubAgentは、BDD受入基準をテストケースに変換し、テストピラミッドを構築し、TDDサイクルで実装を駆動するテスト設計を実行する専門SubAgentです。

**テスト設計の原則**: テストファースト、Red-Green-Refactor、テストピラミッド

詳細は `.claude/agents/test-designer/README.md` を参照してください。

## 前提条件

**このコマンドは `/design-architecture` の後に実行してください**

- `/design-architecture` でアーキテクチャ設計が完了している
- `docs/design-artifacts/architecture/` にアーキテクチャ設計が存在する
- `docs/intents/` または `docs/backlog/` に受入基準（Given/When/Then形式）が定義されている

## 入力内容

{{ARGS}}

## SubAgent起動

以下を実行します：

1. **前提情報の収集**: Backlog/Intent、ユニット定義、ドメイン設計、アーキテクチャ設計を読み込み
2. **BDD受入基準をテストケースに変換**: Given/When/Then形式をテストケースに変換
3. **アーキテクチャタイプの判定**: Lambda/REST API/Web App等を判定
4. **テストレベルの分類**: Unit/Integration/E2Eに分類
5. **テストピラミッドの構成**: 70% Unit, 20% Integration, 10% E2Eの比率
6. **TDDサイクルの計画**: Red-Green-Refactorサイクルを計画
7. **モック・フィクスチャの設計**: 外部依存のモック戦略を決定
8. **テストカバレッジ目標の設定**: Line/Branch/Function Coverage目標
9. **テスト実装の優先順位**: High/Medium/Lowで優先順位付け
10. **E2EテストのCI/CD統合**: デプロイ後のE2E実行フロー
11. **ユーザー承認**: 設計案を提示し、承認を得る
12. **ファイル保存**: テスト設計ドキュメントを保存

---

**実行する処理**:

引数として受け取ったユニット名をもとに、Test Designer SubAgentを起動します。

**ユニット名**: {{ARGS}}

**アーキテクチャ設計パス**: `docs/design-artifacts/architecture/`

**出力先**: `docs/design-artifacts/tests/`

---

## SubAgent処理の詳細

SubAgentは `.claude/agents/test-designer/prompt.md` に定義された手順に従って処理を実行します。

**主要な処理**:

1. 前提情報の収集（Backlog、ユニット定義、ドメイン設計、アーキテクチャ設計）
2. BDD受入基準をテストケースに変換（Given/When/Then → Arrange/Act/Assert）
3. アーキテクチャタイプの判定
   - Lambda / Event-Driven
   - REST API
   - Web Application
   - Batch / CLI
   - GraphQL API
   - Microservices
4. テストレベルの分類
   - Unit Test: 単一関数・メソッドの動作検証（モック使用）
   - Integration Test: コンポーネント間連携検証（LocalStack/Testcontainers）
   - E2E Test: エンドツーエンド検証（**デプロイ先環境に対して実行**）
5. テストピラミッドの構成（70% Unit, 20% Integration, 10% E2E）
6. TDDサイクルの計画（Red → Green → Refactor）
7. モック・フィクスチャの設計
8. テストカバレッジ目標（Line 80%, Branch 70%, Function 90%）
9. テスト実装の優先順位（High/Medium/Low）
10. E2EテストのCI/CD統合

**設計詳細**: `.claude/agents/test-designer/prompt.md` を参照

---

## TDD/BDD統合

### BDD受入基準（Given/When/Then）

```gherkin
Scenario: 注文の作成
  Given 顧客がログインしている
  And カートに商品が1つ以上ある
  When 注文を確定する
  Then 注文が作成される
  And 在庫が減少する
```

### テストケースへの変換

```typescript
it("Scenario: 注文の作成", async () => {
  // Given: 顧客がログインしている
  const customer = createCustomer({ id: "user123" });

  // When: 注文を確定する
  const order = await orderService.createOrder({ customerId: customer.id });

  // Then: 注文が作成される
  expect(order.id).toBeDefined();
});
```

### TDDサイクル（Red-Green-Refactor）

1. **Red**: 失敗するテストを書く
2. **Green**: 最小限の実装で通す
3. **Refactor**: コードを改善

---

## テストレベル

### Unit Test（単体テスト）

- **実行環境**: ローカル、CI/CD
- **対象環境**: なし（モック使用）
- **例**: 値オブジェクトのバリデーション

### Integration Test（統合テスト）

- **実行環境**: ローカル、CI/CD
- **対象環境**: LocalStack/Testcontainers
- **例**: Repository + LocalStack DynamoDB

### E2E Test（エンドツーエンドテスト）

- **実行環境**: **ローカル、CI/CD**（どこから実行するか）
- **対象環境**: **デプロイ先環境（dev/staging）**（何に対して実行するか）⚠️ 重要
- **例**: デプロイ済みAPIエンドポイントへのHTTPリクエスト

**重要な理解**:
- E2Eテストは**デプロイ先環境に対して実行**
- ❌ 誤解: E2E = CI/CDでしか実行しない
- ✅ 正解: E2E = デプロイ済み実環境に対して実行（**ローカルからでもOK**）

---

## テストピラミッド

**理想的な比率**:
```
    E2E (10%)
   ────────
  Integration (20%)
  ──────────────────
      Unit (70%)
──────────────────────────
```

**分類の指針**:
- BDD受入基準 → まずE2Eテストで実装（デプロイ後に実行）
- E2Eテストが遅い/コスト高 → Integrationテストに分解（ローカル実行）
- Integration テストが複雑 → Unit テストで補完（モック使用）

---

## アーキテクチャ別のテスト戦略

### Lambda / Event-Driven

**Integration Test**: LocalStack DynamoDB/SQS + Lambda関数をローカル実行

**E2E Test**: デプロイ済みのLambda関数を実AWS APIで呼び出し

### REST API

**Integration Test**: ローカルサーバー起動 + In-Memory DB

**E2E Test**: デプロイ済みのAPIエンドポイントにHTTPリクエスト

### Web Application

**Integration Test**: ローカルサーバー + Playwright（localhost）

**E2E Test**: デプロイ済みのサイトにPlaywrightでアクセス

### Batch / CLI

**Integration Test**: ローカル実行 + Testcontainers

**E2E Test**: デプロイ済みのバッチジョブを実S3/DBで実行

### GraphQL API

**Integration Test**: ローカルApolloサーバー + In-Memory DB

**E2E Test**: デプロイ済みのGraphQL Endpointにクエリ

### Microservices

**Integration Test**: ローカルQueue Emulator + 各サービス

**E2E Test**: デプロイ済みの全サービス + 実Message Queue

---

## 生成されるファイル

### テスト設計ドキュメント

- `docs/design-artifacts/tests/{Backlog番号}_{ユニット名}_test_design.md` - テスト設計

**ファイル名例**: `046_order-management_test_design.md`

### テスト実装の配置

```
packages/
├── {unit}/
│   ├── src/
│   └── tests/
│       ├── unit/           # Unit Test（ローカル実行）
│       └── integration/    # Integration Test（ローカル実行）
└── e2e/                    # E2E Test専用パッケージ
    ├── tests/
    │   ├── lambda.e2e.test.ts       # Lambda E2E
    │   ├── api.e2e.test.ts          # REST API E2E
    │   └── web.e2e.test.ts          # Web E2E
    └── .env.dev                      # デプロイ先環境の設定
```

---

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

---

## ベストプラクティス

### Given/When/Then コメント

```typescript
it("Scenario: 注文の作成", async () => {
  // Given: 顧客がログインしている
  const customer = createCustomer();

  // When: 注文を確定する
  const order = await orderService.createOrder({ customerId: customer.id });

  // Then: 注文が作成される
  expect(order.id).toBeDefined();
});
```

### Arrange-Act-Assert パターン

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

### テストケース名は具体的に

- ✅ "100階層のノードを5000トークン以内で取得できる"
- ❌ "test1", "動作確認"

---

## 実行後の次のステップ

```bash
# ボルト実行（TDDサイクルで実装）
/bolt unit1
```

`/bolt` 内で、このテスト設計を参照しながら Red → Green → Refactor サイクルを実行します。

---

## 注意事項

1. **テストファーストの原則**: 実装前にテストを書く
2. **Red → Green → Refactor**: TDDサイクルを守る
3. **E2E = デプロイ先環境に対するテスト**: ローカルからでもOK
4. **テストコードも保守対象**: 可読性、保守性を重視

---

## AI-DLC原則との対応

| 原則 | 実装方法 |
|-----|---------|
| **損失関数** | テストで早期に欠陥を検出（実装前に） |
| **段階的詳細化** | BDD受入基準 → テストケース → 実装 |
| **コンテキストメモリ** | テスト設計ドキュメントを永続化 |
| **トレーサビリティ** | 受入基準 → テストケース → 実装コードの紐づけ |
| **会話の逆転** | テストが仕様を定義（実装が後追い） |

---

**SubAgent Version**: 1.0.0
**SubAgent Location**: `.claude/agents/test-designer/`
**作成日**: 2025-11-19
**AI-DLC準拠**: コンストラクションフェーズ（テスト設計・TDD/BDD統合）
