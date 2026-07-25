# CLAUDE.md

このファイルは、レガシーWebシステム → React/Redux移行プロジェクトにおける、AIエージェント（Claude Code）
への恒久指示です。**このファイルは「現新橋（strangler-kit）」を新しい移行プロジェクトに導入する際、
プロジェクトルートに `CLAUDE.md` としてコピーし、そのプロジェクト固有の内容（技術スタック・進捗・
更新履歴）を書き足しながら育てていくための雛形です。** プロジェクトの進行に合わせて随時更新してください
（`kickoff-migration`実行時点で「Phase 1」等、現在のフェーズをここに書き足していく想定）。

## 1. プロジェクト概要

- 中規模のレガシーWebシステム（目安: 数千〜数万行、画面数10〜20程度）を、React/Redux + Spring Boot構成へ、
  設計書ファースト・AIエージェント主体で移行するためのプロジェクトです。
- 実装・設計・検証の大部分をAIエージェントに担わせる前提でプロジェクトを構成しています。
- 全体計画は `docs/00_migration_plan.md`（`kickoff-migration`実行前にプロジェクト固有の内容で作成すること）
  を必ず参照してください。本ファイルとの内容の重複がある場合、詳細・背景は計画書側が正、
  「今この瞬間どう動くべきか」の実務指示は本ファイルが正とします。

## 2. 技術スタック

| レイヤー | 現行 | 移行後 |
|---|---|---|
| フロントエンド | （現状分析で確定・記入） | TypeScript（ES6+）+ React + Redux（Redux Toolkit） |
| バックエンド | （現状分析で確定・記入） | Spring Boot（REST API） |
| DB | （現状分析で確定・記入） | PostgreSQL |

`scaffold-foundation`実行時、`docs/02_foundation_design.md`（`docs/templates/foundation_design_template.md`
に既定値を記載済み）で以下を確定してください。プロジェクト固有の事情（ビルド環境にGradleしか無い等）が
なければ、この既定値をそのまま採用してよいものとして扱います。

**バックエンド既定値**

| 項目 | 既定値 |
|---|---|
| JDK | Java 21（LTS） |
| ビルドツール | Maven |
| フレームワーク | Spring Boot 3.x 最新安定版 |
| DBマイグレーション | Flyway |
| 認証 | Spring Security + JWT（レスポンスボディでトークン返却、`localStorage`等への永続化はしない） |
| バリデーション | Bean Validation（`jakarta.validation`） |
| エラーハンドリング | `@RestControllerAdvice`、統一形式 `{code, message, details}` |
| テスト | JUnit 5 + Mockito + AssertJ + `spring-boot-starter-test`（`@WebMvcTest`でController層をスライステスト） |
| ローカルDB | Docker Compose（`postgres:16-alpine`） |

**フロントエンド既定値**

| 項目 | 既定値 |
|---|---|
| ビルドツール | Vite |
| フレームワーク | React 18系（安定重視）。19系採用も妨げない |
| 状態管理 | Redux Toolkit + RTK Query |
| ルーティング | React Router v6（data router） |
| フォーム | 追加ライブラリなし。`useState`による手動バリデーション＋共通`FormErrorAlert`/`FlashMessageBanner`コンポーネント |
| テスト | Vitest + React Testing Library |
| E2E | Playwright（導入は任意） |
| Lint/Format | ESLint + typescript-eslint + Prettier、または`npm create vite`既定のoxlintでも可（どちらでもよい） |

確定した内容は本セクションにも要約を追記してください。

## 3. ディレクトリ構成

```
<project-root>/
├── CLAUDE.md                    ← 本ファイル（プロジェクトごとにこの雛形からコピーして育てる）
├── docs/
│   ├── 00_migration_plan.md     ← 全体計画書（プロジェクト開始時に作成）
│   ├── 01_legacy_architecture_overview.md  ← 現行アーキテクチャ概要（kickoff-migrationが作成）
│   ├── 02_foundation_design.md  ← 共通基盤設計書（scaffold-foundationが作成、Wave1着手前に必須）
│   ├── screens_inventory.md     ← 画面一覧・進捗管理表
│   ├── templates/               ← 設計書テンプレート一式（本ツールの一部）
│   └── screens/                 ← 画面ごとの設計書実体（<SCR-ID>_<画面名>/ 配下）
├── legacy/                      ← 現行レガシーシステムのソース一式（配置待ち）
├── backend/                     ← 新Spring Bootバックエンド
├── frontend/                    ← 新React/Reduxフロントエンド
└── .claude/
    ├── agents/                  ← 移行用サブエージェント定義（本ツールの一部）
    └── skills/                  ← 移行ワークフロー（SKILL、本ツールの一部）
```

## 4. 進め方の大原則

1. **設計書ファースト**: 実装前に必ず設計書（現状分析・画面設計・API設計）を作成し、人間の承認を得てから実装に着手する。
2. **画面単位のストラングラー移行**: 1画面を1移行単位とし、`docs/00_migration_plan.md` §5 の10ステップサイクルで進める。
3. **現新一致の原則**: 見た目（UI）は刷新してよいが、入力に対する業務結果（計算・登録内容・表示データ）は現行と一致させる。
   **例外**: 認可チェックの欠如、平文パスワード、ハードコードされたバックドア等のセキュリティ上の欠陥は対象外とする
   （現状分析書「11. セキュリティ上の懸念点」に記載し、修正すること自体を移行の目的とする。詳細は
   `docs/00_migration_plan.md` §3.3）。
