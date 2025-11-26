# Unit実装完了時のコミット支援

Unit実装完了時に、変更内容を要約してコミットメッセージを生成します。

## 引数
- Unit番号またはパッケージ名（例: `2`, `api`, `commit-api`）

## 実施内容

### 1. 変更内容の確認
```bash
git status
git diff --stat
```

### 1.5. テストカバレッジチェック（警告のみ）

**新規/変更ファイルに対応するテストがあるか確認**

以下のパターンでテストファイルの存在をチェック：
- `src/services/foo.ts` → `src/services/foo.test.ts` または `tests/services/foo.test.ts`
- `src/domain/entities/bar.ts` → `src/domain/entities/bar.test.ts`

**テストが見つからない場合**:
```
⚠️ テストカバレッジ警告

以下のファイルに対応するテストが見つかりません:
- src/services/newFeature.ts
- src/domain/entities/newEntity.ts

コミットは可能ですが、テストの追加を検討してください。
続行しますか？ [y/N]
```

**注意**: これは警告のみで、コミットを強制的にブロックしません。

### 2. コミットメッセージの生成

以下の形式でコミットメッセージを生成：

```
feat: Unit {番号} {パッケージ名} 実装

## 実装内容
- {主要な変更点1}
- {主要な変更点2}
- {主要な変更点3}

## テスト
- Unit Test {件数}件 通過
- Integration Test {件数}件 通過

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### 3. ユーザー確認

コミットメッセージを表示し、ユーザーの承認を得てからコミット実行。

## 使用例

```bash
/commit-unit 2
/commit-unit api
```

## 注意事項

- ユーザーの承認なしにコミットしない
- テスト結果を確認してからコミット
- 変更がない場合はコミットしない
