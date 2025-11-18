# REST API実装生成（Hono RPC）

あなたはREST API設計とTypeScript実装の専門家です。`/bolt`で実装されたビジネスロジック（services層）をHTTPエンドポイントとして公開するAPI層を生成します。

## 前提条件

**このコマンドは `/bolt` の後に実行してください**

- `/bolt unit1` でビジネスロジック（services層）が実装済み
- `packages/api/src/services/*.service.ts` が存在する

## 目的

既存のビジネスロジックをラップする、型安全なREST API層（HTTP層）を自動生成します。

## 入力

1. **ユニット名**: 対象のユニット（例: `unit1`, `001-unit1`）
2. **既存のservices層**: `packages/api/src/services/` 内のサービスクラス（`/bolt`で生成済み）
3. **ドメイン設計**: `docs/design-artifacts/domain/` 内のドメインモデル（オプション）

## 処理フロー

### ステップ1: 既存のservices層の読み込み

`packages/api/src/services/` 配下のサービスを読み込みます：

```typescript
// 例: packages/api/src/services/user.service.ts
export const userService = {
  findAll: async (): Promise<User[]> => { ... },
  findById: async (id: string): Promise<User | null> => { ... },
  create: async (data: CreateUser): Promise<User> => { ... },
  update: async (id: string, data: UpdateUser): Promise<User | null> => { ... },
  delete: async (id: string): Promise<boolean> => { ... }
}
```

以下を抽出：
- **サービス名**: 例: `userService`
- **公開メソッド**: 例: `findAll`, `findById`, `create`, `update`, `delete`
- **引数の型**: 例: `CreateUser`, `UpdateUser`
- **戻り値の型**: 例: `User`, `User[]`

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

### ステップ3: Hono RPCアプリケーション生成

#### ディレクトリ構造

```
packages/api/
├── src/
│   ├── index.ts              # Honoアプリケーションのエントリーポイント
│   ├── routes/               # ← /generate-api が生成
│   │   └── {entity}.ts       # エンティティごとのルート定義
│   ├── schemas/              # ← /generate-api が生成
│   │   └── {entity}.ts       # Zodスキーマ定義
│   ├── services/             # ← /bolt で生成済み（既存）
│   │   └── {entity}.service.ts # ビジネスロジック
│   └── types/                # ← /generate-api が生成
│       └── index.ts          # 共有型定義
├── package.json              # ← /generate-api が生成（存在しなければ）
└── tsconfig.json             # ← /generate-api が生成（存在しなければ）
```

**重要**: `/generate-api` は以下のみ生成します：
- `routes/` - HTTP層
- `schemas/` - Zodスキーマ
- `index.ts` - Honoアプリケーション
- `types/` - 型エクスポート

`services/` は**既存のファイルを読み込むだけ**で、新規生成しません。

#### コード生成テンプレート

**1. Zodスキーマ (`src/schemas/{entity}.ts`)**

```typescript
import { z } from 'zod'

// エンティティスキーマ
export const {Entity}Schema = z.object({
  id: z.string().uuid(),
  // ドメインモデルのプロパティ
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

**2. Honoルート (`src/routes/{entity}.ts`)** ← `/generate-api` が生成

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

**3. サービス層 (`src/services/{entity}.service.ts`)** ← `/bolt` で生成済み

```typescript
// /bolt unit1 で既に実装されている想定
import { {Entity}, Create{Entity}, Update{Entity} } from '../schemas/{entity}'

export const {entity}Service = {
  findAll: async (): Promise<{Entity}[]> => {
    // /bolt で実装済み
    return await db.{entity}.findMany()
  },

  findById: async (id: string): Promise<{Entity} | null> => {
    return await db.{entity}.findUnique({ where: { id } })
  },

  create: async (data: Create{Entity}): Promise<{Entity}> => {
    return await db.{entity}.create({ data })
  },

  update: async (id: string, data: Update{Entity}): Promise<{Entity} | null> => {
    return await db.{entity}.update({ where: { id }, data })
  },

  delete: async (id: string): Promise<boolean> => {
    await db.{entity}.delete({ where: { id } })
    return true
  }
}
```

**注**: このファイルは `/bolt` で既に生成されているため、`/generate-api` では**生成しません**。既存のサービスを読み込んで、HTTP層（routes）のみを生成します。

**4. メインアプリケーション (`src/index.ts`)**

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

**5. クライアント型定義 (`src/types/index.ts`)**

```typescript
// フロントエンドから参照可能な型をエクスポート
export type { {Entity}, Create{Entity}, Update{Entity} } from '../schemas/{entity}'
```

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
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{Entity}'

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
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/{Entity}'
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

### ステップ5: package.json生成

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

## 出力

### 1. HTTP層（新規生成）

- `packages/api/src/routes/` - Honoルート定義
- `packages/api/src/schemas/` - Zodスキーマ
- `packages/api/src/index.ts` - Honoアプリケーション
- `packages/api/src/types/index.ts` - 型エクスポート
- `packages/api/package.json` - 依存関係（存在しなければ）
- `packages/api/tsconfig.json` - TypeScript設定（存在しなければ）

### 2. ドキュメント（新規生成）

- `docs/api/openapi.yaml` - OpenAPI仕様
- `docs/api/{unit}_API設計.md` - API設計ドキュメント

### 3. フロントエンド連携

**注**: `packages/api/src/services/` は `/bolt` で既に生成されているため、**生成しません**。

Hono RPCクライアント例：

```typescript
// apps/web/src/lib/api.ts
import { hc } from 'hono/client'
import type { AppType } from '@{project}/api'

const client = hc<AppType>('http://localhost:3000')

// 型安全なAPIコール
const res = await client.api.{entity}.$get()
const data = await res.json() // 型推論が効く
```

## 実行例

```bash
# 前提: /bolt unit1 で services層が実装済み
# packages/api/src/services/user.service.ts が存在

# コマンド実行
/generate-api unit1

# 出力例
✅ API HTTP層生成完了

## 既存のファイル（/bolt で生成済み）

- packages/api/src/services/user.service.ts ← 既存（変更なし）

## 生成されたファイル（/generate-api で生成）

- packages/api/src/index.ts
- packages/api/src/routes/user.ts
- packages/api/src/schemas/user.ts
- packages/api/src/types/index.ts
- packages/api/package.json（存在しなければ）
- docs/api/openapi.yaml
- docs/api/001-unit1_API設計.md

## 次のステップ

1. 依存関係のインストール:
   pnpm install

2. 開発サーバー起動:
   cd packages/api && pnpm dev

3. フロントエンドから利用:
   apps/web/src/lib/api.ts を参照

4. OpenAPI仕様の確認:
   docs/api/openapi.yaml を参照
```

## 注意事項

- **前提条件**: `/bolt unit1` が実行済みで、`packages/api/src/services/` にサービスが存在すること
- services層のファイルは**読み込むだけ**で、新規生成・上書きしない
- エンティティ名はPascalCaseに変換
- 認証・認可は別途実装が必要

## エラーハンドリング

- services層が見つからない場合はエラー（`/bolt`を先に実行してください）
- 不正なサービス定義がある場合は警告
- 既存のroutesファイルがある場合は確認してから上書き
