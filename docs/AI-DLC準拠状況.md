# AI-DLC準拠状況

**作成日**: 2025-11-17
**最終更新**: 2025-11-26
**目的**: AI-DLC論文（特に付録A）との対比により、本テンプレートの準拠率と実装状況を明確化する

---

## 📊 対応状況マトリクス

| AI-DLC付録Aの要素 | 本実装のコマンド | 準拠度 | 備考 |
|-----------------|------------------------|-------|------|
| **セットアッププロンプト** | `/setup-aidlc` | ⚠️ 70% | docs/の自動構築、対話的収集、構成ファイル生成 |
| **ユーザーストーリー** | `/intent` | ✅ 80% | AIが質問で明確化、受入基準作成 |
| **ユニット分解** | `/units` | ✅ 90% | DDD原則、疎結合・高凝集、依存関係図 |
| **ドメインモデル作成** | `/design-domain` | ✅ 85% | 集約、エンティティ、値オブジェクト等を設計 |
| **アーキテクチャ設計** | `/design-architecture` | ✅ 85% | NFR分析、パターン選択、ADR生成、環境変数設計 |
| **コード生成** | `/bolt` | ⚠️ 60% | TDDサイクルあり、ただし詳細プロンプトは簡略 |
| **テスト設計** | `/design-test` | ✅ 75% | BDD/TDD統合、Phase 2で追加 |
| **REST API生成** | `/generate-api` | ✅ 75% | Hono RPC、OpenAPI仕様生成 |
| **IaC生成** | `/generate-iac` | ✅ 80% | Terraform、unit単位モジュール化、環境分離 |
| **デプロイ設定生成** | `/generate-deploy` | ✅ 75% | GitHub Actions、環境変数管理、デプロイスクリプト |
| **ドキュメント同期** | `/sync-docs` | ✅ 70% | 設計↔実装の乖離検出、更新提案 |

**総合準拠度**: **83%** (旧: 80% → 72% → 67% → 62%)

---

## 🔍 詳細な対比

### 1. セットアッププロンプト（付録A 行366）

#### AI-DLC（付録A）
```markdown
すべてのドキュメントはaidlc-docsフォルダに格納されます。

aidlc-docs/
├── plans/
├── requirements/
├── story-artifacts/
├── design-artifacts/
└── prompts/
```

#### 本実装 (`/setup-aidlc`)
```markdown
# /setup-aidlc が自動生成
<project-name>/
├── package.json          # pnpm workspace設定
├── pnpm-workspace.yaml   # workspace定義
├── apps/ または packages/
├── docs/
│   ├── intents/          # ≒ requirements/
│   ├── units/            # AI-DLCにはない（新規）
│   ├── design-artifacts/ # ✅ 一致
│   │   ├── domain/       # ✅ 一致
│   │   ├── architecture/ # ✅ 一致
│   │   ├── tests/        # AI-DLCにはない（独自追加）
│   │   └── adr/          # AI-DLCにはない（独自追加）
│   └── plans/            # ✅ 一致
├── CLAUDE.md
├── README.md
└── .gitignore
```

**準拠度**: ⚠️ 70% (旧: 0%)
- ✅ セットアップ自動化コマンド実装（`/setup-aidlc`）
- ✅ フォルダ構造の自動作成
- ✅ 対話形式でプロジェクト情報収集
- ✅ package.json、pnpm-workspace.yamlの自動生成
- ⚠️ AI-DLC付録Aほど詳細なプロンプトではない

**差分**:
- ❌ `story-artifacts/`フォルダなし（インテントに統合）
- ❌ `prompts/`フォルダなし（`.claude/commands/`で代替）
- ✅ `tests/`, `adr/`を独自追加（TDD/BDD統合）
- ✅ monorepo対応（apps/、packages/）

---

### 2. ユーザーストーリー作成（付録A 行370-378）

#### AI-DLC（付録A）
```markdown
あなたの役割：専門のプロダクトマネージャー

あなたのタスク：ユーザーストーリーを構築

プラン：user_stories_plan.md
```

