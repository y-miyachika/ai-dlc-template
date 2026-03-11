---
name: api-generator
description: "REST API実装生成（Hono RPC）。既存のservices層からHTTP層（routes, schemas, types）とOpenAPI仕様を自動生成する。引数: ユニット名（例: unit1）"
---

# API Generator Skill

あなたはREST API設計とTypeScript実装の専門家です。既存のビジネスロジック（services層）をHTTPエンドポイントとして公開するAPI層を生成します。

## 目的

既存のビジネスロジックをラップする、型安全なREST API層（HTTP層）を自動生成します。

**技術スタック**: Hono + Zod + TypeScript

---

## 設定

出力パスは `.claude/skill-config.json` でカスタマイズ可能です。
詳細は [Skill共通設定ガイド](../config.md) を参照してください。

**デフォルト設定**:
```json
{
  "api-generator": {
    "outputDir": "packages/api",
    "docsDir": "docs/api",
    "routesDir": "src/routes",
    "schemasDir": "src/schemas",
    "servicesDir": "src/services"
  }
}
```

---

## 入力

### 必須
- **services層のパス**: 既存のサービスファイル（例: `{outputDir}/src/services/*.service.ts`）
- **ユニット名**: 対象のユニット（例: `unit1`, `001-unit1`）

### オプション
- **ドメイン設計**: `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/domain.md` 内のドメインモデル
- **出力先**: 設定ファイルまたはデフォルト値を使用

---

## 処理フロー

### ステップ1: 既存のservices層の読み込み

指定されたパスから既存のサービスを読み込み、以下を抽出：

- **サービス名**: 例: `userService`
- **公開メソッド**: 例: `findAll`, `findById`, `create`, `update`, `delete`
- **引数の型**: 例: `CreateUser`, `UpdateUser`
- **戻り値の型**: 例: `User`, `User[]`

**例**:
```typescript
// 既存のservices層
export const userService = {
  findAll: async (): Promise<User[]> => { ... },
  findById: async (id: string): Promise<User | null> => { ... },
  create: async (data: CreateUser): Promise<User> => { ... },
  update: async (id: string, data: UpdateUser): Promise<User | null> => { ... },
  delete: async (id: string): Promise<boolean> => { ... }
}
```

---

### ステップ2: REST APIエンドポイントの設計

既存のサービスメソッドから、HTTPエンドポイントをマッピング：

| サービスメソッド | HTTPエンドポイント | HTTPメソッド |
|---------------|------------------|------------|
| `findAll()` | `/api/{entity}` | GET |
| `findById(id)` | `/api/{entity}/:id` | GET |
| `create(data)` | `/api/{entity}` | POST |
| `update(id, data)` | `/api/{entity}/:id` | PUT |
| `delete(id)` | `/api/{entity}/:id` | DELETE |
| カスタムメソッド | `/api/{entity}/{action}` | POST |

#### 検証ルール
- サービスメソッドの引数型から、Zodスキーマを生成
- バリデーションエラーは400 Bad Requestで返す
- 存在しないリソースは404 Not Foundで返す

---

### ステップ3: ファイル生成

#### 生成するファイル

```
packages/api/
├── src/
│   ├── index.ts              # Honoアプリケーション
│   ├── routes/               # ← 生成
│   │   └── {entity}.ts       # エンティティごとのルート
│   ├── schemas/              # ← 生成
│   │   └── {entity}.ts       # Zodスキーマ
│   ├── services/             # ← 既存（読み込みのみ）
│   │   └── {entity}.service.ts
│   └── types/                # ← 生成
│       └── index.ts          # 型エクスポート
├── package.json              # ← 生成（存在しなければ）
└── tsconfig.json             # ← 生成（存在しなければ）
```

**重要**: `services/` は既存ファイルを読み込むだけで、新規生成しません。

---

#### テンプレート1: Zodスキーマ (`src/schemas/{entity}.ts`)

