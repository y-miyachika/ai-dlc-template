# アーキテクチャ設計（AI-DLC準拠）

このコマンドは**Architecture Designer SubAgent**を使用して、NFRを考慮した最適なアーキテクチャを設計します。

## SubAgentについて

`architecture-designer` SubAgentは、NFR（非機能要件）を満たす最適なアーキテクチャパターンを選択し、トレードオフを明示し、ADR（Architecture Decision Record）を作成する専門SubAgentです。

**アーキテクチャ設計の原則**: NFR駆動、トレードオフ分析、ADRによる記録

詳細は `.claude/agents/architecture-designer/README.md` を参照してください。

## 前提条件

**このコマンドは `/design-domain` の後に実行してください**

- `/design-domain` でドメイン設計が完了している
- `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/domain.md` にドメイン設計が存在する
- `docs/intents/{Intent番号}_{Intent名}/intent.md` にNFRが定義されている

## 入力内容

{{ARGS}}

## SubAgent起動

**推奨SubAgentタイプ: Plan**

アーキテクチャ設計はNFR駆動で段階的に詳細化するため、`Plan` subagent_type が最適です。
Task toolを使用する場合は `subagent_type: "Plan"` を指定してください。

**既存コードベースがある場合**:
アーキテクチャ調査の前段階で、`Explore` subagent_typeを使って既存コード構造を探索することを推奨します。

### EnterPlanMode統合

アーキテクチャ設計は複数の選択肢からトレードオフを比較して決定するため、**`EnterPlanMode` で Plan Mode に入ってから設計を進める**。

**Plan Mode内で実行する内容：**

1. **前提情報の収集**: Backlog/Intent、ユニット定義、ドメイン設計を読み込み
2. **既存コードベースの探索**（Glob/Grep/Read）: 既存のアーキテクチャパターン、技術スタック、ディレクトリ構造を把握
3. **NFR分析**: 非機能要件をカテゴリ別に整理、優先度を確認
4. **アーキテクチャパターンの候補列挙**: 3つのオプションを選定
5. **トレードオフ分析**: 各オプションをNFR、コスト、リスク観点で評価
6. **推奨オプションの決定**: 最適なパターンを推奨、トレードオフを明示
7. **アーキテクチャ詳細設計**: コンポーネント構成、データフロー、技術スタック
8. **ADR生成**: Architecture Decision Recordを作成

**`ExitPlanMode` で設計案の承認を得る** → 承認後にファイル保存：
- `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md` にアーキテクチャ設計
- `docs/adr/` にADR

---

**実行する処理**:

引数として受け取ったユニット名をもとに、Architecture Designer SubAgentを起動します。

**ユニット名**: {{ARGS}}

**ドメイン設計パス**: `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/domain.md`

**出力先**:
- `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md`
- `docs/adr/`

---

## SubAgent処理の詳細

SubAgentは `.claude/agents/architecture-designer/prompt.md` に定義された手順に従って処理を実行します。

**主要な処理**:

1. 前提情報の収集（Backlog、ユニット定義、ドメイン設計）
2. NFR分析
   - カテゴリ別整理（パフォーマンス、スケーラビリティ、可用性等）
   - 優先度設定（High/Medium/Low）
   - 制約条件の抽出
3. アーキテクチャパターンの候補列挙
   - レイヤードアーキテクチャ
   - ヘキサゴナルアーキテクチャ
   - クリーンアーキテクチャ
   - イベント駆動アーキテクチャ
   - サーバーレスアーキテクチャ
4. トレードオフ分析
   - NFR評価（⭐1-5）
   - 実装コスト（Simple/Medium/Large）
   - リスク評価
   - ドメインモデルとの適合性
5. 推奨オプションの決定
6. アーキテクチャ詳細設計
   - コンポーネント構成図（Mermaid）
   - レイヤー構成
   - データフロー
   - ディレクトリ構造
7. 技術スタック選定
8. ADR（Architecture Decision Record）生成

**設計詳細**: `.claude/agents/architecture-designer/prompt.md` を参照

---

## NFR（非機能要件）カテゴリ

### パフォーマンス（Performance）
- API応答時間、スループット、レイテンシ

### スケーラビリティ（Scalability）
- 同時接続数、データ量の増加対応

### 可用性（Availability）
- 稼働率、ダウンタイム許容値

### セキュリティ（Security）
- 認証、認可、暗号化、脆弱性対策

### 保守性（Maintainability）
- コードカバレッジ、技術的負債の管理

