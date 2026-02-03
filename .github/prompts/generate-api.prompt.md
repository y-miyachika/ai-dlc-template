---
name: generate-api
description: REST API実装生成（Hono RPC、OpenAPI）
agent: agent
argument-hint: ユニット名（例: unit1）
---

# REST API実装生成（Hono RPC）

あなたは**API Generator**として、REST API層を自動生成します。

## 前提条件

`/bolt` でビジネスロジック（services層）が実装済みであること。

## 技術スタック

Hono + Zod + TypeScript

## 実行手順

### 1. services層の読み込み

`packages/api/src/services/` 配下のサービスを分析：
- 公開メソッド
- 型情報
- 依存関係

### 2. HTTP層の生成

- `packages/api/src/routes/{entity}.ts` - Honoルート定義
- `packages/api/src/schemas/{entity}.ts` - Zodスキーマ
- `packages/api/src/index.ts` - Honoアプリケーション
- `packages/api/src/types/index.ts` - 型エクスポート

### 3. OpenAPI仕様の生成

`docs/api/openapi.yaml` を生成。

### 4. API設計ドキュメントの生成

`docs/api/{unit}_API設計.md` を生成。

## 生成例

```typescript
// routes/users.ts
import { Hono } from 'hono'
import { zValidator } from '@hono/zod-validator'
import { userSchema } from '../schemas/user'
import { userService } from '../services/user.service'

const app = new Hono()

app.get('/', async (c) => {
  const users = await userService.findAll()
  return c.json(users)
})

app.post('/', zValidator('json', userSchema), async (c) => {
  const data = c.req.valid('json')
  const user = await userService.create(data)
  return c.json(user, 201)
})

export default app
```

## フロントエンド連携

Hono RPCクライアントで型安全に利用：

```typescript
import { hc } from 'hono/client'
import type { AppType } from '@{project}/api'

const client = hc<AppType>('http://localhost:3000')
const res = await client.api.users.$get()
const data = await res.json() // 型推論が効く
```

## 次のステップ

```bash
pnpm install
cd packages/api && pnpm dev
```

## 参照

詳細な手順: `.claude/skills/api-generator/prompt.md`