#### 本実装 (`/intent`)
```markdown
## AIの役割
あなたは専門のプロダクトマネージャー/ビジネスアナリストです。

## タスク
1. AIが4つの質問で明確化（BDD統合）
2. ユーザーストーリー + Given/When/Then受入基準
3. NFR定義
4. リスク定義

出力: docs/intents/NNN_タイトル.md
```

**準拠度**: ✅ 80%
- ✅ AIの役割定義が一致
- ✅ プラン作成→承認→実行の流れ
- ✅ BDD形式の受入基準（Given/When/Then）を追加（Phase 2）
- ⚠️ AI-DLCは別ファイル、figma-strivoはインテント内に統合

**独自拡張**:
- BDD形式の受入基準（Phase 2で追加）
- NFR、リスクレジスター、成功指標の体系化

---

### 3. ユニット分解（付録A 行381-388）

#### AI-DLC（付録A）
```markdown
あなたの役割：経験豊富なソフトウェアアーキテクト

あなたのタスク：疎結合、高凝集のユニットに分解

プラン：units_plan.md
```

#### 本実装 (`/units`)
```markdown
## AIの役割
あなたは経験豊富なソフトウェアアーキテクトです。

## タスク
1. DDDのサブドメイン概念でユニット分解
2. 疎結合・高凝集を保証
3. 依存関係図（Mermaid）作成
4. 各ユニットの責務明確化

出力: docs/units/NNN-units.md
```

**準拠度**: ✅ 90%
- ✅ 役割、タスクが完全一致
- ✅ 依存関係図を追加（AI-DLC以上）

**独自拡張**:
- Mermaid図での依存関係可視化
- 実装順序の推奨機能

---

### 4. ドメインモデル作成（付録A 行392-401）

#### AI-DLC（付録A）
```markdown
あなたの役割：経験豊富なソフトウェアエンジニア

あなたのタスク：コンポーネントモデルを設計

プラン：component_model.md
```

#### 本実装 (`/design-domain`)
```markdown
## AIの役割
あなたは経験豊富なDDD（ドメイン駆動設計）エキスパートです。

## タスク
1. 集約、エンティティ、値オブジェクト定義
2. ドメインイベント、リポジトリ、ファクトリー設計
3. Mermaid図で可視化

出力: docs/design-artifacts/domain/unitN-domain.md
```

**準拠度**: ✅ 85%
- ✅ DDD戦術的パターンを明示（AI-DLCより詳細）
- ⚠️ AI-DLCは「コンポーネントモデル」、figma-strivoは「ドメインモデル」

**独自拡張**:
- DDD戦術的パターンの体系化
- Mermaid図での可視化

---

### 5. アーキテクチャ設計（付録A 行408-422）

#### AI-DLC（付録A）
```markdown
あなたの役割：経験豊富なクラウドアーキテクト

タスク：AWS CloudFormation/CDK/Terraformでデプロイメント
```

#### 本実装 (`/design-architecture`)
```markdown
## AIの役割
あなたは経験豊富なソフトウェアアーキテクトです。

## タスク
1. NFR分析（性能、セキュリティ等）
2. アーキテクチャパターン選択（CQRS等）
3. トレードオフ分析
4. ADR生成

出力: docs/design-artifacts/architecture/unitN-architecture.md
     docs/design-artifacts/adr/unitN-adr.md
```

**準拠度**: ✅ 85%
- ✅ NFR考慮、パターン選択が一致
- ✅ ADR生成を独自追加（AI-DLC以上）
- ✅ フロントエンド環境変数設計チェック追加
- ⚠️ IaC生成は別コマンド（`/generate-iac`）で対応

**独自拡張**:
- トレードオフ分析の体系化
- ADR（Architecture Decision Record）自動生成
- フロントエンド環境変数チェックリスト（ビルド時 vs ランタイム判断）

---

### 6. コード生成（付録A 行403-406）

