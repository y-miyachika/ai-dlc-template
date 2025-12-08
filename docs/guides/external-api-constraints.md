# 外部API制約対応ガイド

このガイドでは、外部APIに依存するシステムを設計・実装する際の制約管理と対応方法を説明します。

## 外部API制約とは

外部API（サードパーティAPI、SaaS、マイクロサービス等）を利用する際に考慮すべき制約：

| 制約カテゴリ | 例 | 影響 |
|-------------|---|------|
| レート制限 | 100リクエスト/分 | スロットリング、429エラー |
| クォータ | 10,000リクエスト/月 | コスト増、サービス停止 |
| ペイロードサイズ | 最大10MB | 分割送信が必要 |
| タイムアウト | 30秒 | 非同期処理が必要 |
| 認証方式 | OAuth2.0、APIキー | トークン管理 |
| SLA | 99.9%可用性 | フォールバック設計 |

---

## 設計フェーズでの対応

### /design-architecture での記録

`/design-architecture` 実行時に、以下を設計ドキュメントに記録してください：

```markdown
## 外部API制約

### API: Stripe Payment API

| 制約 | 値 | 対応策 |
|-----|-----|-------|
| レート制限 | 100 req/sec | クライアントサイドでレート制限実装 |
| タイムアウト | 30秒 | 非同期処理 + Webhook |
| ペイロード | 64KB | ページネーション |
| SLA | 99.95% | フォールバック決済手段 |

### API: SendGrid Email API

| 制約 | 値 | 対応策 |
|-----|-----|-------|
| レート制限 | 1000 msg/sec | キュー処理 |
| 月間クォータ | 40,000通 | 使用量監視アラート |
| 添付ファイル | 30MB | S3プリサインドURL |
```

---

## 制約カテゴリ別の対応パターン

### 1. レート制限対応

#### パターン1: クライアントサイドレート制限

```typescript
// utils/rateLimiter.ts
type RateLimiterConfig = {
  maxRequests: number
  windowMs: number
}

export const createRateLimiter = (config: RateLimiterConfig) => {
  const queue: Array<() => Promise<void>> = []
  let requestCount = 0
  let windowStart = Date.now()

  const processQueue = async () => {
    const now = Date.now()
    if (now - windowStart >= config.windowMs) {
      windowStart = now
      requestCount = 0
    }

    while (queue.length > 0 && requestCount < config.maxRequests) {
      const request = queue.shift()
      if (request) {
        requestCount++
        await request()
      }
    }
  }

  return {
    enqueue: <T>(fn: () => Promise<T>): Promise<T> => {
      return new Promise((resolve, reject) => {
        queue.push(async () => {
          try {
            resolve(await fn())
          } catch (e) {
            reject(e)
          }
        })
        processQueue()
      })
    }
  }
}

// 使用例
const stripeRateLimiter = createRateLimiter({
  maxRequests: 100,
  windowMs: 1000, // 1秒
})

const result = await stripeRateLimiter.enqueue(() =>
  stripe.customers.create({ email: 'user@example.com' })
)
```

#### パターン2: 指数バックオフリトライ

```typescript
// utils/retry.ts
type RetryConfig = {
  maxRetries: number
  baseDelayMs: number
  maxDelayMs: number
}

export const withRetry = async <T>(
  fn: () => Promise<T>,
  config: RetryConfig
): Promise<T> => {
  let lastError: Error | undefined

  for (let attempt = 0; attempt <= config.maxRetries; attempt++) {
    try {
      return await fn()
    } catch (error) {
      lastError = error as Error

      // 429エラー（レート制限）の場合のみリトライ
      if ((error as any).status !== 429) {
        throw error
      }

      if (attempt < config.maxRetries) {
        const delay = Math.min(
          config.baseDelayMs * Math.pow(2, attempt),
          config.maxDelayMs
        )
        await new Promise(resolve => setTimeout(resolve, delay))
      }
    }
  }

  throw lastError
}

// 使用例
const result = await withRetry(
  () => externalApi.call(),
  { maxRetries: 3, baseDelayMs: 1000, maxDelayMs: 30000 }
)
```

### 2. クォータ管理

#### 使用量監視

```typescript
// services/quotaMonitor.ts
type QuotaConfig = {
  monthlyLimit: number
  warningThreshold: number // 0.8 = 80%
  criticalThreshold: number // 0.95 = 95%
}

export const createQuotaMonitor = (config: QuotaConfig) => {
  let currentUsage = 0

  return {
    increment: (count = 1) => {
      currentUsage += count
      const usage = currentUsage / config.monthlyLimit

      if (usage >= config.criticalThreshold) {
        console.error(`[CRITICAL] API quota at ${(usage * 100).toFixed(1)}%`)
        // アラート送信
      } else if (usage >= config.warningThreshold) {
        console.warn(`[WARNING] API quota at ${(usage * 100).toFixed(1)}%`)
      }

      return { currentUsage, remaining: config.monthlyLimit - currentUsage }
    },
    getUsage: () => ({
      current: currentUsage,
      limit: config.monthlyLimit,
      percentage: (currentUsage / config.monthlyLimit) * 100,
    }),
  }
}
```

### 3. タイムアウト対応

#### 非同期処理 + Webhook

```typescript
// services/asyncProcessor.ts
type AsyncJob = {
  id: string
  status: 'pending' | 'processing' | 'completed' | 'failed'
  result?: unknown
  error?: string
}

// 長時間処理はジョブとしてキューに投入
export const submitAsyncJob = async (payload: unknown): Promise<string> => {
  const jobId = generateUUID()

  await jobQueue.push({
    id: jobId,
    payload,
    status: 'pending',
  })

  return jobId
}

// Webhookで結果を受信
export const handleWebhook = async (jobId: string, result: unknown) => {
  await jobRepository.update(jobId, {
    status: 'completed',
    result,
  })

  // クライアントへ通知（WebSocket、Push等）
  await notifyClient(jobId, result)
}
```

