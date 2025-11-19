# テスト設計（AI-DLC準拠・TDD/BDD統合）

引数として受け取ったユニット名をもとに、TDD（Test-Driven Development）とBDD（Behavior-Driven Development）を統合したテスト設計を行います。

## 入力内容
{{ARGS}}

---

## テスト設計とは

AI-DLCにおける**品質保証フェーズ**です：
- 受入基準（BDD）をテストケースに変換
- テストファーストで実装を駆動（TDD）
- 自動化可能なテストスイートの設計
- テストカバレッジ目標の設定

**目的**: 実装前にテストを設計し、品質を組み込む

---

## 動作フロー

### ステップ1: 前提情報の収集

以下のドキュメントを読み込む：

1. **インテント/Backlog**
   - `docs/intents/[番号]_*.md` または `docs/backlog/[番号]_*.md`
   - 受入基準（Given/When/Then形式）を確認

2. **ユニット分解**
   - `docs/units/[番号]_units.md`
   - 対象ユニットの責務、依存関係を確認

3. **ドメイン設計**
   - `docs/design-artifacts/domain/[番号]_[ユニット名]_domain.md`
   - ドメインモデル（エンティティ、集約等）を確認

4. **アーキテクチャ設計**
   - `docs/design-artifacts/architecture/[番号]_[ユニット名]_architecture.md`
   - コンポーネント構成、データフローを確認

---

### ステップ2: BDD形式の受入基準をテストケースに変換

**受入基準の例**:
```gherkin
Scenario:  APIレスポンスのトークン削減
  Given 100階層の
  When 
  Then トークン使用量が5000以内である
  And レイアウト情報の精度が従来版と同等である
```

**変換後のテストケース構造**:
```markdown
### テストケース1: トークン削減の検証

**前提条件（Setup）**:
- 100階層の
- フィクスチャファイル: `fixtures/100-layer-node.json`

**実行（Exercise）**:
- ` ツールを実行
- パラメータ: `{ fileKey: "test", nodeIds: "root" }`

**検証（Verify）**:
- アサーション1: レスポンストークン数が5000以内
- アサーション2: レイアウト情報の各プロパティが期待値と一致
- アサーション3: エラーが発生しない

**後処理（Teardown）**:
- テストデータのクリーンアップ
```

---

### ステップ3: テストレベルの分類

各テストケースを以下のレベルに分類：

| レベル | 目的 | スコープ | 実行する場所 | 対象環境 | 実行タイミング |
|--------|------|---------|------------|---------|--------------|
| **Unit Test** | 単一関数・メソッドの動作検証 | 1関数/クラス | ローカル、CI/CD | なし（モック） | コミット時、PR作成時、いつでも |
| **Integration Test** | コンポーネント間の連携検証 | 複数コンポーネント | ローカル、CI/CD | LocalStack/Testcontainers | Phase完了時、コミット前、PR作成時 |
| **E2E Test** | エンドツーエンドのユーザーシナリオ検証 | システム全体 | **ローカル、CI/CD** | **デプロイ先環境（dev/staging）** ⚠️ 重要 | デプロイ完了後 |

**重要な理解**:
- E2Eテストは**どこで実行するか**ではなく、**何に対して実行するか**が重要
- ❌ 誤解: E2E = CI/CDでしか実行しない
- ✅ 正解: E2E = デプロイ済み実環境に対して実行（**ローカルからでもCI/CDからでもOK**）

**例**:
```bash
# 開発者のローカルマシンから、dev環境に対してE2E実行
pnpm test:e2e:dev  # ← ローカルで実行、対象はhttps://dev-api.example.com

