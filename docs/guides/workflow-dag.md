# AI-DLC コマンド依存関係図（DAG）

このガイドでは、AI-DLCスラッシュコマンド間の依存関係を図示し、正しい実行順序と並列実行可能な組み合わせを説明します。

## 全体依存関係図

```mermaid
graph TD
    subgraph "セットアップ"
        setup["/setup-aidlc"]
    end

    subgraph "インセプションフェーズ"
        intent["/intent"]
        units["/units"]
    end

    subgraph "コンストラクションフェーズ - 設計"
        domain["/design-domain"]
        arch["/design-architecture"]
        test["/design-test"]
    end

    subgraph "コンストラクションフェーズ - 実装"
        bolt["/bolt"]
    end

    subgraph "コード生成"
        api["/generate-api"]
        iac["/generate-iac"]
        deploy["/generate-deploy"]
    end

    subgraph "管理・同期"
        commit["/commit-unit"]
        progress["/progress"]
        sync["/sync-docs"]
        retro["/retro"]
    end

    %% 依存関係
    setup --> intent
    intent --> units
    units --> domain
    units --> arch
    units --> test
    domain --> bolt
    arch --> bolt
    test --> bolt
    bolt --> api
    arch --> iac
    arch --> deploy
    bolt --> commit
    api --> commit
    iac --> commit
    deploy --> commit
```

## コマンド別依存関係

### 1. セットアップ

| コマンド | 前提条件 | 出力 |
|---------|---------|------|
| `/setup-aidlc` | なし（初回のみ） | package.json, pnpm-workspace.yaml, docs/ |

### 2. インセプションフェーズ

| コマンド | 前提条件 | 出力 |
|---------|---------|------|
| `/intent` | `/setup-aidlc` 完了 | docs/intents/{番号}_{タイトル}.md |
| `/units` | `/intent` で Intent 作成済み | docs/units/{Intent番号}_ユニット分解.md |

### 3. コンストラクションフェーズ - 設計

| コマンド | 前提条件 | 出力 | 並列実行 |
|---------|---------|------|---------|
| `/design-domain` | `/units` 完了 | docs/design-artifacts/domain/ | 可能* |
| `/design-architecture` | `/units` 完了 | docs/design-artifacts/architecture/, adr/ | 可能* |
| `/design-test` | `/units` 完了 | docs/design-artifacts/tests/ | 可能* |

**\*注意**: 3つの設計コマンドは**同一ユニットに対しては順次実行を推奨**。異なるユニット間では並列実行可能。

```mermaid
flowchart LR
    subgraph same["推奨順序（同一ユニット）"]
        D1["/design-domain unit1"] --> A1["/design-architecture unit1"] --> T1["/design-test unit1"]
    end
```

```mermaid
flowchart TB
    subgraph parallel["並列可能（異なるユニット）"]
        D1["/design-domain unit1"] --> A1["/design-architecture unit1"]
        D2["/design-domain unit2"] --> A2["/design-architecture unit2"]
    end
```

### 4. コンストラクションフェーズ - 実装

| コマンド | 前提条件 | 出力 |
|---------|---------|------|
| `/bolt` | 3つの設計コマンド完了 | src/, tests/, docs/plans/ |

### 5. コード生成

| コマンド | 前提条件 | 出力 | 並列実行 |
|---------|---------|------|---------|
| `/generate-api` | `/bolt` で services 層実装済み | packages/api/, docs/api/ | 可能 |
| `/generate-iac` | `/design-architecture` 完了 | terraform/ | 可能 |
| `/generate-deploy` | `/design-architecture` 完了 | .github/workflows/, scripts/ | 可能 |

```
並列実行可能:
/generate-api unit1  |  /generate-iac unit1  |  /generate-deploy unit1
```

## 詳細フロー図

### 新規プロジェクト開発フロー

```mermaid
flowchart TB
    setup["/setup-aidlc"] --> intent["/intent 機能概要"]
    intent --> units["/units 001"]

    units --> domain["/design-domain unit1"]
    units --> arch["/design-arch unit1"]
    units --> test["/design-test unit1"]

    domain --> bolt["/bolt unit1"]
    arch --> bolt
    test --> bolt

    bolt --> api["/generate-api unit1"]
    bolt --> iac["/generate-iac unit1"]
    bolt --> deploy["/generate-deploy unit1"]

    api --> commit["/commit-unit 1"]
    iac --> commit
    deploy --> commit

    commit --> next["次のユニット（unit2）へ"]
```

