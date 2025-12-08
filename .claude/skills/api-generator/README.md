# API Generator Skill

Hono + Zod + TypeScript ベースの REST API 生成 Skill

## 概要

既存のビジネスロジック（services層）から、型安全なREST API層（HTTP層）を自動生成します。

## 使い方

### 前提条件

- ビジネスロジック（services層）が実装済み
- 例: `packages/api/src/services/user.service.ts`

### 実行

```bash
# AI-DLCテンプレートのスラッシュコマンドから
/generate-api unit1

# または、Skillを直接呼び出し
# （他プロジェクトでも使用可能）
```

### 生成されるファイル

```
packages/api/
├── src/
│   ├── index.ts              # Honoアプリケーション
│   ├── routes/               # HTTPエンドポイント
│   │   └── user.ts
│   ├── schemas/              # Zodバリデーション
│   │   └── user.ts
│   ├── services/             # 既存（変更なし）
│   │   └── user.service.ts
│   └── types/                # 型エクスポート
│       └── index.ts
└── package.json
```

## 技術スタック

- **Hono**: 高速なWeb framework
- **Zod**: バリデーションライブラリ
- **TypeScript**: 型安全性
- **Hono RPC**: フロントエンドとの型共有

## 特徴

### 1. 型安全性

フロントエンドからバックエンドまで、一貫した型定義：

```typescript
// バックエンド（packages/api/）
export const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  email: z.string().email()
})

export type User = z.infer<typeof UserSchema>

// フロントエンド（apps/web/）
import { hc } from 'hono/client'
import type { AppType } from '@project/api'

const client = hc<AppType>('http://localhost:3000')
const res = await client.api.users.$get()
const data = await res.json() // 型推論が効く
```

### 2. 自動バリデーション

Zodスキーマによる自動バリデーション：

```typescript
// 不正なリクエスト
POST /api/users
{
  "email": "invalid-email"
}

// レスポンス
400 Bad Request
{
  "error": "Invalid email format"
}
```

### 3. OpenAPI仕様生成

`docs/api/openapi.yaml` が自動生成されます。

## カスタマイズ

出力パスのカスタマイズは[Skill共通設定ガイド](../config.md)を参照してください。

### 認証・認可の追加

```typescript
// src/middleware/auth.ts
export const authMiddleware = async (c, next) => {
  const token = c.req.header('Authorization')
  // トークン検証
  await next()
}

// src/routes/user.ts
import { authMiddleware } from '../middleware/auth'

app.use('*', authMiddleware) // 認証を追加
```

### エラーハンドリングのカスタマイズ

```typescript
app.onError((err, c) => {
  console.error(err)
  return c.json({ error: err.message }, 500)
})
```

## 他プロジェクトでの利用

このSkillは、AI-DLCテンプレート以外のプロジェクトでも利用可能です：

1. `.claude/skills/api-generator/` をコピー
2. プロジェクトに配置
3. Skillを呼び出し

```bash
# 他プロジェクトで
claude skill api-generator --services-path src/services
```

## バージョン履歴

- **1.0.0** (2025-11-19): 初版リリース
  - Hono + Zod ベースのAPI生成
  - OpenAPI仕様生成
  - Hono RPCクライアント対応

## 品質基準

出力品質を確保するため、[品質チェックリスト](../../agents/quality-checklist.md)を参照してください。

## ライセンス

MIT
