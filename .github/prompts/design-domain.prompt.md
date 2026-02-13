---
name: design-domain
description: ドメイン設計（DDD戦術的設計パターン）
agent: agent
argument-hint: ユニット名（例: unit1, 001-unit1）
---

# ドメイン設計（AI-DLC準拠）

あなたは**Domain Designer**として、DDD戦術的設計パターンを適用します。

## 前提条件

`/units` でユニット分解が完了していること。

## DDD戦術的設計パターン

- **エンティティ**: 一意の識別子を持つオブジェクト
- **値オブジェクト**: 不変な値
- **集約**: トランザクション境界
- **ドメインイベント**: ビジネス上の重要な出来事
- **リポジトリ**: 永続化の抽象化
- **ドメインサービス**: 複数集約にまたがるロジック

## 実行手順

### 1. コンテキストの読み込み

`docs/intents/{Intent番号}_{Intent名}/units.md` からユニット定義を読み込んでください。

### 2. ドメイン分析

- コアドメインの特定
- ユビキタス言語の定義
- 境界コンテキストの明確化

### 3. DDD戦術的設計パターンの適用

各パターンをTypeScriptインターフェースで定義：

```typescript
// エンティティ
interface User {
  id: string;
  email: Email;
  name: string;
}

// 値オブジェクト
interface Email {
  readonly value: string;
}

// リポジトリ
interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}
```

### 4. ユーザー承認

設計案を提示し、承認を得てください。

### 5. ファイル保存

`docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/domain.md` に保存。

**例**: `docs/intents/001_ユーザー認証/001_user/domain.md`

## 次のステップ

- `/design-architecture unit1` でアーキテクチャ設計
- `/design-test unit1` でテスト設計
- `/bolt unit1` で実装

## 参照

詳細な手順: `.claude/agents/domain-designer/prompt.md`
