# AI-DLC ベストプラクティス

このドキュメントは、AI-DLCテンプレートを使用する際のベストプラクティスをまとめたものです。

**作成日**: 2025-11-19
**対象**: AI-DLC準拠開発を実践する開発者

---

## コミット戦略

### 推奨タイミング

実際のプロジェクトで明らかになった課題：
- `/bolt` 完了後にコミットタイミングが不明確だった
- unit単位かphase単位か、判断基準がなかった

**改善**: 以下の基準でコミットタイミングを明確化

#### 1. unit単位コミット（推奨）

**タイミング**:
- `/bolt {unit}` 完了後（Phase 1-3完了）
- テストが全てパス
- 実装完了チェックリスト達成

**メリット**:
- 明確な区切り
- ロールバックが容易
- コードレビューがしやすい

**コミットメッセージ例**:
```bash
git add .
git commit -m "$(cat <<'EOF'
機能追加: {unit名}の実装完了（Phase 1-3）

## 実装内容
- ドメイン層: {エンティティ、集約}
- インフラ層: {リポジトリ実装}
- テスト: Unit/Integration Test（カバレッジ XX%）

## 次のステップ
- 次のunitの実装
- E2Eテスト追加（デプロイ後）

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

#### 2. phase単位コミット（大きなunitの場合）

**タイミング**:
- Phase 1完了後: ドメイン層
- Phase 2完了後: インフラ層
- Phase 3完了後: 統合テスト

**メリット**:
- 段階的な進捗確認
- 途中での方向転換が容易

**コミットメッセージ例**:
```bash
git commit -m "$(cat <<'EOF'
機能追加: commit-collector Phase 1完了（ドメイン層）

## 実装内容
- Repository集約（Root）
- Commit値オブジェクト
- Branch値オブジェクト
- ドメインイベント定義

## 次のステップ
- Phase 2: インフラ層実装

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

#### 3. feature単位コミット（複数unit構成の場合）

**タイミング**:
- 全unit完了後
- E2Eテスト完了後
- ドキュメント更新完了後

**メリット**:
- 1つの機能として完結
- 複数unitの整合性が保証される

---

## テストレベルの定義

### 課題

実際のプロジェクトで発生した混乱：
- E2E vs Integration Testの区別が曖昧だった
- LocalStackを使ったテストを「E2E」と呼んでいた
- 実AWS APIを使うテストをどう呼ぶか不明確だった

### 改善: 実行環境による明確な定義

| テストレベル | スコープ | 実行する場所 | 対象環境 | 実行タイミング | 前提条件 |
|------------|---------|------------|---------|--------------|---------|
| **Unit Test** | 1クラス/1関数 | ローカル、CI/CD | なし（モック） | いつでも | なし |
| **Integration Test** | 複数コンポーネント統合 | ローカル、CI/CD | LocalStack/Testcontainers | Phase完了時、コミット前、PR作成時 | LocalStack起動 |
| **E2E Test** | システム全体のユーザーシナリオ | **ローカル、CI/CD** | **デプロイ先環境（dev/staging）** | デプロイ完了後 | デプロイ済み、実環境稼働中 |

**重要な理解**:
- E2Eテストは**どこで実行するか**ではなく、**何に対して実行するか**が重要
- ❌ よくある誤解: E2E = CI/CDでしか実行しない
- ✅ 正しい理解: E2E = デプロイ済み実環境に対して実行（**ローカルからでもCI/CDからでもOK**）

**実際の開発フロー**:
```bash
# 開発者のローカルマシン

# 1. 実装
/bolt unit1

# 2. dev環境にデプロイ
cd packages/infrastructure
terraform apply

# 3. ローカルからE2E実行（対象はdev環境）
cd ../../
pnpm test:e2e:dev  # ← ローカルで実行、対象はhttps://dev-api.example.com

# 結果: FAIL - Lambda timeoutエラー

# 4. すぐ修正 → 再デプロイ → 再度E2E実行
# CI/CD待たずに高速デバッグ可能
```

**LocalStackとの違い**:
- LocalStack: ローカルで**モック環境**を起動 → Integration Test
- E2E: ローカルから**実環境**にアクセス → E2E Test

---

### アーキテクチャ別のテスト定義

