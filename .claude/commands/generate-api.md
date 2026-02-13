# REST API実装生成（Hono RPC）

このコマンドは**api-generator Skill**を使用して、REST API層を自動生成します。

## Skillについて

`api-generator` Skillは、既存のビジネスロジック（services層）から型安全なREST API層（HTTP層）を生成する再利用可能なSkillです。

**技術スタック**: Hono + Zod + TypeScript

詳細は `.claude/skills/api-generator/README.md` を参照してください。

## 前提条件

**このコマンドは `/bolt` の後に実行してください**

- `/bolt unit1` でビジネスロジック（services層）が実装済み
- `packages/api/src/services/*.service.ts` が存在する

## 入力内容

{{ARGS}}

## Skill起動

### EnterPlanMode統合

コード生成の前に **`EnterPlanMode` で Plan Mode に入り、生成計画を立てる**。

**Plan Mode内で実行する内容：**

1. **services層の探索**（Glob/Grep/Read）
   - `packages/api/src/services/` 配下のサービスファイルを読み込み
   - 公開メソッド、引数の型、戻り値の型を抽出
   - 既存のroutes/schemas/があれば確認（上書き範囲の把握）
2. **エンドポイントマッピング計画**
   - 各サービスメソッド → HTTPエンドポイントの対応表を作成
   - HTTPメソッド（GET/POST/PUT/DELETE）の選定
   - パスパラメータ、クエリパラメータの設計
3. **生成ファイル一覧の提示**
   - 新規作成/上書きされるファイルのリスト
   - 既存ファイルへの影響範囲

**`ExitPlanMode` で生成計画の承認を得る** → 承認後にコード生成を実行。

---

**実行する処理**:

承認後、引数として受け取ったユニット名をもとに、api-generator Skillを起動します。

**ユニット名**: {{ARGS}}

**services層のパス**: `packages/api/src/services/`

**ドメイン設計パス**: `docs/design-artifacts/domain/` （存在する場合）

**出力先**: `packages/api/`

---

## Skill処理の詳細

Skillは `.claude/skills/api-generator/prompt.md` に定義された手順に従って処理を実行します。

**主要な処理**:

1. 既存のservices層を読み込み、公開メソッド・型情報を抽出
2. HTTPエンドポイントへのマッピング（GET/POST/PUT/DELETE）
3. Zodスキーマの生成（バリデーションルール）
4. Honoルート定義の生成（routes/）
5. 型エクスポートの生成（types/）
6. Honoアプリケーションのエントリーポイント生成（index.ts）
7. OpenAPI仕様の生成（docs/api/openapi.yaml）
8. API設計ドキュメントの生成（docs/api/{unit}_API設計.md）

**コード生成テンプレート詳細**: `.claude/skills/api-generator/prompt.md` を参照

---

## 生成されるファイル

### 1. HTTP層（新規生成）

- `packages/api/src/routes/{entity}.ts` - Honoルート定義
- `packages/api/src/schemas/{entity}.ts` - Zodスキーマ
- `packages/api/src/index.ts` - Honoアプリケーション
- `packages/api/src/types/index.ts` - 型エクスポート
- `packages/api/package.json` - 依存関係（存在しなければ）
- `packages/api/tsconfig.json` - TypeScript設定（存在しなければ）

### 2. ドキュメント（新規生成）

- `docs/api/openapi.yaml` - OpenAPI仕様
- `docs/api/{unit}_API設計.md` - API設計ドキュメント

### 3. 既存ファイル（変更なし）

- `packages/api/src/services/` - `/bolt` で生成済み（読み込むだけ）

---

## フロントエンド連携

生成されたAPIは、Hono RPCクライアントで型安全に利用できます：

```typescript
// apps/web/src/lib/api.ts
import { hc } from 'hono/client'
import type { AppType } from '@{project}/api'

const client = hc<AppType>('http://localhost:3000')

// 型安全なAPIコール
const res = await client.api.{entity}.$get()
const data = await res.json() // 型推論が効く
```

---

## 実行後の次のステップ

```bash
# 1. 依存関係のインストール
pnpm install

# 2. 開発サーバー起動
cd packages/api && pnpm dev

# 3. OpenAPI仕様の確認
cat docs/api/openapi.yaml
```

---

## 注意事項

- **前提条件**: `/bolt {unit}` が実行済みで、`packages/api/src/services/` にサービスが存在すること
- services層のファイルは**読み込むだけ**で、新規生成・上書きしません
- 既存のroutesファイルがある場合は確認してから上書き
- 認証・認可は別途実装が必要

---

**Skill Version**: 1.0.0
**Skill Location**: `.claude/skills/api-generator/`