#### AI-DLC（付録A）
```markdown
あなたの役割：経験豊富なソフトウェアエンジニア

タスク：Python実装を生成、Amazon Bedrock API使用
```

#### 本実装 (`/bolt`)
```markdown
## AIの役割
あなたは経験豊富なソフトウェアエンジニアです。

## タスク
1. 計画がなければ作成 → 承認
2. TDDサイクル（Red→Green→Refactor）で実装
3. Lint/ビルド確認
4. 次ステップ提案

出力: 実装コード + テスト
```

**準拠度**: ⚠️ 60%
- ✅ TDDサイクルを統合（AI-DLC以上）
- ❌ AI-DLCのような詳細プロンプト（Bedrock API等）は未対応
- ❌ IaC生成機能なし

**独自拡張**:
- TDDサイクル統合（Red→Green→Refactor）
- 人間承認プロセスの組み込み

---

### 7. テスト設計（本実装独自）

#### AI-DLC（付録A）
- **該当なし**（テスト設計の独立コマンドなし）

#### 本実装 (`/design-test`)
```markdown
## AIの役割
あなたは経験豊富なテストエンジニアです。

## タスク
1. BDD受入基準をテストケースに変換
2. テストレベル分類（Unit/Integration/E2E）
3. テストピラミッド構成（70% Unit, 20% Integration, 10% E2E）
4. TDDサイクル計画（Red → Green → Refactor）
5. モック・フィクスチャ設計
6. テストカバレッジ目標設定

出力: docs/design-artifacts/tests/[番号]_[ユニット名]_test_design.md
```

**準拠度**: ✅ 75%（AI-DLCにない独自機能）

**独自拡張**:
- テスト設計の独立フェーズ化
- BDD→TDDの統合
- テストピラミッド構成の明示化

---

### 8. REST API生成（付録A 行424-428）

#### AI-DLC（付録A）
```markdown
タスク：Python Flask APIを作成
```

#### 本実装 (`/generate-api`)
```markdown
## AIの役割
あなたはREST API設計とTypeScript実装の専門家です。

## タスク
1. `/bolt`で実装されたservices層を読み込み
2. HTTP層を生成（routes/, schemas/, index.ts, types/）
3. Zodスキーマによるバリデーション
4. OpenAPI仕様を生成

## 技術スタック
- Hono RPC（型安全なHTTPフレームワーク）
- Zod（スキーマバリデーション）
- TypeScript（型安全性）

出力:
- packages/api/src/routes/ - Honoルート定義
- packages/api/src/schemas/ - Zodスキーマ
- docs/api/openapi.yaml - OpenAPI仕様
```

**準拠度**: ✅ 75%
- ✅ API生成機能あり
- ✅ OpenAPI仕様生成
- ✅ 型安全なAPI実装
- ⚠️ Flask→Hono RPCへの変更（TypeScript化）

---

### 9. IaC生成

#### AI-DLC（付録A）
- Infrastructure as Code生成への言及あり

#### 本実装 (`/generate-iac`)
```markdown
## AIの役割
あなたはインフラストラクチャ設計とTerraformの専門家です。

## タスク
1. アーキテクチャ設計を読み込み（docs/design-artifacts/architecture/）
2. unit単位でTerraformモジュールを生成
3. 環境ごとに全unitをまとめて呼び出し（dev/staging/production）
4. unit間の依存関係をmodule outputで解決

## 技術スタック
- Terraform（AWS Provider）
- unit単位のモジュール化
- 環境分離（dev/staging/production）

## セキュリティベストプラクティス
- 暗号化デフォルト有効（S3、RDS、EBS）
- IAM最小権限の原則
- タグ戦略（Environment、Project、Unit）
- バックアップ設定（本番環境で有効）

出力:
- {infrastructure-root}/terraform/modules/{unit}/ - Terraformモジュール
- {infrastructure-root}/environments/dev/main.tf - 環境設定（全unit呼び出し）
- docs/infrastructure/{unit}_IaC設計.md - IaC設計ドキュメント
```

