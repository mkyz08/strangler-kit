---
name: scaffold-foundation
description: Phase 1.5（共通基盤構築）を実行する。認証方式・状態管理方針・共通APIクライアント等を共通基盤設計書として確定し、backend/frontendのプロジェクト雛形と共通実装を作成する。kickoff-migration完了後、Wave1のmigrate-screen着手前に1回だけ実行する。
---

# scaffold-foundation

`kickoff-migration` で判明した内容と `docs/00_migration_plan.md` §11 の未確定事項をもとに、全画面共通の
技術方針（認証方式・状態管理方針・共通APIクライアント等）を先に確定し、`backend/` `frontend/` の
プロジェクト雛形を作成する。**これをWave1の1画面目に場当たり的に決めさせない**ことが目的。

## 実行手順

1. `docs/00_migration_plan.md` §11、`docs/01_legacy_architecture_overview.md` を確認する。
   `docs/templates/foundation_design_template.md` には各項目の既定値（Maven、Java 21、Spring Boot
   3.x、Flyway、Vite、React 18系、Redux Toolkit + RTK Query、Tailwind CSS等）が記載済み。

2. **既定値を最初にまとめてユーザーに選択させる（重要）。** 既定値を無断で採用しない。かといって
   1項目ずつ逐一質問するのでもなく、**主要な技術選定を1つの表にまとめて一括提示**し、
   「すべて既定値でよいか、変更したい項目があるか」を確認する（例: 「ビルドツールはMavenを既定値と
   していますが、Gradleへの変更も可能です」のように、既定値と代替案を並べて示す）。特にCSS
   フレームワークは、既定値のTailwind CSSに加えて**「現行システムが使用しているCSSフレームワークを
   踏襲する」という選択肢も必ず提示する**（`docs/01_legacy_architecture_overview.md`に現行の
   CSSフレームワークの記載があれば参照する。無ければ`legacy/`を確認する）。
   `docs/00_migration_plan.md` §11の未確定事項（DB種別、実行環境の制約等、既定値では判断できないもの）も
   同じタイミングでまとめて確認する。

3. ユーザーの回答をもとに `docs/templates/foundation_design_template.md` に従い
   `docs/02_foundation_design.md` を作成する。既定値をそのまま採用した項目・変更した項目の両方について、
   理由（既定値を踏襲した／ユーザーが選択した、等）を簡潔に添える。

4. **共通基盤設計レビュー（人間承認ゲート）**: 作成した設計書をユーザーに提示し、承認を得るまで次に
   進まない。ステップ2で確認済みの内容も含め全体としての最終確認であり、ステップ2を省略してよい理由には
   ならない（ステップ2は「選ぶ」機会、ここは「選んだ結果が正しく文書化されているか」の確認）。
   以降のすべての画面設計・実装がここでの決定に従うため、他の承認ゲート以上に慎重に確認を求める。

5. 承認後、`backend-coder` を「共通基盤構築タスク」として呼び出し、`docs/02_foundation_design.md` に基づく
   バックエンドのプロジェクト雛形・認証実装・共通例外ハンドリング・DBマイグレーション基盤を作成させる。

6. 同様に `frontend-coder` を「共通基盤構築タスク」として呼び出し、フロントエンドのプロジェクト雛形・
   ルーティング・Redux store初期構成・共通APIクライアント・共通レイアウトに加えて、**Playwright E2E基盤**
   （`@playwright/test`導入、`playwright.config.ts`、`e2e/`ディレクトリ、`npm run test:e2e`スクリプト。
   `docs/02_foundation_design.md`のE2E既定値を参照）も作成させる（backend-coderと並列実行可）。

7. **実機ブラウザでの疎通確認（重要）**: バックエンド・フロントエンド双方を起動し、Playwright
   （`npx playwright install chromium` — システム全体へのインストール権限が無い環境でもダウンロード可能）で
   実際にログインできることを確認する。curl/単体テストだけでは検出できない不具合（CORS設定漏れ等、
   ブラウザの同一オリジンポリシーに起因するもの）がここで初めて顕在化することがある。問題があれば
   `backend-coder`/`frontend-coder`に差し戻して修正し、再度この手順で確認する。

8. 完了後、作成された雛形の概要（実機ブラウザ確認の結果を含む）をユーザーに報告し、Wave1の
   `migrate-screen` に進んでよいか確認する。

## 注意事項

- ここで決めた認証方式・状態管理方針（RTK Query採用可否等）は、以降 `screen-designer` `api-designer` が
  個別画面設計時に踏襲する前提になる。後から方針変更する場合は影響範囲（既に移行済みの画面）を洗い出すこと。
- 個別画面の業務ロジックはここでは一切扱わない。
- ステップ2（既定値の選択確認）を省略しない。「既定値がある＝確認不要」ではない。
- ステップ7を省略しない。curl・単体テストが全件成功していても、ブラウザからは接続できないという
  ケースが実際に発生している（CORS設定の丸ごと欠落）。