### 観測可能性（Observability）
- ログ、メトリクス、トレース

### 信頼性（Reliability）
- エラー率、リカバリー時間

### デプロイ容易性（Deployability）
- 環境変数管理、ビルド・デプロイプロセス
- **フロントエンド環境変数**: ビルド時埋め込み vs ランタイム取得の判断

### 外部API制約（External API Constraints）
- **レート制限**: APIの呼び出し回数制限（例: GitHub API 5000req/hour）
- **同時接続数制限**: 並列リクエストの上限
- **クォータ**: 日次/月次の呼び出し上限
- **対策パターン**: 同時実行数制限、Exponential Backoff、キュー + バッチ処理

**⚠️ 重要**: 外部APIを呼び出すアーキテクチャでは、必ずレート制限を確認し、同時実行数を保守的に設定すること（例: Lambda 100→2-5）

---

## アーキテクチャパターン

### レイヤードアーキテクチャ（Layered Architecture）
- **適用場面**: CRUD中心、シンプルなビジネスロジック
- **メリット**: シンプル、実装コスト低
- **デメリット**: 複雑化すると保守困難

### ヘキサゴナルアーキテクチャ（Ports & Adapters）
- **適用場面**: 外部システムとの連携が多い
- **メリット**: テストしやすい、外部変更に強い
- **デメリット**: 初期実装コスト高

### クリーンアーキテクチャ（Clean Architecture）
- **適用場面**: ビジネスロジックが複雑、長期保守
- **メリット**: ビジネスロジックが独立、技術的変更に強い
- **デメリット**: 学習コスト高、初期実装コスト高

### イベント駆動アーキテクチャ（Event-Driven Architecture）
- **適用場面**: ユニット間連携、非同期処理
- **メリット**: 疎結合、スケーラビリティ高
- **デメリット**: 複雑性高、デバッグ困難

### サーバーレスアーキテクチャ（Serverless）
- **適用場面**: 短時間処理、トラフィック変動
- **メリット**: スケーラビリティ自動、運用コスト低
- **デメリット**: コールドスタート、実行時間制限

---

## 生成されるファイル

### アーキテクチャ設計ドキュメント

- `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md` - アーキテクチャ設計

**パス例**: `docs/intents/002_注文管理/001_order-management/architecture.md`

### ADR（Architecture Decision Record）

- `docs/adr/ADR-{連番}_{タイトル}.md` - アーキテクチャ決定記録

**ファイル名例**: `ADR-001_クリーンアーキテクチャの採用.md`

---

## NFR駆動の設計原則

### ✅ ベストプラクティス

**NFRを最優先**:
```
NFR: API応答3秒以内
  ↓
パターン選択: 段階的取得アーキテクチャ
  ↓
実装: depth指定、部分取得、キャッシュ
```

**トレードオフの明示**:
```
採用: オプションB（段階的取得）
トレードオフ:
- 複雑性が増す（許容）
- 初期実装コスト高（NFR達成のため必要）
```

### ❌ アンチパターン

**NFRを無視した設計**:
```
パターン選択: 一括取得（シンプル）
  ↓
NFR未達: 10秒かかる（要件違反）
```

**トレードオフの隠蔽**:
```
採用: オプションA（シンプル）
理由: 実装が簡単
（NFRへの言及なし）
```

---

## 実行後の次のステップ

```bash
# テスト設計
/design-test unit1

# ボルト実行
/bolt unit1
```

---

## 注意事項

1. **NFRを最優先**: シンプルさよりもNFR達成を優先
2. **トレードオフの明示**: デメリットを隠さず、受け入れ理由を説明
3. **過度な抽象化を避ける**: 将来の要件を推測せず、現在のNFRを満たす設計
4. **ドメインモデルの尊重**: アーキテクチャパターンがドメインモデルを歪めないこと

---

## AI-DLC原則との対応

| 原則 | 実装方法 |
|-----|---------|
| **損失関数** | NFR未達を早期検出（設計段階で） |
| **段階的詳細化** | ドメインモデル → アーキテクチャ → 実装 |
| **設計技術の統合** | DDD原則を保ちつつアーキテクチャ選択 |
| **トレーサビリティ** | NFR → アーキテクチャ選択 → ADR |

---

**SubAgent Version**: 1.0.0
**SubAgent Location**: `.claude/agents/architecture-designer/`
**作成日**: 2025-11-19
**AI-DLC準拠**: コンストラクションフェーズ（論理設計・アーキテクチャ設計）