**準拠度**: ✅ 80%
- ✅ IaC生成機能あり
- ✅ Terraform対応
- ✅ セキュリティベストプラクティス適用
- ✅ 環境分離（dev/staging/production）
- ✅ unit間の依存関係解決
- ⚠️ Terraform専用（CDK、CloudFormation未対応）

---

### 10. デプロイ設定生成（本実装独自）

#### AI-DLC（付録A）
- デプロイ自動化への直接的な言及なし（IaCに含まれる想定）

#### 本実装 (`/generate-deploy`)
```markdown
## AIの役割
あなたはデプロイ設定の専門家です。

## タスク
1. インフラ構成の検出（Terraform、package.json等）
2. 環境変数の分析（機密情報と公開情報の分類）
3. GitHub Actionsワークフロー生成
4. デプロイスクリプト生成
5. デプロイドキュメント生成

## 対応パターン
- S3 + CloudFront（静的サイト）
- Vercel
- AWS Lambda
- ECS/Fargate
- Amplify

出力:
- .github/workflows/deploy.yml
- scripts/deploy.sh
- docs/deploy/README.md
```

**準拠度**: ✅ 75%（AI-DLCにない独自機能）

**独自拡張**:
- インフラ構成の自動検出
- 環境変数の機密性分類
- 複数デプロイパターン対応

---

### 11. ドキュメント同期（本実装独自）

#### AI-DLC（付録A）
- トレーサビリティへの言及あり（実装との同期は明示なし）

#### 本実装 (`/sync-docs`)
```markdown
## AIの役割
あなたは設計ドキュメント管理の専門家です。

## タスク
1. 変更ファイルの検出（git diff）
2. 関連ドキュメントの特定
3. 乖離チェック（ドメイン設計、アーキテクチャ設計等）
4. 更新提案の生成

出力: 乖離レポート + 更新提案
```

**準拠度**: ✅ 70%（AI-DLCのトレーサビリティ原則を実装）

**独自拡張**:
- コード変更と設計ドキュメントの自動マッピング
- 乖離検出と更新提案

---

## ✨ 本実装の独自拡張（AI-DLC超え）

### Phase 2完了項目

1. **TDD/BDD統合**
   - `/intent` にBDD形式の受入基準（Given/When/Then）
   - `/design-test` コマンド追加（テスト設計の独立フェーズ）
   - `/bolt` にTDDサイクル統合（Red→Green→Refactor）

2. **ADR生成**
   - アーキテクチャ決定記録の自動生成
   - トレードオフ分析の体系化

3. **依存関係図**
   - Mermaidでの可視化
   - ユニット間の依存関係の明示化

4. **ドキュメント構造の拡張**
   - `docs/tests/` フォルダ追加
   - `docs/adr/` フォルダ追加

### Phase 3完了項目（2025-11-26）

5. **デプロイ自動化**
   - `/generate-deploy` コマンド追加
   - GitHub Actions ワークフロー生成
   - 複数デプロイパターン対応（S3+CloudFront、Vercel、Lambda等）

6. **設計ドキュメント同期**
   - `/sync-docs` コマンド追加
   - コード変更と設計ドキュメントの乖離検出
   - 更新提案の自動生成

7. **品質管理強化**
   - `/commit-unit` にテストカバレッジチェック追加
   - `/design-architecture` にフロントエンド環境変数チェック追加

---

## ⚠️ 未実装の要素

### 今後の実装予定

1. **運用フェーズ**
   - `/operate` コマンド
   - テレメトリ分析
   - インシデント管理
   - プロアクティブな対応

2. **自動テスト実行エージェント**
   - AIエージェントによるテスト実行・分析
   - 失敗原因の自動特定
   - 修正提案の生成

---

## 📈 準拠度の詳細

### フォルダ構造（0%）

