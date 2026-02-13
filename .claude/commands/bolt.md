# ボルト実行（AI-DLC準拠）

引数として受け取ったユニット番号をもとに、**高速反復サイクル（Bolt）**を実行します。

## 入力内容
{{ARGS}}

---

## Boltとは

AI-DLCにおける**最小の反復サイクル**です：
- 1つのユニット、または複数のユーザーストーリーを実装
- 設計→実装→テストを1サイクルで完結
- 各ステップで人間の承認を得る
- 時間スケール：数時間〜数日

---

## 動作フロー

### ステップ0: Boltサイクル全体のタスク登録

`TaskCreate` APIを使って、Boltサイクルの主要ステップをタスク登録する：

```
TaskCreate: subject="テスト設計の確認", activeForm="テスト設計を確認中"
TaskCreate: subject="実装計画の作成（Plan Mode）", activeForm="実装計画を作成中"
TaskCreate: subject="Phase 1: Red（失敗テスト作成）", activeForm="失敗テストを作成中"
TaskCreate: subject="Phase 2: Green（実装）", activeForm="実装中"
TaskCreate: subject="Phase 3: Refactor", activeForm="リファクタリング中"
```

これにより、ユーザーに進捗スピナーが表示され、全体の見通しが明確になる。
各ステップ開始時に `TaskUpdate(status: "in_progress")`、完了時に `TaskUpdate(status: "completed")` を実行する。

---

### ステップ1: テスト設計の確認（TDD統合）✨NEW

1. `docs/design-artifacts/tests/{Intent}-{Unit}-{名前}.md` が存在するか確認
   - 例: `docs/design-artifacts/tests/002-001-login-api.md`
   - 命名規則: Intent番号3桁-Unit番号3桁-ユニット名
2. **存在しない場合**: `/design-test` 相当の処理を実行
   - 受入基準（BDD）をテストケースに変換
   - テストレベル分類（Unit/Integration/E2E）
   - TDDサイクル計画の作成
   - ユーザーにテスト設計を提示
3. **存在する場合**: 既存テスト設計を読み込み

---

### ステップ2: 実装計画の作成（EnterPlanMode統合）

Claude Codeの **Plan Mode** を活用して、コードベース探索と計画作成を一体化する。

#### 2a. 既存計画の確認

`docs/plans/{Intent}-{Unit}-{名前}.md` が存在するか確認：
- 例: `docs/plans/002-001-login-api.md`
- 命名規則: Intent番号3桁-Unit番号3桁-ユニット名

#### 2b. 計画が存在しない場合 → EnterPlanMode

**`EnterPlanMode` を呼び出してPlan Modeに入る。**

Plan Mode内で以下を実行：

1. **コードベース探索**（Glob/Grep/Readツール）
   - 既存のディレクトリ構造・ファイル構成を確認
   - 関連する既存コード（型定義、インターフェース等）を読み込み
   - 依存パッケージや設定ファイルの状態を確認

2. **計画の設計**（テスト設計を参照しながら）
   - Phase分割（ドメイン層→インフラ層→API層など）
   - 各Phaseのタスク一覧
   - 実装スコープの明確化

3. **計画ファイルに書き出し**
   計画の出力先を `docs/plans/{Intent}-{Unit}-{名前}.md` に設定し、以下の必須項目を含める：
   - 対象ユニット、見積もり、関連ドキュメント
   - Phase分割と各Phaseのタスク一覧（チェックリスト形式）
   - 実装スコープ（下記参照）
   - 完了状況セクション（実装後に更新）

4. **`ExitPlanMode` で承認を得る**
   Plan Modeの専用UIでユーザーが計画をレビュー・承認する。
   承認されたら自動的に実装フェーズに移行。

#### 2c. 計画が存在する場合

既存計画を読み込み、内容に問題がなければそのまま使用。
更新が必要な場合は `EnterPlanMode` で修正。

**重要**: 実装計画ファイルなしでの実装開始は禁止。計画ファイルは実装の進捗管理と振り返りに必須。

---

#### 実装スコープ（計画に含める）

計画ファイル内に以下のスコープを明記する：

**基本スコープ**:
- [ ] ドメイン層（エンティティ、値オブジェクト、集約）
- [ ] アプリケーション層（ユースケース、サービス）
- [ ] インフラ層（リポジトリ実装、外部API統合）
- [ ] **Handlers層（Lambda/APIエントリーポイント）** ← 特に重要
- [ ] 設定ファイル（環境変数、依存性注入）

**Lambda関数がある場合**:
- [ ] Orchestrator Handler実装
- [ ] Worker Handler実装
- [ ] Lambda固有の設定（timeout、memory、environment）
- [ ] デプロイ可能なzipファイル生成準備

**外部API統合がある場合**:
- [ ] 実装コード完成（モックだけでなく実コードも）
- [ ] エラーハンドリング実装
- [ ] リトライロジック実装（該当する場合）

---

### ステップ3: TDDサイクル実行（Red → Green → Refactor）

Plan Mode承認後、以下をTDDサイクルで実行：

#### Phase 1: Red（失敗するテストを書く）

1. **テストファイル作成**
   - テスト設計を参照し、テストコードを生成
   - `tests/[ユニット名]/[機能名].test.ts`
   - BDD形式（Given/When/Then）でテストケースを記述

2. **テスト実行（失敗確認）**
   ```bash
   pnpm test [テストファイル]
   ```
   - すべてのテストが失敗することを確認
   - 失敗理由が期待通りか確認（実装がないため）

#### Phase 2: Green（最小限の実装で通す）

