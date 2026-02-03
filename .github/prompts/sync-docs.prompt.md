---
name: sync-docs
description: 設計ドキュメントと実装の同期チェック
agent: agent
argument-hint: パッケージ名またはUnit番号（省略時は全体）
---

# 設計ドキュメント同期チェック

変更されたファイルと関連する設計ドキュメントを検出し、更新提案を行います。

## 実行手順

### 1. 変更ファイルの検出

```bash
git diff --name-only HEAD~1   # 直近コミット
git diff --name-only --staged # ステージング
git diff --name-only HEAD     # 未コミット
```

### 2. 関連ドキュメントの特定

| 変更ファイル | 関連ドキュメント |
|------------|----------------|
| `src/domain/**` | `docs/design-artifacts/domain/*.md` |
| `src/services/**` | `docs/design-artifacts/domain/*.md` |
| `src/routes/**` | `docs/api/openapi.yaml` |
| `terraform/**` | `docs/design-artifacts/architecture/*.md` |
| `*.test.ts` | `docs/design-artifacts/tests/*.md` |

### 3. 乖離チェック

**ドメイン設計との乖離**:
- 新しいエンティティ/値オブジェクトが追加されていないか
- ドメインモデルの構造が変更されていないか
- 新しいドメインイベントが追加されていないか

**アーキテクチャ設計との乖離**:
- 新しいコンポーネントが追加されていないか
- データフローが変更されていないか
- 技術スタックが変更されていないか

**テスト設計との乖離**:
- 新しいテストケースが追加されていないか

### 4. 結果レポート

```markdown
## 同期チェック結果

### 変更されたファイル
- `packages/api/src/domain/entities/User.ts` (modified)
- `packages/api/src/domain/entities/UserProfile.ts` (added)

### 関連ドキュメント
| ドキュメント | ステータス | 必要なアクション |
|------------|----------|----------------|
| `docs/.../domain.md` | ⚠️ 更新推奨 | UserProfile追加 |

### 更新提案
[具体的な更新内容]
```

### 5. ユーザー確認

```
上記の更新を適用しますか？
1. すべて適用
2. 選択して適用
3. 後で手動対応
```

## 使用例

```bash
/sync-docs        # 全体チェック
/sync-docs api    # 特定パッケージ
/sync-docs 001    # 特定Unit
```