### マルチユニット並列開発フロー

複数ユニットが**独立**している場合、並列開発が可能です：

```mermaid
flowchart TB
    units["/units 001"] --> unit1["unit1: 認証ドメイン"]
    units --> unit2["unit2: ユーザー管理"]

    subgraph Unit1Flow["Unit1 フロー"]
        unit1 --> d1["/design-domain unit1"]
        d1 --> a1["/design-architecture unit1"]
        a1 --> t1["/design-test unit1"]
        t1 --> b1["/bolt unit1"]
    end

    subgraph Unit2Flow["Unit2 フロー"]
        unit2 --> d2["/design-domain unit2"]
        d2 --> a2["/design-architecture unit2"]
        a2 --> t2["/design-test unit2"]
        t2 --> b2["/bolt unit2"]
    end

    b1 --> integration["統合テスト実行"]
    b2 --> integration
    integration --> commit["/commit-unit（両ユニット）"]
```

**注意**: 依存関係があるユニット（例: unit3 → unit1）は順次実行が必要。

## SubAgent/Skill の使い分け

```mermaid
flowchart TB
    subgraph SubAgents["SubAgents（深い思考・対話型）"]
        intent["intent-definer"] --> units["units-decomposer"]
        units --> domain["domain-designer"]
        domain --> arch["architecture-designer"]
        intent --> test["test-designer"]
        arch --> test
    end

    subgraph Skills["Skills（コード生成型）"]
        api["api-generator"]
        iac["iac-generator"]
        deploy["deploy-generator"]
        api --> code["実装コード生成"]
        iac --> code
        deploy --> code
    end

    SubAgents --> Skills
```

## 実行順序の判断フローチャート

```mermaid
flowchart TB
    Start["開始"] --> Q1{"プロジェクトは<br/>セットアップ済み?"}
    Q1 -->|No| setup["/setup-aidlc"]
    setup --> Q1
    Q1 -->|Yes| Q2{"Intent は<br/>定義済み?"}

    Q2 -->|No| intent["/intent 概要"]
    intent --> Q2
    Q2 -->|Yes| Q3{"Unit に<br/>分解済み?"}

    Q3 -->|No| units["/units {Intent番号}"]
    units --> Q3
    Q3 -->|Yes| Q4{"設計3種は<br/>完了?"}

    Q4 -->|No| design["/design-domain → /design-architecture → /design-test"]
    design --> Q4
    Q4 -->|Yes| Q5{"実装は完了?"}

    Q5 -->|No| bolt["/bolt unit{N}"]
    bolt --> Q5
    Q5 -->|Yes| Q6{"API/IaC/Deploy<br/>が必要?"}

    Q6 -->|No| commit["/commit-unit"]
    Q6 -->|Yes| generate["/generate-api, /generate-iac, /generate-deploy"]
    generate --> commit

    commit --> Q7{"次のUnitあり?"}
    Q7 -->|Yes| Q4
    Q7 -->|No| End["完了"]
```

## 補助コマンドの使用タイミング

| コマンド | 使用タイミング | 依存関係 |
|---------|---------------|---------|
| `/progress` | いつでも | なし |
| `/blocker` | 実装中にブロッカー発生時 | `/bolt` 実行中 |
| `/sync-docs` | 実装後、コミット前 | `/bolt` 完了後 |
| `/retro` | 会話終了時、フェーズ完了時 | なし |

## よくある間違い

### NG: 設計なしで実装

```bash
# ❌ 設計をスキップして bolt
/units 001
/bolt unit1  # エラー: 設計ドキュメントがありません
```

### NG: Intent なしでユニット分解

```bash
# ❌ Intent 未定義
/units  # エラー: Intent が見つかりません
```

### NG: services 層なしで API 生成

```bash
# ❌ bolt で services 層が未実装
/generate-api unit1  # 警告: services 層が見つかりません
```

## 参考資料

- [実装スコープ](./implementation-scope.md) - 各コマンドの完了条件
- [E2Eテストフロー](./e2e-testing-flow.md) - テスト実行タイミング
- [ワークフローガイド](./workflow.md) - フェーズ詳細
- [開始ガイド](./getting-started.md) - 最初のステップ
- [ベストプラクティス](./best-practices.md) - 推奨パターン
