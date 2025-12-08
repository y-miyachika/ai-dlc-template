# プロジェクトテンプレート別ガイド

このガイドでは、プロジェクトタイプ（モノレポ、シングルアプリ、マイクロサービス）ごとのAI-DLC適用方法を説明します。

## プロジェクトタイプの選択

### 判断フローチャート

```
プロジェクト開始
      │
      ▼
┌─────────────────────┐
│ 複数のデプロイ単位   │
│ が必要?              │
└──────────┬──────────┘
           │
     ┌─────┴─────┐
    Yes         No
     │           │
     ▼           ▼
┌─────────┐   ┌─────────────────┐
│複数チーム│   │ シングルアプリ   │
│で開発?   │   │ を選択          │
└────┬────┘   └─────────────────┘
     │
  ┌──┴──┐
 Yes    No
  │      │
  ▼      ▼
┌────────────┐  ┌──────────┐
│マイクロ     │  │ モノレポ  │
│サービス     │  │ を選択    │
└────────────┘  └──────────┘
```

### タイプ比較

| 項目 | シングルアプリ | モノレポ | マイクロサービス |
|-----|--------------|---------|----------------|
| **チーム規模** | 1-3人 | 3-10人 | 10人以上 |
| **デプロイ** | 一括 | 個別可能 | 完全独立 |
| **依存管理** | シンプル | 中程度 | 複雑 |
| **AI-DLC適用** | 最もシンプル | 推奨 | 適用可能 |

---

## 1. シングルアプリ

### 概要

単一のアプリケーションとしてデプロイされるプロジェクト。

**適用例**:
- Webアプリケーション（Next.js、Remix）
- APIサーバー（Hono、Express）
- CLIツール
- Lambda関数（単体）

### ディレクトリ構造

```
my-app/
├── .claude/
│   ├── commands/          # AI-DLCコマンド
│   ├── agents/            # SubAgents
│   └── skills/            # Skills
├── src/
│   ├── domain/            # ドメイン層
│   │   ├── entities/
│   │   ├── value-objects/
│   │   └── services/
│   ├── infrastructure/    # インフラ層
│   │   └── repositories/
│   └── presentation/      # プレゼンテーション層
│       └── routes/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/
│   ├── intents/           # AI-DLC成果物
│   ├── units/
│   ├── design-artifacts/
│   │   ├── domain/
│   │   ├── architecture/
│   │   ├── tests/
│   │   └── adr/
│   └── plans/
├── terraform/             # IaC（必要な場合）
│   ├── modules/
│   └── environments/
├── package.json
├── tsconfig.json
├── CLAUDE.md
└── README.md
```

### AI-DLC適用フロー

```bash
# 1. セットアップ
/setup-aidlc my-app

# 2. インテント定義
/intent ユーザー認証機能

# 3. ユニット分解
/units 001

# 4. 設計（順次実行）
/design-domain unit1
/design-architecture unit1
/design-test unit1

# 5. 実装
/bolt unit1

# 6. API/IaC生成（必要な場合）
/generate-api unit1
/generate-iac unit1
```

### 注意点

- すべてのユニットが1つの `src/` に配置される
- 依存関係は同一パッケージ内で解決
- テストは `tests/` に集約

---

## 2. モノレポ（推奨）

### 概要

複数のパッケージを1つのリポジトリで管理。pnpm workspaceを使用。

**適用例**:
- フロントエンド + バックエンド
- 共有ライブラリを含むプロジェクト
- 複数のLambda関数

### ディレクトリ構造

```
my-project/
├── .claude/
│   ├── commands/
│   ├── agents/
│   ├── skills/
│   └── skill-config.json  # Skill設定
├── packages/
│   ├── api/               # バックエンド
│   │   ├── src/
│   │   ├── tests/
│   │   └── package.json
│   ├── web/               # フロントエンド
│   │   ├── src/
│   │   ├── tests/
│   │   └── package.json
│   ├── shared/            # 共有ライブラリ
│   │   ├── src/
│   │   └── package.json
│   └── infrastructure/    # IaC
│       ├── terraform/
│       ├── environments/
│       └── package.json
├── docs/                  # AI-DLC成果物（ルートに集約）
│   ├── intents/
│   ├── units/
│   ├── design-artifacts/
│   │   ├── domain/
│   │   ├── architecture/
│   │   ├── tests/
│   │   └── adr/
│   └── plans/
├── tests/
│   └── e2e/               # E2Eテスト（クロスパッケージ）
├── package.json           # ルートpackage.json
├── pnpm-workspace.yaml
├── CLAUDE.md
└── README.md
```

### AI-DLC適用フロー

```bash
# 1. セットアップ
/setup-aidlc my-project  # → モノレポを選択

# 2. インテント定義
/intent ユーザー認証機能

# 3. ユニット分解
/units 001
# → unit1: api（認証API）
# → unit2: web（ログイン画面）
# → unit3: shared（認証共通ライブラリ）

# 4. 共通部分から設計・実装
/design-domain unit3    # shared
/bolt unit3

# 5. 依存順にAPI → Web
/design-domain unit1    # api
/design-architecture unit1
/design-test unit1
/bolt unit1

/design-domain unit2    # web
/bolt unit2

# 6. 統合
/generate-api unit1
/generate-iac unit1
```

### 注意点