1. **TaskCreate APIでタスク登録**
   - テストを通すための最小限のタスクを抽出
   - `TaskCreate` で各タスクを登録（subject、description、activeForm を設定）
   - タスク間の依存関係がある場合は `TaskUpdate` の `addBlockedBy` で設定
   - 例:
     ```
     TaskCreate: subject="ドメイン層実装", activeForm="ドメイン層を実装中"
     TaskCreate: subject="インフラ層実装", activeForm="インフラ層を実装中"
     TaskUpdate: taskId=2, addBlockedBy=[1]  # ドメイン層完了後に開始
     ```

2. **段階的実装（TaskUpdateで進捗管理）**
   - 各タスクの開始時に `TaskUpdate(status: "in_progress")` で状態遷移
   - ユーザーにリアルタイムでスピナー表示（activeForm）
   - タスク完了時に `TaskUpdate(status: "completed")` で完了
   - `TaskList` で全体進捗を確認しながら次のタスクへ
   - テストが Green になるまで実装

3. **テスト実行（成功確認）**
   ```bash
   pnpm test [テストファイル]
   ```
   - すべてのテストが通ることを確認

4. **Phase完了時のIntegration Test実行**（🆕）

   Phase 2に進む前、またはPhase完了時に、Integration Testを実行：

   ```bash
   # Unit Test（常に実行）
   pnpm test:unit

   # Integration Test（Phase完了時に実行）
   pnpm test:integration
   ```

   **確認事項**:
   - [ ] 全てのUnit Testがパス
   - [ ] 全てのIntegration Testがパス（LocalStack使用）
   - [ ] コンポーネント間の連携が正常

   **テストが失敗した場合**: 次のPhaseに進まず、現在のPhaseの実装を修正

#### Phase 3: Refactor（リファクタリング）

1. **コード品質改善**
   - 重複コードの削除
   - 関数の分割・統合
   - 命名の改善

2. **Lint/ビルド確認**
   ```bash
   pnpm lint
   pnpm build
   ```
   - エラーがあれば修正

3. **テスト実行（回帰確認）**
   ```bash
   pnpm test
   ```
   - リファクタリング後もテストが通ることを確認

4. **実装完了チェックリスト（🆕）**

   以下を確認してから完了報告：

   **コード品質チェック**:
   - [ ] すべてのpublicメソッドが実装済み（TODOコメントなし）
   - [ ] すべてのテストがパス
   - [ ] カバレッジ目標達成（80%以上推奨）
   - [ ] Lint/ビルドエラーなし

   **外部統合チェック（該当する場合）**:
   - [ ] 外部APIクライアントが実装済み（モックだけでなく実コードも）
   - [ ] エラーハンドリングが実装済み
   - [ ] リトライロジックが実装済み（該当する場合）

   **Lambda関数チェック（該当する場合）**:
   - [ ] Handlers実装済み（Orchestrator、Worker等）
   - [ ] Lambda固有設定完了（timeout、memory、environment）
   - [ ] デプロイ可能なzipファイル生成可能

   **チェックリストが全て✅であることを確認してから次へ**

5. **完了報告**
   - 実装内容のサマリー
   - 変更されたファイル一覧
   - テストカバレッジレポート
   - 実装完了チェックリスト結果
   - 次のステップ提案

---

### ステップ4: 次のステップ提案（改善版）

実装完了後、以下を提案：

✅ **実装完了！次のステップ**:

1. **コミット推奨**（このタイミングで）:
   ```bash
   git add .
   git commit -m "機能追加: {unit}の実装完了（Phase 1-3）

   ## 実装内容
   - ドメイン層: [実装したエンティティ等]
   - インフラ層: [実装したリポジトリ等]
   - テスト: Unit/Integration Test（カバレッジ XX%）

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   "
   ```

2. **振り返り（推奨）**:
   - `/retro` で会話を振り返り、CLAUDE.mdやコマンドの改善提案を受ける
   - 得られた知見を次のUnitに活かす

3. **次の選択肢**:
   - [ ] 他ユニットの実装（依存関係に基づく）
   - [ ] E2Eテストの追加（**デプロイ後に実行**、`packages/e2e/`に配置）
   - [ ] `/generate-api` でREST API生成
   - [ ] `/generate-iac` でインフラ生成

4. **コミット単位の推奨**:
   - **unit単位**（推奨）: 各unitのPhase 1-3完了後
   - **phase単位**: 大きなunitの場合、Phase 1→2→3で分割コミット
   - **feature単位**: 複数unitで1つの機能の場合、全unit完了後

**重要**: テストが全てパスし、実装完了チェックリストが全て✅であることを確認してからコミットしてください。

---

## 注意事項

- **人間の承認は必須**: 実装開始前に必ず承認を得る
- **git commitは手動**: 自動コミットしない（ユーザーに確認）
- **エラー時の対応**: エラーが発生した場合は即座に報告し、対応を相談

---

## AI-DLC原則との対応

| 原則 | 実装方法 |
|-----|---------|
| **会話の逆転** | Plan Modeで計画を提示し、専用UIで承認を得る |
| **損失関数** | TDDで早期に欠陥を検出（Red → Green） |
| **コンテキストメモリ** | テスト設計・計画・実装結果をdocs/に永続化 |
| **段階的詳細化** | BDD受入基準 → テスト → 実装 |
| **ステージ最小化** | テスト設計→計画→TDD実装を1サイクルで |

---

**作成日**: 2025-11-12
**更新日**: 2025-11-13（TDD/BDD統合）
**AI-DLC準拠**: コンストラクションフェーズ（TDD/BDD統合版）