#### Lambda / Event-Driven

**Integration Test** （ローカル実行）:
- LocalStack DynamoDB/SQS
- Lambda関数をローカル実行
- モックを使用

**E2E Test** （デプロイ先実行）:
```typescript
// デプロイ済みのLambda関数を実AWS APIで呼び出し
const lambdaClient = new LambdaClient({ region: 'ap-northeast-1' })
await lambdaClient.invoke({
  FunctionName: 'my-app-dev-orchestrator' // デプロイ済み
})

// 実DynamoDBに保存されているか確認
const dynamoClient = new DynamoDBClient({ region: 'ap-northeast-1' })
const item = await dynamoClient.getItem({
  TableName: 'my-app-dev-commits' // デプロイ済み
})
```

#### REST API

**Integration Test** （ローカル実行）:
- ローカルサーバー起動（localhost:3000）
- In-Memory DB
- モックを使用

**E2E Test** （デプロイ先実行）:
```typescript
// デプロイ済みのAPIエンドポイントにアクセス
const API_URL = 'https://dev-api.example.com'
const response = await fetch(`${API_URL}/api/users`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'test@example.com' })
})
```

#### Web Application

**Integration Test** （ローカル実行）:
- ローカルサーバー起動（localhost:3000）
- Playwright でlocalhost にアクセス

**E2E Test** （デプロイ先実行）:
```typescript
// デプロイ済みのサイトにPlaywrightでアクセス
const APP_URL = 'https://dev.example.com'
await page.goto(`${APP_URL}/register`)
await page.fill('[name="email"]', 'test@example.com')
await page.click('button[type="submit"]')
```

#### Batch / CLI

**Integration Test** （ローカル実行）:
- ローカル実行
- Testcontainers でDB起動

**E2E Test** （デプロイ先実行）:
```typescript
// 実S3にテストデータをアップロード
await s3.putObject({
  Bucket: 'prod-data-bucket',
  Key: 'test-batch-input.csv',
  Body: 'test,data,here'
})

// デプロイ済みのバッチジョブをトリガー
const jobId = await triggerBatchJob({ inputKey: 'test-batch-input.csv' })
```

---

### テスト実装の配置

```
packages/
├── {unit}/
│   ├── src/
│   └── tests/
│       ├── unit/           # Unit Test（ローカル実行）
│       │   └── *.test.ts
│       └── integration/    # Integration Test（ローカル実行、LocalStack）
│           └── *.test.ts
└── e2e/                    # E2E Test専用パッケージ
    ├── package.json
    ├── tests/
    │   ├── lambda.e2e.test.ts       # Lambda E2E（デプロイ先実行）
    │   ├── api.e2e.test.ts          # REST API E2E（デプロイ先実行）
    │   └── web.e2e.test.ts          # Web E2E（デプロイ先実行）
    ├── playwright.config.ts          # Web E2Eの場合
    └── .env.dev                      # デプロイ先環境の設定
```

**重要**: E2Eテストは専用パッケージに分離し、デプロイ先の環境変数を使用します。

---

### CI/CDパイプライン統合

E2Eテストは**デプロイ後に実行**する必要があるため、以下の順序を推奨：

```yaml
# GitHub Actions例
jobs:
  # 1. ローカルで実行可能なテスト
  unit-and-integration:
    runs-on: ubuntu-latest
    steps:
      - run: pnpm test:unit
      - run: docker compose up -d localstack
      - run: pnpm test:integration

  # 2. dev環境にデプロイ
  deploy-dev:
    needs: unit-and-integration
    runs-on: ubuntu-latest
    steps:
      - run: terraform apply -auto-approve

  # 3. デプロイ後にE2Eテスト実行 ← 重要
  e2e-dev:
    needs: deploy-dev
    runs-on: ubuntu-latest
    env:
      TEST_ENV: dev
      API_URL: https://dev-api.example.com
    steps:
      - name: Wait for deployment
        run: until curl -f $API_URL/health; do sleep 5; done
      - run: pnpm test:e2e:dev

  # 4. staging環境へのデプロイ（mainブランチのみ）
  deploy-staging:
    needs: e2e-dev
    if: github.ref == 'refs/heads/main'
    steps:
      - run: terraform apply -auto-approve

  # 5. staging環境でもE2E実行
  e2e-staging:
    needs: deploy-staging
    env:
      TEST_ENV: staging
    steps:
      - run: pnpm test:e2e:staging
```