- ドキュメントはルートの `docs/` に集約（横断的な参照のため）
- ユニットは `packages/` 内のパッケージに対応
- 依存順序を意識（shared → api → web）
- E2Eテストはルートの `tests/e2e/` に配置

### Skill設定

`.claude/skill-config.json`:

```json
{
  "api-generator": {
    "outputDir": "packages/api",
    "docsDir": "docs/api"
  },
  "iac-generator": {
    "outputDir": "packages/infrastructure/terraform"
  },
  "deploy-generator": {
    "workflowsDir": ".github/workflows",
    "scriptsDir": "scripts",
    "docsDir": "docs/deploy"
  }
}
```

---

## 3. マイクロサービス

### 概要

複数の独立したサービスで構成されるシステム。各サービスは独自のリポジトリを持つことも可能。

**適用例**:
- 大規模Webサービス
- イベント駆動アーキテクチャ
- 複数チームでの開発

### ディレクトリ構造（単一リポジトリ版）

```
my-platform/
├── .claude/
│   ├── commands/
│   ├── agents/
│   └── skills/
├── services/
│   ├── user-service/
│   │   ├── src/
│   │   ├── tests/
│   │   ├── terraform/     # サービス固有のIaC
│   │   ├── docs/          # サービス固有のドキュメント
│   │   └── package.json
│   ├── order-service/
│   │   ├── src/
│   │   ├── tests/
│   │   ├── terraform/
│   │   ├── docs/
│   │   └── package.json
│   └── payment-service/
│       └── ...
├── shared/
│   ├── contracts/         # サービス間契約（OpenAPI、Protobuf）
│   ├── events/            # イベントスキーマ
│   └── libraries/         # 共有ライブラリ
├── infrastructure/
│   ├── terraform/
│   │   ├── modules/
│   │   │   ├── networking/    # 共通ネットワーク
│   │   │   ├── messaging/     # SQS、EventBridge
│   │   │   └── observability/ # 監視基盤
│   │   └── environments/
│   └── kubernetes/        # K8s設定（使用する場合）
├── docs/
│   ├── intents/           # プラットフォーム全体のIntent
│   ├── units/
│   ├── design-artifacts/
│   │   ├── domain/
│   │   ├── architecture/
│   │   ├── tests/
│   │   └── adr/
│   └── api-contracts/     # サービス間API仕様
├── tests/
│   └── contract/          # 契約テスト
├── pnpm-workspace.yaml
├── CLAUDE.md
└── README.md
```

### AI-DLC適用フロー

```bash
# 1. セットアップ
/setup-aidlc my-platform  # → マイクロサービスを選択

# 2. インテント定義（プラットフォームレベル）
/intent 注文処理システム

# 3. ユニット分解（サービス単位）
/units 001
# → unit1: user-service（ユーザー管理）
# → unit2: order-service（注文管理）
# → unit3: payment-service（決済処理）
# → unit4: shared/events（イベント定義）

# 4. 共通基盤から設計
/design-architecture unit4  # 共通イベントスキーマ
/bolt unit4

# 5. 各サービスを並列で設計・実装
# チームAが担当
/design-domain unit1
/design-architecture unit1
/design-test unit1
/bolt unit1

# チームBが並行して担当
/design-domain unit2
/design-architecture unit2
/design-test unit2
/bolt unit2

# 6. 統合テスト
# 契約テストを実行
pnpm test:contract

# 7. インフラデプロイ
/generate-iac unit1
/generate-iac unit2
/generate-deploy unit1
/generate-deploy unit2
```

### 注意点

- **サービス間通信**: イベント駆動 or REST/gRPC
- **契約テスト**: サービス間の契約を検証
- **インフラ分離**: 各サービスが独自のIaCを持つ
- **デプロイ独立性**: サービスごとに独立してデプロイ可能

### サービス間契約の管理

`shared/contracts/order-service.yaml`:

```yaml
openapi: 3.0.0
info:
  title: Order Service API
  version: 1.0.0
paths:
  /orders:
    post:
      summary: Create order
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrder'
```

`shared/events/order-created.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "eventType": { "const": "ORDER_CREATED" },
    "orderId": { "type": "string" },
    "customerId": { "type": "string" },
    "items": { "type": "array" },
    "createdAt": { "type": "string", "format": "date-time" }
  },
  "required": ["eventType", "orderId", "customerId", "items", "createdAt"]
}
```

---

## テンプレート選択チェックリスト

### シングルアプリを選ぶ場合

- [ ] チーム規模が1-3人
- [ ] デプロイ単位が1つ
- [ ] 外部依存が少ない
- [ ] シンプルさを優先

### モノレポを選ぶ場合

- [ ] フロントエンド + バックエンドがある
- [ ] 共有ライブラリが必要
- [ ] チーム規模が3-10人
- [ ] コード共有のメリットが大きい

### マイクロサービスを選ぶ場合

- [ ] チーム規模が10人以上
- [ ] サービスごとに独立したデプロイが必要
- [ ] チームごとの自律性が重要
- [ ] スケーラビリティ要件が高い

---

## 参考資料

- [ワークフローDAG](./workflow-dag.md) - コマンド間の依存関係
- [実装スコープ](./implementation-scope.md) - 完了条件
- [Skill共通設定](./.claude/skills/config.md) - 出力パスのカスタマイズ