```typescript
import { z } from 'zod'

export const {Entity}Schema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  createdAt: z.date(),
  updatedAt: z.date(),
})

export const Create{Entity}Schema = {Entity}Schema.omit({
  id: true,
  createdAt: true,
  updatedAt: true,
})

export const Update{Entity}Schema = Create{Entity}Schema.partial()

export type {Entity} = z.infer<typeof {Entity}Schema>
export type Create{Entity} = z.infer<typeof Create{Entity}Schema>
export type Update{Entity} = z.infer<typeof Update{Entity}Schema>
```

---

#### テンプレート2: Honoルート (`src/routes/{entity}.ts`)

```typescript
import { Hono } from 'hono'
import { zValidator } from '@hono/zod-validator'
import { Create{Entity}Schema, Update{Entity}Schema } from '../schemas/{entity}'
import { {entity}Service } from '../services/{entity}.service'

const app = new Hono()

app.get('/', async (c) => {
  const items = await {entity}Service.findAll()
  return c.json(items)
})

app.get('/:id', async (c) => {
  const id = c.req.param('id')
  const item = await {entity}Service.findById(id)
  if (!item) return c.json({ error: 'Not found' }, 404)
  return c.json(item)
})

app.post('/', zValidator('json', Create{Entity}Schema), async (c) => {
  const data = c.req.valid('json')
  const item = await {entity}Service.create(data)
  return c.json(item, 201)
})

app.put('/:id', zValidator('json', Update{Entity}Schema), async (c) => {
  const id = c.req.param('id')
  const data = c.req.valid('json')
  const item = await {entity}Service.update(id, data)
  if (!item) return c.json({ error: 'Not found' }, 404)
  return c.json(item)
})

app.delete('/:id', async (c) => {
  const id = c.req.param('id')
  const deleted = await {entity}Service.delete(id)
  if (!deleted) return c.json({ error: 'Not found' }, 404)
  return c.json({ success: true })
})

export default app
```

---

#### テンプレート3: メインアプリケーション (`src/index.ts`)

```typescript
import { Hono } from 'hono'
import { logger } from 'hono/logger'
import { cors } from 'hono/cors'
import {entity}Routes from './routes/{entity}'

const app = new Hono()

app.use('*', logger())
app.use('*', cors())
app.route('/api/{entity}', {entity}Routes)
app.get('/health', (c) => c.json({ status: 'ok' }))

export default app
export type AppType = typeof app
```

---

#### テンプレート4: 型エクスポート (`src/types/index.ts`)

```typescript
export type { {Entity}, Create{Entity}, Update{Entity} } from '../schemas/{entity}'
```

---

#### テンプレート5: package.json

```json
{
  "name": "@{project}/api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest"
  },
  "dependencies": {
    "hono": "^4.0.0",
    "zod": "^3.22.0",
    "@hono/zod-validator": "^0.2.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0",
    "tsx": "^4.0.0",
    "vitest": "^1.0.0"
  }
}
```

---

### ステップ4: OpenAPI仕様生成

`docs/api/openapi.yaml` にOpenAPI 3.0仕様を生成。エンドポイント一覧、リクエスト/レスポンススキーマを含む。

---

### ステップ5: ドキュメント生成

`docs/api/{unit}_API設計.md` にエンドポイント一覧、使用例（Hono RPCクライアント）を生成。

---

## 出力

1. **HTTP層**: routes/, schemas/, index.ts, types/index.ts
2. **設定ファイル**（存在しなければ）: package.json, tsconfig.json
3. **ドキュメント**: openapi.yaml, {unit}_API設計.md

---

## エラーハンドリング

- services層が見つからない場合はエラー
- 不正なサービス定義がある場合は警告
- 既存のroutesファイルがある場合は確認してから上書き

---

## フロントエンド連携

```typescript
import { hc } from 'hono/client'
import type { AppType } from '@{project}/api'

const client = hc<AppType>('http://localhost:3000')
const res = await client.api.{entity}.$get()
const data = await res.json() // 型推論が効く
```