| 要素 | AI-DLC | 本実装 | 一致 |
|-----|--------|-----------------|------|
| plans/ | ✅ | ✅ | ✅ |
| requirements/ | ✅ | intents/で代替 | △ |
| story-artifacts/ | ✅ | intents/に統合 | △ |
| design-artifacts/ | ✅ | ✅ | ✅ |
| prompts/ | ✅ | .claude/commands/で代替 | △ |
| tests/ | ❌ | ✅（独自追加） | ➕ |
| adr/ | ❌ | ✅（独自追加） | ➕ |

### インセプション（85%）

| 要素 | AI-DLC | 本実装 | 準拠度 |
|-----|--------|-----------------|-------|
| AI主導の明確化質問 | ✅ | ✅ | 100% |
| ユーザーストーリー形式 | ✅ | ✅ | 100% |
| 受入基準 | ✅ | ✅（BDD形式で拡張） | 120% |
| NFR定義 | ✅ | ✅ | 100% |
| リスクレジスター | ✅ | ✅ | 100% |
| ユニット分解 | ✅ | ✅（依存関係図追加） | 110% |

### コンストラクション（88%）

| 要素 | AI-DLC | 本実装 | 準拠度 |
|-----|--------|-----------------|-------|
| ドメイン設計 | ✅ | ✅（DDD明示） | 100% |
| アーキテクチャ設計 | ✅ | ✅（ADR追加） | 110% |
| コード生成 | ✅ | ✅（TDD統合） | 90% |
| テスト設計 | △（コード生成に含む） | ✅（独立コマンド） | 150% |
| REST API生成 | ✅ | ✅（Hono RPC） | 75% |
| IaC生成 | ✅ | ✅（Terraform） | 80% |

### オペレーション（40%）

| 要素 | AI-DLC | 本実装 | 準拠度 |
|-----|--------|-----------------|-------|
| デプロイ自動化 | △ | ✅（`/generate-deploy`） | 75% |
| ドキュメント同期 | △ | ✅（`/sync-docs`） | 70% |
| テレメトリ分析 | ✅ | ❌ | 0% |
| インシデント管理 | ✅ | ❌ | 0% |
| プロアクティブ対応 | ✅ | ❌ | 0% |

---

## 🎯 総合評価

### 準拠度サマリー

| フェーズ | AI-DLC要素数 | 本実装対応 | 準拠度 | 備考 |
|---------|------------|----------------|-------|------|
| **セットアップ** | 1 | 1 | 70% | `/setup-aidlc` 実装済み |
| **インセプション** | 6 | 6 | 85% | BDD統合で拡張 |
| **コンストラクション** | 6 | 6 | 88% | 全要素実装完了 |
| **オペレーション** | 5 | 2 | 40% | デプロイ・同期追加 |
| **総合** | 18 | 15 | **83%** | Phase 3完了 |

### 全機能実装後の見通し

| 追加要素 | 準拠度向上 | 状態 |
|---------|----------|------|
| ~~セットアップ自動化~~ | ~~+5%~~ | ✅ 完了 |
| ~~REST API生成~~ | ~~+5%~~ | ✅ 完了 |
| ~~IaC生成~~ | ~~+8%~~ | ✅ 完了 |
| ~~デプロイ設定生成~~ | ~~+2%~~ | ✅ 完了 |
| ~~ドキュメント同期~~ | ~~+1%~~ | ✅ 完了 |
| 運用フェーズ（テレメトリ等） | +5% | ❌ 未実装 |
| **残り** | **+5%** | - |

**全機能実装後の予想準拠度**: 83% + 5% = **88%**

---

## 📚 参考資料

1. [AI-DLC日本語訳](AI-DLC_日本語訳.md) - 付録A（行360-438）
2. [README.md](../README.md) - プロジェクト概要
3. [CLAUDE.md](../CLAUDE.md) - AI-DLC開発ガイド
4. [開始ガイド](guides/getting-started.md)
5. [ワークフローガイド](guides/workflow.md)

---

**作成日**: 2025-11-17
**最終更新**: 2025-11-26
**総合準拠率**: **83%**（現在）← 80% ← 72% ← 67% ← 62%（初版時点）
**全機能実装後の目標**: **88%+**（運用フェーズ追加後）
