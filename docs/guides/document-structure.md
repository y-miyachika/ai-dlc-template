# AI-DLCドキュメント構造

## ルートdocs/ - AI-DLCフレームワーク共通（テンプレートに含まれる）

```
docs/
├── guides/              # AI-DLC開発ガイド
│   ├── getting-started.md
│   └── workflow.md
├── AI-DLC_日本語訳.md   # AI-DLC論文日本語訳
└── AI-DLC準拠状況.md    # 準拠率分析・実装状況
```

## プロジェクトのdocs/ - `/setup-aidlc`実行後に作成される

プロジェクトタイプ（app/monorepo）に応じて、以下の構造が作成されます：

```
docs/
├── intents/                          # Intent階層（成果物集約）
│   └── {Intent番号}_{Intent名}/
│       ├── intent.md                 # インテント定義
│       ├── units.md                  # ユニット分解
│       ├── 000_shared/               # 共通（オプション）
│       └── {Unit番号}_{Unit名}/
│           ├── domain.md             # ドメイン設計
│           ├── architecture.md       # アーキテクチャ設計
│           ├── tests.md              # テスト設計
│           └── plan.md              # 実装計画
└── adr/                              # アーキテクチャ決定記録（横断的）
```

**理由**: Intent→Unit単位で成果物を集約し、1ユニットの全成果物を1ディレクトリで参照可能

## 命名規則

**Intent階層構造を使用**（例: `docs/intents/002_ユーザー認証/001_login-api/domain.md`）

```
docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/
```

| 種類 | パス | 例 |
|------|------|-----|
| Intent定義 | `docs/intents/{Intent番号}_{Intent名}/intent.md` | `docs/intents/002_ユーザー認証/intent.md` |
| ユニット分解 | `docs/intents/{Intent番号}_{Intent名}/units.md` | `docs/intents/002_ユーザー認証/units.md` |
| ドメイン設計 | `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/domain.md` | `docs/intents/002_ユーザー認証/001_login-api/domain.md` |
| アーキテクチャ設計 | `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/architecture.md` | 同上パターン |
| テスト設計 | `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/tests.md` | 同上パターン |
| 実装計画 | `docs/intents/{Intent番号}_{Intent名}/{Unit番号}_{Unit名}/plan.md` | 同上パターン |
| ADR | `docs/adr/ADR-{連番}_{タイトル}.md` | `docs/adr/ADR-001_技術選定.md` |

- Intent番号・Unit番号は3桁ゼロ埋め
- Unit番号`000`はshared/共通拡張用（例: `002_ユーザー認証/000_shared/`）
- ファイル名は固定: intent.md, units.md, domain.md, architecture.md, tests.md, plan.md