# CI/CDから、dev環境に対してE2E実行（自動）
# GitHub Actionsが同じテストを自動実行
```

**LocalStackとの違い**:
- LocalStack: ローカルで**モック環境**を起動 → Integration Test
- E2E: ローカルから**実環境**にアクセス → E2E Test

---

### ステップ3.5: アーキテクチャタイプの判定（🆕）

ドメイン設計とアーキテクチャ設計から、システムのアーキテクチャタイプを判定：

**検出可能なアーキテクチャ**:
- [ ] **Lambda / Event-Driven**: Lambda関数、SQS、EventBridge等
- [ ] **REST API**: Hono、Express、FastAPI等
- [ ] **Web Application**: Next.js、React、Vue等（SSR/SPA）
- [ ] **Batch / CLI**: バッチ処理、CLIツール
- [ ] **GraphQL API**: Apollo Server、Hasura等
- [ ] **Microservices**: 複数サービス間の連携

**アーキテクチャ別のE2E定義**:

#### Lambda / Event-Driven
- **Integration Test**: LocalStack DynamoDB/SQS + Lambda関数をローカル実行
- **E2E Test**: デプロイ済みのLambda関数を実AWS APIで呼び出し
  ```typescript
  // E2E: デプロイ先のLambda呼び出し
  await lambdaClient.invoke({
    FunctionName: 'my-app-dev-orchestrator' // デプロイ済み
  })
  ```

#### REST API
- **Integration Test**: ローカルサーバー起動 + In-Memory DB
- **E2E Test**: デプロイ済みのAPIエンドポイントにHTTPリクエスト
  ```typescript
  // E2E: デプロイ先のAPI呼び出し
  await fetch('https://dev-api.example.com/users')
  ```

#### Web Application
- **Integration Test**: ローカルサーバー + Playwright（localhost）
- **E2E Test**: デプロイ済みのサイトにPlaywrightでアクセス
  ```typescript
  // E2E: デプロイ先のサイトにアクセス
  await page.goto('https://dev.example.com')
  ```

#### Batch / CLI
- **Integration Test**: ローカル実行 + Testcontainers
- **E2E Test**: デプロイ済みのバッチジョブを実S3/DBで実行
  ```typescript
  // E2E: デプロイ先のバッチジョブトリガー
  await triggerBatchJob({ inputKey: 'test.csv' })
  ```

#### GraphQL API
- **Integration Test**: ローカルApolloサーバー + In-Memory DB
- **E2E Test**: デプロイ済みのGraphQL Endpointにクエリ
  ```typescript
  // E2E: デプロイ先のGraphQLエンドポイント
  const client = new ApolloClient({
    uri: 'https://dev.example.com/graphql'
  })
  ```

#### Microservices
- **Integration Test**: ローカルQueue Emulator + 各サービス
- **E2E Test**: デプロイ済みの全サービス + 実Message Queue
  ```typescript
  // E2E: デプロイ先の分散トランザクション
  await orderService.createOrder({ /* ... */ })
  await waitForEvent('PAYMENT_COMPLETED')
  ```

---

**分類の指針**:
- BDD受入基準 → まずE2Eテストで実装（デプロイ後に実行）
- E2Eテストが遅い/コスト高 → Integrationテストに分解（ローカル実行）
- Integration テストが複雑 → Unit テストで補完（モック使用）

---

### ステップ4: テストピラミッドの構成

**理想的な比率**:
```
    E2E (10%)
   ────────
  Integration (20%)
  ──────────────────
      Unit (70%)
──────────────────────────
```

**このユニットの目標**:
| レベル | 目標ケース数 | 理由 |
|--------|------------|------|
| Unit | 10件 | 主要関数のカバレッジ70%以上 |
| Integration | 3件 | コンポーネント連携の主要パス |
| E2E | 1件 | クリティカルな受入基準のみ |

---

### ステップ5: TDDサイクルの計画

各テストケースについて、以下のサイクルを計画：

```markdown
### テストケース1: トークン削減の検証（TDDサイクル）

**Red（失敗するテストを書く）**:
- テストファイル: `src/tools/
- テストコード:
  ```typescript
  describe(" () => {
    it("100階層のノードを5000トークン以内で取得できる", async () => {
      // Arrange
      const fixture = loadFixture("100-layer-node.json");

      // Act
      const result = await  fileKey: "test", nodeIds: "root" });

      // Assert
      expect(result.tokenCount).toBeLessThanOrEqual(5000);
    });
  });
  ```

**Green（最小限の実装で通す）**:
- 実装ファイル: `src/tools/
- 実装内容: トークン削減ロジックの最小実装

