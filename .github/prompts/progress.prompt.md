---
name: progress
description: プロジェクト/パッケージの現状確認
agent: agent
argument-hint: パッケージ名（省略時は全体）
---

# プロジェクト/パッケージの現状確認

指定されたパッケージ（または全体）の現状を確認し、サマリーを表示します。

## 実行手順

### 1. Git状況の確認

```bash
git status
git log --oneline -5
```

### 2. 実装計画の進捗確認

`docs/plans/` 配下の実装計画を読み込み：
- 完了タスク数
- 進行中タスク数
- 未着手タスク数
- ブロッカーの有無

### 3. テスト状況の確認

最新のテスト実行結果があれば表示。

## 出力形式

```markdown
## {パッケージ名} ステータス

### Git状況
- ブランチ: main
- 変更ファイル: X件
- 最新コミット: {hash} {message}

### 実装進捗
| フェーズ | ステータス |
|---------|-----------|
| Phase 1 | ✅ 完了 |
| Phase 2 | 🟡 進行中 |
| Phase 3 | ⏳ 未着手 |

### ブロッカー
🔴 {ブロッカー内容}

### 次のアクション
- {推奨アクション}
```

## 使用例

```bash
/progress           # 全体
/progress api       # apiパッケージ
/progress collector # collectorパッケージ
```
