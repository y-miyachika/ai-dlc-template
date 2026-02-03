---
applyTo: "**/*.ts,**/*.tsx"
---

# TypeScript コーディング規約

このファイルは TypeScript ファイル（`.ts`, `.tsx`）に適用されるインストラクションです。

## 基本原則

### strict modeの使用

- `tsconfig.json` で `"strict": true` を設定
- 型推論に頼りすぎず、明示的な型定義を優先

### any型の禁止

- `any` 型の使用は禁止
- 代わりに `unknown` を使用し、型ガードで絞り込む

```typescript
// ❌ 悪い例
function process(data: any): void {
  console.log(data.name);
}

// ✅ 良い例
function process(data: unknown): void {
  if (isUser(data)) {
    console.log(data.name);
  }
}
```

### classを使わない

- 関数ベースの設計を優先
- 依存性注入は引数で渡す
- 状態管理はクロージャまたはオブジェクトで

```typescript
// ❌ 悪い例
class UserService {
  private repository: UserRepository;

  constructor(repository: UserRepository) {
    this.repository = repository;
  }

  async findById(id: string): Promise<User | null> {
    return this.repository.findById(id);
  }
}

// ✅ 良い例
interface UserService {
  findById: (id: string) => Promise<User | null>;
}

function createUserService(repository: UserRepository): UserService {
  return {
    findById: (id) => repository.findById(id),
  };
}
```

## 型定義

### インターフェースの使用

- 型定義には `interface` を優先（拡張が容易）
- ユニオン型や交差型が必要な場合は `type` を使用

```typescript
// インターフェース
interface User {
  id: string;
  name: string;
  email: Email;
}

// 型エイリアス（ユニオン型）
type Status = 'pending' | 'completed' | 'failed';
```

### 値オブジェクトの型定義

- 不変性を保証するために `readonly` を使用

```typescript
interface Email {
  readonly value: string;
}

function createEmail(value: string): Email {
  if (!isValidEmail(value)) {
    throw new Error('Invalid email format');
  }
  return { value };
}
```

## 関数

### 関数の型注釈

- 引数と戻り値に明示的な型を付ける
- 推論可能でも、公開APIには型を付ける

```typescript
// ✅ 明示的な型注釈
async function findUserById(id: string): Promise<User | null> {
  // ...
}
```

### Result型の使用（推奨）

- エラーハンドリングには例外よりも Result 型を使用

```typescript
type Result<T, E = Error> =
  | { success: true; value: T }
  | { success: false; error: E };

function divide(a: number, b: number): Result<number> {
  if (b === 0) {
    return { success: false, error: new Error('Division by zero') };
  }
  return { success: true, value: a / b };
}
```

## 非同期処理

### async/await の使用

- Promise よりも async/await を優先
- エラーハンドリングは try/catch または Result 型

```typescript
// ✅ async/await
async function fetchUser(id: string): Promise<User | null> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) return null;
    return response.json();
  } catch {
    return null;
  }
}
```

## インポート/エクスポート

### 名前付きエクスポートの優先

- デフォルトエクスポートよりも名前付きエクスポートを優先
- 型と値は分離してエクスポート

```typescript
// ✅ 名前付きエクスポート
export { createUserService };
export type { User, UserService };
```

### インポート順序

1. 外部ライブラリ
2. 内部モジュール（@/）
3. 相対パス
4. 型のみのインポート

```typescript
import { z } from 'zod';
import { createEmail } from '@/domain/value-objects';
import { UserRepository } from '../repositories';
import type { User } from './types';
```

## テスト

### テストファイルの配置

- テストファイルは `tests/` ディレクトリに配置
- `*.test.ts` の命名規則

### BDD形式のテスト

- Given/When/Then コメントを使用

```typescript
describe('UserService', () => {
  it('should find user by id', async () => {
    // Given: ユーザーが存在する
    const user = createTestUser({ id: 'user-1' });
    await repository.save(user);

    // When: IDで検索する
    const result = await userService.findById('user-1');

    // Then: ユーザーが返される
    expect(result).toEqual(user);
  });
});
```

## Zod によるバリデーション

### スキーマ定義

```typescript
import { z } from 'zod';

export const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

export type CreateUser = z.infer<typeof CreateUserSchema>;
```

## エラーハンドリング

### カスタムエラークラス

```typescript
class DomainError extends Error {
  constructor(
    message: string,
    public readonly code: string,
  ) {
    super(message);
    this.name = 'DomainError';
  }
}

class UserNotFoundError extends DomainError {
  constructor(userId: string) {
    super(`User not found: ${userId}`, 'USER_NOT_FOUND');
  }
}
```

## その他

### コメント

- 自明なコードにはコメントを書かない
- 「なぜ」を説明するコメントを書く
- JSDoc は公開APIにのみ使用

### 命名規則

- 変数・関数: camelCase
- 型・インターフェース: PascalCase
- 定数: UPPER_SNAKE_CASE
- ファイル名: kebab-case