**Refactor（リファクタリング）**:
- コードの整理、パフォーマンス改善
- テストが通ることを確認しながら改善
```

---

### ステップ6: モック・フィクスチャの設計

**外部依存の特定**:
-  REST API
- ファイルシステム
- 環境変数

**モック戦略**:
| 依存 | モック方法 | 理由 |
|-----|----------|------|
|  REST API | `vi.mock()` でモック | 実APIは遅い、レート制限あり |
| ファイルシステム | テスト用ディレクトリ | 実ファイル操作が必要 |
| 環境変数 | `vi.stubEnv()` | 環境に依存しないテスト |

**フィクスチャファイル**:
```
tests/
├── fixtures/
│   ├── 100-layer-node.json       # 100階層の
│   ├── simple-node.json          # シンプルなノード
│   └── error-response.json       # エラーレスポンス
└── helpers/
    └── loadFixture.ts            # フィクスチャ読み込みヘルパー
```

---

### ステップ7: テストカバレッジ目標の設定

**目標カバレッジ**:
| 指標 | 目標値 | 理由 |
|-----|--------|------|
| **Line Coverage** | 80%以上 | 主要ロジックをカバー |
| **Branch Coverage** | 70%以上 | 条件分岐を網羅 |
| **Function Coverage** | 90%以上 | すべての関数をテスト |

**未カバーを許容する箇所**:
- ロギング処理
- エラーハンドリングの一部（実現困難なケース）
- デバッグ用コード

---

### ステップ8: テスト実装の優先順位

**優先度の判定基準**:
1. **High**: クリティカルな受入基準、リスクが高い機能
2. **Medium**: 重要だが回避策がある機能
3. **Low**: 補助的な機能、UI調整等

**実装順序**:
```markdown
### Phase 1: クリティカルパス（受入基準直結）
- [ ] テストケース1: トークン削減の検証（High）
- [ ] テストケース2: レイアウト精度の検証（High）

### Phase 2: エッジケース
- [ ] テストケース3: エラーハンドリング（Medium）
- [ ] テストケース4: 空ノードの処理（Medium）

### Phase 3: 補助機能
- [ ] テストケース5: ロギング（Low）
- [ ] テストケース6: キャッシュ機能（Low）
```

---

## 出力先

### テスト設計ドキュメント
`docs/design-artifacts/tests/[番号]_[ユニット名]_test_design.md`

**内容**:
- BDD受入基準のテストケース変換
- アーキテクチャタイプの判定（🆕）
- テストレベルの分類（実行環境明記）
- テストピラミッド構成
- TDDサイクルの計画
- モック・フィクスチャ設計
- カバレッジ目標
- 実装優先順位
- E2Eテストの実行環境とCI/CD統合方法（🆕）

### テスト実装の配置

```
packages/
├── {unit}/
│   ├── src/
│   └── tests/
│       ├── unit/           # Unit Test（ローカル実行）
│       │   └── *.test.ts
│       └── integration/    # Integration Test（ローカル実行、LocalStack）
│           └── *.test.ts
└── e2e/                    # 🆕 E2E Test専用パッケージ
    ├── package.json
    ├── tests/
    │   ├── lambda.e2e.test.ts       # Lambda E2E（デプロイ先実行）
    │   ├── api.e2e.test.ts          # REST API E2E（デプロイ先実行）
    │   └── web.e2e.test.ts          # Web E2E（デプロイ先実行）
    ├── playwright.config.ts          # Web E2Eの場合
    └── .env.dev                      # デプロイ先環境の設定