---

## 実装完了の定義

### 課題

実際のプロジェクトで発生しやすい問題：
- 外部APIクライアントがTODOのまま完了報告
- Lambda Handlers が未実装のまま `/generate-iac` 実行
- X-Rayが有効なまま（POC環境で不要）

### 改善: 実装完了チェックリスト

詳細は[品質チェックリスト](../../.claude/agents/quality-checklist.md)を参照してください。

#### 必須条件

- [ ] すべてのpublicメソッドが実装済み（TODOコメントなし）
- [ ] すべてのテストがパス
- [ ] カバレッジ80%以上
- [ ] Lint/ビルドエラーなし

#### Lambda関数がある場合の追加条件

- [ ] Handlers実装済み（Orchestrator, Worker等）
- [ ] Lambda固有設定完了（timeout, memory, environment）
- [ ] デプロイ可能なzipファイル生成可能

#### 外部API統合がある場合の追加条件

- [ ] 実装コード完成（**モックだけでなく実コードも**）
- [ ] エラーハンドリング実装
- [ ] リトライロジック実装（該当する場合）

---

## インフラ設計のベストプラクティス

### 環境別デフォルト設定

**課題**: X-Rayを全環境で有効にしていた → POC環境でコスト増

**改善**: 環境ごとにデフォルト設定を変える

#### POC/開発環境（dev）

- X-Ray: **無効** （コスト削減）
- Point-in-Time Recovery: **無効** （DynamoDB）
- Multi-AZ: **無効** （RDS）
- バックアップ: **無効**
- インスタンスサイズ: **最小** （db.t3.micro、lambda 128MB等）

#### ステージング環境（staging）

- X-Ray: **無効** （コスト削減）
- Point-in-Time Recovery: **有効**
- Multi-AZ: **無効**
- バックアップ: **3日**
- インスタンスサイズ: **小** （db.t3.small、lambda 256MB等）

#### 本番環境（production）

- X-Ray: **有効** （パフォーマンス監視）
- Point-in-Time Recovery: **有効**
- Multi-AZ: **有効** （高可用性）
- バックアップ: **7日**
- インスタンスサイズ: **要件に応じて** （NFRから決定）

### ドキュメント配置

**課題**: 設計ドキュメントを各パッケージに分散配置 → unit間の依存関係が見えにくい

**改善**: ルートの `docs/` に集約

```
project-root/
├── docs/                        # ← ルートに集約
│   ├── intents/                 # Intent階層化
│   │   └── {Intent番号}_{Intent名}/
│   │       ├── intent.md
│   │       ├── units.md
│   │       └── {Unit番号}_{Unit名}/
│   │           ├── domain.md
│   │           ├── architecture.md  # IaC設計もここ
│   │           ├── tests.md
│   │           └── plan.md
│   └── adr/
└── packages/
    └── infrastructure/
        └── terraform/
```

**理由**:
- 横断的な参照が容易
- unit間の依存関係を把握しやすい
- 全体像を俯瞰できる

---

## トラブルシューティング

### ユニット分解の粒度ミス

**問題**: Lambda Handlersが実装スコープに含まれていなかった

**対処**:
1. `/units` 実行時に「実装スコープ」を明示
2. `/bolt` 開始前に実装スコープを再確認
3. Lambda関数がある場合、Handlers実装を必ずPhaseに含める

### テスト命名の混乱

**問題**: LocalStackを使ったテストを「E2E」と呼んでいた

**対処**:
1. LocalStack = Integration Test
2. 実AWS API = E2E Test
3. テストファイル名で区別: `*.test.ts` (Unit/Integration), `*.e2e.test.ts` (E2E)

### コミットタイミング不明

**問題**: いつコミットすべきか不明確

**対処**:
1. `/bolt` 完了後に必ずコミット推奨メッセージ
2. unit単位を基本とする
3. 大きなunitの場合はphase単位も可

---

**更新履歴**:
- 2025-11-19: 初版作成（実プロジェクトのフィードバックを反映）