4. **不明点は推測しない**: レガシーコードの意図が読み取れない、仕様が曖昧、DB/API定義に矛盾がある等の場合は、
   設計書に「未確定事項」として明記し、実装を進めず人間に確認する。仕様を勝手に決めて実装しない。
5. **人間承認ゲート**: 設計レビュー完了時、リリース判定時の2箇所は必ず人間の承認を経る。AIエージェントはここを自己判断で通過しない。
6. **テンプレート厳守**: 設計書は `docs/templates/` のテンプレートに従って作成する。独自フォーマットで作らない。
7. **ドキュメントの一貫性維持**: 後続工程で前工程の記載が誤り・古いと判明した場合（画面名の誤認識、
   サンプルコードのフィールド名が確定仕様と食い違う、既にリリース済みの他画面に手を入れたが設計書側の
   更新が漏れている等）、気づいたエージェントが上流ドキュメントの該当箇所（見出しを含む）を修正する。
   全文修正が難しい場合は、その場に訂正の注記を残す。「後で誰かが直すだろう」で放置しない。

## 5. サブエージェント一覧（`.claude/agents/`）

| エージェント名 | 役割 | 入力 | 出力 |
|---|---|---|---|
| `legacy-analyzer` | レガシー画面のソースを読み解き現状分析書を作成 | `legacy/` 配下の該当画面ソース | `01_legacy_analysis.md` |
| `screen-designer` | 現状分析を基に新画面（React/Redux）を設計 | `01_legacy_analysis.md` | `02_screen_design.md` |
| `api-designer` | 新API・DB差分を設計 | `01_legacy_analysis.md`, `02_screen_design.md` | `03_api_design.md`（必要に応じ `04_db_design.md`） |
| `backend-coder` | Spring Boot側の実装 | `03_api_design.md`, `04_db_design.md` | `backend/` 配下のコード＋単体テスト |
| `frontend-coder` | React/Redux側の実装 | `02_screen_design.md`, `03_api_design.md` | `frontend/` 配下のコード＋単体テスト |
| `test-engineer` | テスト仕様作成・現新比較検証 | 上記設計書一式＋実装 | `05_test_spec.md`、結合/E2Eテスト |
| `migration-reviewer` | 実装と設計書の整合性・コード品質レビュー | 実装一式・設計書一式 | レビュー結果、`06_checklist.md` 更新 |

エージェントは `docs/screens/<SCR-ID>_<画面名>/` に成果物を作成・更新することを前提に動く。

## 6. SKILL一覧（`.claude/skills/`）

| SKILL | 用途 | 実行タイミング |
|---|---|---|
| `kickoff-migration` | レガシー全体を棚卸し、画面一覧・優先順位・現行アーキテクチャ概要を作成 | Phase 1の開始時（`legacy/` にソース配置後）に1回 |
| `scaffold-foundation` | 認証方式・状態管理方針等を共通基盤設計書として確定し、backend/frontendの雛形を作成 | Phase 1.5（`kickoff-migration`完了後、Wave1着手前）に1回 |
| `migrate-screen` | 1画面分の移行を10ステップサイクルで一気通貫に実行 | 画面ごとにPhase 2で実行（`scaffold-foundation`完了後） |
| `verify-migration` | 特定画面の現新比較検証を単独で再実行 | 実装修正後の再検証、リリース判定前の最終確認など |

`migrate-screen` は `docs/02_foundation_design.md` が存在しない場合、`scaffold-foundation` が未実施と判断して
先にそちらの実行を確認する（認証方式・状態管理方針を画面ごとに個別判断させないため）。

## 7. 設計書格納規約

各画面は `docs/screens/<SCR-ID>_<画面名>/` にフォルダを作成し、以下のファイル名で統一する。

```
docs/screens/SCR-001_ログイン/
├── 01_legacy_analysis.md
├── 02_screen_design.md
├── 03_api_design.md
├── 04_db_design.md      (差分がある場合のみ)
├── 05_test_spec.md
└── 06_checklist.md
```

`SCR-ID` は `docs/screens_inventory.md` で採番・管理する。

## 8. コーディング規約

- フロントエンド: TypeScript strict mode、Redux Toolkit + RTK Queryを標準採用（§2参照）。クラスコンポーネントは
  使わず関数コンポーネント＋Hooksに統一。フォームは新規ライブラリを導入せず`useState`による手動バリデーション
  ＋共通`FormErrorAlert`/`FlashMessageBanner`（Wave2着手時に確立、詳細は`docs/02_foundation_design.md`）。
- バックエンド: Spring Bootの標準レイヤードアーキテクチャ（Controller / Service / Repository / Entity・DTO）を
  機能単位パッケージング（`<basePackage>.<screen機能ドメイン>.*`）で踏襲。共通例外は`@RestControllerAdvice`に
  集約し、統一エラー形式`{code, message, details}`に従う。
- 命名・Lint/Formatterルールの詳細、および§2の既定値からの変更点はPhase 1（`kickoff-migration`/
  `scaffold-foundation`）確定後に追記する。

## 9. AIエージェントが守るべき禁止事項

- 設計書のレビュー・承認前に実装を進めないこと。
- 現状分析で読み取れない業務ロジックを推測で実装しないこと（必ず未確定事項として明記）。
- テンプレートにない独自形式で設計書を作らないこと。
- 人間承認ゲート（設計レビュー・リリース判定）を自己判断でスキップしないこと。
- レガシーコードを直接改変しないこと（`legacy/` は参照専用）。

## 10. 更新履歴

- YYYY-MM-DD: 初版作成。