```

**重要**: E2Eテストは専用パッケージに分離し、デプロイ先の環境変数を使用します。

### テスト実行コマンド

```json
// package.json (各unitまたはルート)
{
  "scripts": {
    "test:unit": "vitest run tests/unit",
    "test:integration": "vitest run tests/integration",
    "test:e2e:dev": "cd ../e2e && TEST_ENV=dev vitest run",
    "test:e2e:staging": "cd ../e2e && TEST_ENV=staging vitest run"
  }
}
```

### CI/CDパイプライン統合（🆕）

E2Eテストはデプロイ後に実行する必要があるため、以下の順序を推奨：

```yaml
# 例: GitHub Actions
jobs:
  # 1. ローカルで実行可能なテスト
  unit-and-integration:
    runs-on: ubuntu-latest
    steps:
      - run: pnpm test:unit
      - run: docker compose up -d localstack
      - run: pnpm test:integration

  # 2. dev環境にデプロイ
  deploy-dev:
    needs: unit-and-integration
    runs-on: ubuntu-latest
    steps:
      - run: terraform apply -auto-approve

  # 3. デプロイ後にE2Eテスト実行 ← 重要
  e2e-dev:
    needs: deploy-dev
    runs-on: ubuntu-latest
    env:
      TEST_ENV: dev
      API_URL: https://dev-api.example.com
    steps:
      - name: Wait for deployment
        run: until curl -f $API_URL/health; do sleep 5; done
      - run: pnpm test:e2e:dev

  # 4. staging環境へのデプロイ（mainブランチのみ）
  deploy-staging:
    needs: e2e-dev
    if: github.ref == 'refs/heads/main'
    steps:
      - run: terraform apply -auto-approve

  # 5. staging環境でもE2E実行
  e2e-staging:
    needs: deploy-staging
    env:
      TEST_ENV: staging
    steps:
      - run: pnpm test:e2e:staging
