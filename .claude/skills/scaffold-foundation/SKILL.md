---
name: scaffold-foundation
description: Phase 1.5（共通基盤構築）を実行する。認証方式・状態管理方針・共通APIクライアント等を共通基盤設計書として確定し、backend/frontendのプロジェクト雛形と共通実装を作成する。kickoff-migration完了後、Wave1のmigrate-screen着手前に1回だけ実行する。
---

# scaffold-foundation

`kickoff-migration` で判明した内容と `docs/00_migration_plan.md` §11 の未確定事項をもとに、全画面共通の
技術方針（認証方式・状態管理方針・共通APIクライアント等）を先に確定し、`backend/` `frontend/` の
プロジェクト雛形を作成する。**これをWave1の1画面目に場当たり的に決めさせない**ことが目的。

## 実行手順

1. `docs/00_migration_plan.md` §11、`docs/01_legacy_architecture_overview.md` を確認する。認証方式・DB種別・
   ビルドツールなど、共通基盤設計に必須の未確定事項が残っている場合は、先にユーザーに確認する
   （このSKILL内で仮決めしない）。

2. `docs/templates/foundation_design_template.md` をもとに `docs/02_foundation_design.md` を作成する。
   決定事項だけでなく、その理由（既存のシステム制約、規模感から見た妥当性等）も簡潔に添える。

3. **共通基盤設計レビュー（人間承認ゲート）**: 内容をユーザーに提示し、承認を得るまで次に進まない。
   以降のすべての画面設計・実装がここでの決定に従うため、他の承認ゲート以上に慎重に確認を求める。

4. 承認後、`backend-coder` を「共通基盤構築タスク」として呼び出し、`docs/02_foundation_design.md` に基づく
   バックエンドのプロジェクト雛形・認証実装・共通例外ハンドリング・DBマイグレーション基盤を作成させる。

5. 同様に `frontend-coder` を「共通基盤構築タスク」として呼び出し、フロントエンドのプロジェクト雛形・
   ルーティング・Redux store初期構成・共通APIクライアント・共通レイアウトを作成させる（backend-coderと並列実行可）。

6. 完了後、作成された雛形の概要をユーザーに報告し、Wave1の `migrate-screen` に進んでよいか確認する。

## 注意事項

- ここで決めた認証方式・状態管理方針（RTK Query採用可否等）は、以降 `screen-designer` `api-designer` が
  個別画面設計時に踏襲する前提になる。後から方針変更する場合は影響範囲（既に移行済みの画面）を洗い出すこと。
- 個別画面の業務ロジックはここでは一切扱わない。
