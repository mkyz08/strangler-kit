# 現新橋（strangler-kit）

レガシーWebシステムを **React + Redux（TypeScript）/ Spring Boot / PostgreSQL** 構成へ、
[Claude Code](https://claude.com/claude-code) のエージェント・SKILLを使ってAI主体で移行するための
ツールキットです。

名前の由来: プロジェクトを通じて核に据えている「**現新一致**」（現行システムと新システムの業務結果を
一致させる、というこのツールの検証原則）から。現行（現）と新（新）の間を橋渡しする、という意味を込めています。

## 特徴

- **設計書ファースト**: 現状分析 → 画面設計 → API設計 → （人間レビュー）→ 実装 → 検証 → レビュー →
  （人間承認）という10ステップサイクルを、画面（Strangler Fig パターンでの移行単位）ごとに回します。
- **人間承認ゲート**: 設計レビューとリリース判定の2箇所は必ず人間が承認します。AIが自己判断で先に進むことはありません。
- **現新一致の検証**: 業務結果（表示データ・登録内容・計算結果）が現行システムと一致するかを毎回検証します。
  ただし認可チェックの欠如や平文パスワードのようなセキュリティ上の欠陥は、忠実に再現すべき仕様ではなく
  修正対象として明示的に除外します。
- **不明点は推測しない**: レガシーコードから意図が読み取れない場合、AIは仕様を勝手に決めず「未確定事項」として
  記録し、人間に確認を求めます。
- **実機ブラウザでの検証を標準採用**: 単体テスト・curlでの結合確認だけでは、CORS設定漏れのようなブラウザの
  同一オリジンポリシーに起因する不具合を検出できません（検証中に実例あり）。Playwright E2Eテストを
  「任意」ではなく標準の検証手段として組み込んでいます。

## 構成

```
.
├── README.md
├── docs/
│   └── templates/          ← 設計書テンプレート一式（本ツールの中核）
│       ├── CLAUDE.template.md              新しいプロジェクトの CLAUDE.md の雛形
│       ├── foundation_design_template.md   共通基盤設計書テンプレート
│       ├── legacy_analysis_template.md     現状分析レポートテンプレート
│       ├── screen_design_template.md       画面設計書テンプレート
│       ├── api_design_template.md          API設計書テンプレート
│       ├── db_design_template.md           DB設計書（差分）テンプレート
│       ├── test_spec_template.md           テスト仕様書テンプレート
│       ├── migration_checklist_template.md 移行完了チェックリストテンプレート
│       └── handover_template.md            引き継ぎドキュメントテンプレート
└── .claude/
    ├── agents/             ← 移行用サブエージェント（legacy-analyzer, screen-designer, api-designer,
    │                           backend-coder, frontend-coder, test-engineer, migration-reviewer）
    └── skills/             ← 移行ワークフロー（kickoff-migration, scaffold-foundation, migrate-screen,
                                verify-migration, handover-migration）
```

エージェント・SKILLの役割の詳細は `docs/templates/CLAUDE.template.md` を参照してください。

## 使い方

1. このリポジトリの内容（`.claude/`, `docs/templates/`）を移行対象プロジェクトのルートにコピーする。
2. `docs/templates/CLAUDE.template.md` をプロジェクトルートに `CLAUDE.md` としてコピーし、
   プロジェクト概要・技術スタックを埋める。
3. 移行対象のレガシーソース一式を `legacy/` に配置する。
4. Claude Codeで `/kickoff-migration` を実行し、画面棚卸し・現行アーキテクチャ概要を作成する（Phase 1）。
5. `/scaffold-foundation` を実行し、認証方式・状態管理方針等の共通基盤を確定・実装する（Phase 1.5）。
6. 画面ごとに `/migrate-screen <SCR-ID>` を実行し、標準10ステップサイクルで移行する（Phase 2）。
7. 全Wave（全画面）のリリース判定が完了したら、`/handover-migration` を実行し、開発・保守担当者向けの
   引き継ぎドキュメント（`docs/03_handover.md`）を作成する。

途中で作成される計画書・設計書・進捗管理表（`docs/00_migration_plan.md` 等）や、移行対象プロジェクト自体の
ソースコード（`legacy/` `backend/` `frontend/`）は、このツールキット自体のリポジトリには含めていません。
プロジェクトごとに別リポジトリで管理してください。

## 想定対象システム

中規模のレガシーWebシステム（目安: 数千〜数万行、画面数10〜20程度）。特定の言語・フレームワークには
依存しません。サーバーサイドレンダリング型のWebシステムであれば、Controller/View/Service（または
Model）といった層構造を`legacy-analyzer`が読み解いて移行します。これまでJSP/Servlet、
Spring MVC + Thymeleaf構成（Java）での動作を確認しており、PHP等の他言語での検証も進めています。