```

### E2Eテストで検証すべき項目（アーキテクチャ別）

#### Lambda / Event-Driven
- [ ] 実Lambda関数が起動するか
- [ ] 実DynamoDB/RDSに書き込めるか
- [ ] IAMロールの権限が正しいか
- [ ] 環境変数が正しく設定されているか
- [ ] タイムアウト設定が適切か

#### REST API
- [ ] 実APIエンドポイントにアクセスできるか
- [ ] CORS設定が正しいか
- [ ] 認証・認可が動作するか
- [ ] レート制限が動作するか
- [ ] エラーレスポンスが適切か

#### Web Application
- [ ] 実サイトが表示されるか
- [ ] 静的アセット（CSS/JS/画像）が配信されるか
- [ ] ログイン/ログアウトフローが動作するか
- [ ] パフォーマンス（LCP、FID等）が許容範囲か
- [ ] SEO（OGP、meta tags）が正しいか

#### Batch / CLI
- [ ] 実S3からデータを読み込めるか
- [ ] 実DBに結果を保存できるか
- [ ] バッチジョブが正常に完了するか
- [ ] エラー時のリトライ処理が動作するか

#### GraphQL API
- [ ] 実GraphQL Endpointにアクセスできるか
- [ ] Mutation/Queryが動作するか
- [ ] Subscriptionが動作するか（該当する場合）
- [ ] 認証・認可が動作するか

#### Microservices
- [ ] サービス間通信が動作するか
- [ ] イベント駆動フローが動作するか
- [ ] 最終的な整合性が保たれるか
- [ ] サービス障害時のフォールバック処理が動作するか

---

## 次のステップ

テスト設計完了後、以下に進む：

```bash
# ボルト実行（TDDサイクルで実装）
/bolt [ユニット名]
```

`/bolt` 内で、このテスト設計を参照しながら Red → Green → Refactor サイクルを実行する。

---

## ベストプラクティス

### 1. テストケース名の命名規則

**良い例**:
```typescript
describe(" () => {
  describe("トークン削減機能", () => {
    it("100階層のノードを5000トークン以内で取得できる", async () => {
      // ...
    });

    it("レイアウト情報の精度が従来版と同等である", async () => {
      // ...
    });
  });

  describe("エラーハンドリング", () => {
    it("ネットワークエラー時に適切なエラーメッセージを返す", async () => {
      // ...
    });
  });
});
```

**悪い例**:
```typescript
it("test1", () => { /* ... */ });
it("動作確認", () => { /* ... */ });
```

---

### 2. Arrange-Act-Assert パターン

**推奨構造**:
```typescript
it("100階層のノードを5000トークン以内で取得できる", async () => {
  // Arrange（Given）: テストデータの準備
  const fixture = loadFixture("100-layer-node.json");
  const mock = vi.fn().mockResolvedValue(fixture);

  // Act（When）: テスト対象の実行
  const result = await 
    fileKey: "test",
    nodeIds: "root"
  });

  // Assert（Then）: 期待値の検証
  expect(result.tokenCount).toBeLessThanOrEqual(5000);
  expect(result.layout).toEqual(expect.objectContaining({
    width: expect.any(Number),
    height: expect.any(Number)
  }));
});
```

---

### 3. Given/When/Then コメントの活用

```typescript
it("Scenario: トークン削減の検証", async () => {
  // Given: 100階層の
  const fixture = loadFixture("100-layer-node.json");

  // When: 
  const result = await 
    fileKey: "test",
    nodeIds: "root"
  });

  // Then: トークン使用量が5000以内である
  expect(result.tokenCount).toBeLessThanOrEqual(5000);

  // And: レイアウト情報の精度が従来版と同等である
  expect(result.layout).toMatchSnapshot();
});
```

---

### 4. モックの使い分け

| 状況 | 推奨方法 | 理由 |
|-----|---------|------|
| HTTP API呼び出し | `vi.mock("axios")` | 実APIは遅い、不安定 |
| ファイル読み込み | 実ファイル（フィクスチャ） | ファイル操作の検証が必要 |
| 時刻依存 | `vi.setSystemTime()` | 決定的なテストのため |
| 乱数 | `vi.spyOn(Math, "random")` | 再現可能なテストのため |

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

## トラブルシューティング

### 受入基準が曖昧な場合

**対処**:
1. `/intent` または `/backlog` を確認
2. Given/When/Then形式で書かれているか確認
3. 曖昧な場合は、ユーザーに明確化を依頼

**例**:
```
❌ 曖昧: "うまく動く"
✅ 明確: "100階層のノードを5000トークン以内で取得できる"
```

---

### テストケースが多すぎる場合

**対処**:
1. テストピラミッドの比率を確認（70% Unit, 20% Integration, 10% E2E）
2. 重複するテストケースを統合
3. 優先度Low のテストケースを延期

---

### モックが複雑になりすぎる場合

**対処**:
1. Integration テストに切り替え（実DB/API使用）
2. テスト用のFactoryパターンを導入
3. フィクスチャファイルを活用

---

## テスト実装テンプレート（Vitest）

```typescript
import { describe, expect, it, vi, beforeEach, afterEach } from "vitest";
import { loadFixture } from "../helpers/loadFixture";

describe("[ユニット名]", () => {
  describe("[機能名]", () => {
    beforeEach(() => {
      // テスト前の準備
      vi.clearAllMocks();
    });

    afterEach(() => {
      // テスト後のクリーンアップ
    });

    it("Scenario: [シナリオ名]", async () => {
      // Given: [前提条件]
      const fixture = loadFixture("[ファイル名].json");

      // When: [実行]
      const result = await [関数名]([引数]);

      // Then: [検証]
      expect(result).toEqual([期待値]);

      // And: [追加検証]
      expect(result).toMatchSnapshot();
    });

    it("エラーケース: [エラーシナリオ]", async () => {
      // Given: [エラーを引き起こす条件]
      const invalidInput = { /* ... */ };

      // When/Then: [エラーが発生することを検証]
      await expect([関数名](invalidInput)).rejects.toThrowError("[期待されるエラーメッセージ]");
    });
  });
});
```

---

**作成日**: 2025-11-13
**AI-DLC準拠**: コンストラクションフェーズ（テスト設計・TDD/BDD統合）
