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

// エンティティスキーマ
export const {Entity}Schema = z.object({
  id: z.string().uuid(),
  // ドメインモデルのプロパティから生成
  name: z.string().min(1).max(100),
  email: z.string().email(),
  createdAt: z.date(),
  updatedAt: z.date(),
})

// 作成時のスキーマ（IDなし）
export const Create{Entity}Schema = {Entity}Schema.omit({
  id: true,
  createdAt: true,
  updatedAt: true,
})

// 更新時のスキーマ（部分的）
export const Update{Entity}Schema = Create{Entity}Schema.partial()

// 型エクスポート
export type {Entity} = z.infer<typeof {Entity}Schema>
export type Create{Entity} = z.infer<typeof Create{Entity}Schema>
export type Update{Entity} = z.infer<typeof Update{Entity}Schema>
```

---

#### テンプレート2: Honoルート (`src/routes/{entity}.ts`)

```typescript
import { Hono } from 'hono'
import { zValidator } from '@hono/zod-validator'
import { {Entity}Schema, Create{Entity}Schema, Update{Entity}Schema } from '../schemas/{entity}'
import { {entity}Service } from '../services/{entity}.service'

const app = new Hono()

// 一覧取得
app.get('/', async (c) => {
  const items = await {entity}Service.findAll()
  return c.json(items)
})

// 単一取得
app.get('/:id', async (c) => {
  const id = c.req.param('id')
  const item = await {entity}Service.findById(id)

  if (!item) {
    return c.json({ error: 'Not found' }, 404)
  }

  return c.json(item)
})

// 作成
app.post('/', zValidator('json', Create{Entity}Schema), async (c) => {
  const data = c.req.valid('json')
  const item = await {entity}Service.create(data)
  return c.json(item, 201)
})

// 更新
app.put('/:id', zValidator('json', Update{Entity}Schema), async (c) => {
  const id = c.req.param('id')
  const data = c.req.valid('json')
  const item = await {entity}Service.update(id, data)

  if (!item) {
    return c.json({ error: 'Not found' }, 404)
  }

  return c.json(item)
})

// 削除
app.delete('/:id', async (c) => {
  const id = c.req.param('id')
  const deleted = await {entity}Service.delete(id)

  if (!deleted) {
    return c.json({ error: 'Not found' }, 404)
  }

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

// ルートのインポート
import {entity}Routes from './routes/{entity}'

const app = new Hono()

// ミドルウェア
app.use('*', logger())
app.use('*', cors())

// ルート登録
app.route('/api/{entity}', {entity}Routes)

// ヘルスチェック
app.get('/health', (c) => c.json({ status: 'ok' }))

export default app
export type AppType = typeof app
```

---

#### テンプレート4: 型エクスポート (`src/types/index.ts`)

```typescript
// フロントエンドから参照可能な型をエクスポート
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

`docs/api/openapi.yaml` を生成：

```yaml
openapi: 3.0.0
info:
  title: {Unit Name} API
  version: 1.0.0
  description: Generated from domain model

servers:
  - url: http://localhost:3000
    description: Development server

paths:
  /api/{entity}:
    get:
      summary: Get all {entity} items
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/{Entity}'

    post:
      summary: Create {entity}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Create{Entity}'
      responses:
        '201':
          description: Created

  /api/{entity}/{id}:
    get:
      summary: Get {entity} by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Successful response
        '404':
          description: Not found

components:
  schemas:
    {Entity}:
      type: object
      properties:
        id:
          type: string
          format: uuid
        # ドメインモデルから生成
```

---

### ステップ5: ドキュメント生成

`docs/api/{unit}_API設計.md` を生成：

```markdown
# {Unit} - API設計

## 概要

{ユニットの説明}

## エンドポイント一覧

| エンドポイント | メソッド | 説明 | 認証 |
|--------------|---------|------|------|
| /api/{entity} | GET | 一覧取得 | 不要 |
| /api/{entity}/:id | GET | 単一取得 | 不要 |
| /api/{entity} | POST | 作成 | 必要 |
| /api/{entity}/:id | PUT | 更新 | 必要 |
| /api/{entity}/:id | DELETE | 削除 | 必要 |

## 使用例

### フロントエンドからの利用（Hono RPC）

\`\`\`typescript
import { hc } from 'hono/client'
import type { AppType } from '@{project}/api'

const client = hc<AppType>('http://localhost:3000')

// 型安全なAPIコール
const res = await client.api.{entity}.$get()
const data = await res.json() // 型推論が効く
\`\`\`
```

---

## 出力

### 生成されるファイル

1. **HTTP層**
   - `packages/api/src/routes/` - Honoルート定義
   - `packages/api/src/schemas/` - Zodスキーマ
   - `packages/api/src/index.ts` - Honoアプリケーション
   - `packages/api/src/types/index.ts` - 型エクスポート

2. **設定ファイル**（存在しなければ）
   - `packages/api/package.json`
   - `packages/api/tsconfig.json`

3. **ドキュメント**
   - `docs/api/openapi.yaml` - OpenAPI仕様
   - `docs/api/{unit}_API設計.md` - API設計ドキュメント

---

## エラーハンドリング

- services層が見つからない場合はエラー
- 不正なサービス定義がある場合は警告
- 既存のroutesファイルがある場合は確認してから上書き

---

## フロントエンド連携

Hono RPCクライアントを使用することで、型安全なAPI呼び出しが可能：

```typescript
// apps/web/src/lib/api.ts
import { hc } from 'hono/client'
import type { AppType } from '@{project}/api'

const client = hc<AppType>('http://localhost:3000')

// 型推論が効く
const res = await client.api.{entity}.$get()
const data = await res.json()
```

---

**Skill Version**: 1.0.0
**Last Updated**: 2025-11-19