### 4. SLA対応（フォールバック）

#### サーキットブレーカーパターン

```typescript
// utils/circuitBreaker.ts
type CircuitBreakerConfig = {
  failureThreshold: number // 失敗回数の閾値
  resetTimeoutMs: number // 回復までの待機時間
}

type CircuitState = 'closed' | 'open' | 'half-open'

export const createCircuitBreaker = <T>(
  primaryFn: () => Promise<T>,
  fallbackFn: () => Promise<T>,
  config: CircuitBreakerConfig
) => {
  let state: CircuitState = 'closed'
  let failureCount = 0
  let lastFailureTime = 0

  return async (): Promise<T> => {
    // 回復待機中
    if (state === 'open') {
      if (Date.now() - lastFailureTime >= config.resetTimeoutMs) {
        state = 'half-open'
      } else {
        console.log('[CircuitBreaker] Open - using fallback')
        return fallbackFn()
      }
    }

    try {
      const result = await primaryFn()
      // 成功したらリセット
      state = 'closed'
      failureCount = 0
      return result
    } catch (error) {
      failureCount++
      lastFailureTime = Date.now()

      if (failureCount >= config.failureThreshold) {
        state = 'open'
        console.error('[CircuitBreaker] Opened due to failures')
      }

      console.log('[CircuitBreaker] Failure - using fallback')
      return fallbackFn()
    }
  }
}

// 使用例
const getPaymentStatus = createCircuitBreaker(
  () => stripeApi.getStatus(paymentId), // Primary
  () => localCache.getStatus(paymentId), // Fallback
  { failureThreshold: 5, resetTimeoutMs: 30000 }
)
```

---

## テストでの対応

### /design-test での考慮

外部API依存のテスト戦略：

| テストレベル | 外部API | 対応 |
|------------|--------|------|
| Unit | モック | `vi.mock()` でモック |
| Integration | スタブ/エミュレータ | WireMock、LocalStack |
| E2E | 実API（Sandbox） | テスト用アカウント |

### モックの例

```typescript
// tests/mocks/stripeApi.ts
export const mockStripeApi = {
  customers: {
    create: vi.fn().mockResolvedValue({
      id: 'cus_test123',
      email: 'test@example.com',
    }),
  },
  paymentIntents: {
    create: vi.fn().mockResolvedValue({
      id: 'pi_test123',
      status: 'requires_payment_method',
    }),
  },
}

// レート制限エラーのシミュレーション
export const mockStripeRateLimitError = () => {
  mockStripeApi.customers.create.mockRejectedValueOnce({
    status: 429,
    message: 'Rate limit exceeded',
  })
}
```

### 制約違反テスト

```typescript
describe('外部API制約', () => {
  describe('レート制限', () => {
    it('429エラー時にリトライする', async () => {
      mockStripeRateLimitError()

      const result = await withRetry(
        () => stripeService.createCustomer({ email: 'test@example.com' }),
        { maxRetries: 3, baseDelayMs: 100, maxDelayMs: 1000 }
      )

      expect(result.id).toBe('cus_test123')
      expect(mockStripeApi.customers.create).toHaveBeenCalledTimes(2)
    })
  })

  describe('サーキットブレーカー', () => {
    it('連続失敗後にフォールバックを使用する', async () => {
      // 5回失敗させる
      for (let i = 0; i < 5; i++) {
        mockStripeApi.customers.create.mockRejectedValueOnce(new Error('API Error'))
      }

      const getCustomer = createCircuitBreaker(
        () => stripeService.getCustomer('cus_123'),
        () => Promise.resolve({ id: 'cus_123', source: 'cache' }),
        { failureThreshold: 5, resetTimeoutMs: 30000 }
      )

      // 6回目はフォールバックが使われる
      const result = await getCustomer()
      expect(result.source).toBe('cache')
    })
  })
})
```

---

## 実装でのチェックリスト

### Unit実装時（/bolt）

- [ ] レート制限が必要なAPIを特定
- [ ] リトライロジックを実装
- [ ] タイムアウト設定を確認
- [ ] エラーハンドリングを実装

### Integration時

- [ ] サーキットブレーカーを設定
- [ ] フォールバック処理を実装
- [ ] 使用量監視を設定

### デプロイ前

- [ ] 本番用APIキーを設定
- [ ] クォータアラートを設定
- [ ] SLA監視を設定

---

## よくある外部APIの制約一覧

### 決済系

| API | レート制限 | 特記事項 |
|-----|-----------|---------|
| Stripe | 100 req/sec | テストモードは別制限 |
| PayPal | 50 req/sec | Sandbox無制限 |
| Square | 300 req/min | |

### メール系

| API | レート制限 | 月間クォータ |
|-----|-----------|------------|
| SendGrid | 1000 msg/sec | プランによる |
| Mailgun | 300 msg/min | 10,000通/月（無料） |
| Amazon SES | 200 msg/sec | プランによる |

### 認証系

| API | レート制限 | 特記事項 |
|-----|-----------|---------|
| Auth0 | 10 req/sec（Management） | |
| Firebase Auth | 100 req/min | |
| Cognito | 10 TPS | リージョンによる |

---

## 参考資料

- [ワークフローDAG](./workflow-dag.md) - コマンド間の依存関係
- [実装スコープ](./implementation-scope.md) - 完了条件チェックリスト
- [E2Eテストフロー](./e2e-testing-flow.md) - テスト戦略
