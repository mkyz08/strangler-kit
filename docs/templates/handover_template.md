# 引き継ぎドキュメント

- 作成: SKILL `handover-migration`
- 作成日:
- 最終更新日:
- 対象読者: 開発・保守担当者（このドキュメントを最初に読むことでプロジェクト全体を把握できることを目指す）
- 参照元: `docs/00_migration_plan.md`, `docs/01_legacy_architecture_overview.md`,
  `docs/02_foundation_design.md`, `docs/screens_inventory.md`, `docs/screens/*/06_checklist.md`

> **本ドキュメントの位置づけ**: 各種設計書・チェックリストの要約であり、正本ではない。本文と参照元
> ドキュメントの内容に矛盾がある場合は参照元（特に`docs/02_foundation_design.md`、`docs/screens/`配下の
> 設計書）を正とする。参照元が更新された場合、本ドキュメントの該当箇所も更新すること（`CLAUDE.md` 原則7）。

## 0. 本ドキュメントのスコープと鮮度

- 作成時点の完了状況（全<N>画面中<M>画面完了。未完了画面がある場合はここに明記）
- 元にした主要ドキュメントの最終更新日（`docs/02_foundation_design.md`等）
- 更新方針: `handover-migration` SKILLの「再実行時の扱い」を参照

## 1. プロジェクト概要

- 対象システム、移行の目的、現行→移行後の技術スタック概要
- 移行完了状況のサマリ（画面数、Wave数、完了時期）

## 2. アーキテクチャ概要

- フロント/バック/DB構成図（Mermaid可）
- 技術スタック一覧（現行→移行後の対比表。`docs/02_foundation_design.md`の要約）
- 主要ディレクトリ構成（`backend/`・`frontend/`の要点）

## 3. ローカル環境構築・起動手順

- 前提ソフトウェア（JDK、Node.js、Docker等のバージョン）
- 起動手順（番号付き、実コマンド。例: `docker compose up -d` → `mvn spring-boot:run` → `npm run dev`）
- 動作確認URL・初期認証情報（該当する場合）
- テストの実行方法（単体テスト、E2Eテスト）

## 4. 主要設計判断のサマリ

- `docs/02_foundation_design.md`からの要約表（項目・決定・理由・詳細への参照）

| 項目 | 決定 | 理由 | 詳細 |
|---|---|---|---|
| | | | `docs/02_foundation_design.md` §? |

## 5. 画面一覧・移行状況

- `docs/screens_inventory.md`へのポインタ、Wave別サマリ
- 画面ごとの設計書の場所・見方（`docs/screens/<SCR-ID>_<画面名>/`の各ファイルの役割）

## 6. 既知の制約・未解決事項

### 6.1 未解決の指摘事項（人間の追加確認・対応が望ましいもの）

| SCR-ID | 指摘内容の要約 | 詳細 |
|---|---|---|
| | | `docs/screens/<SCR-ID>_<画面名>/06_checklist.md` |

### 6.2 意図的に許容した制約・スコープ外事項（対応しないと確定済みのもの）

| 項目 | 内容 | 根拠 |
|---|---|---|
| | | |

## 7. 修正・追加開発時の参照先

- 恒久指示: `CLAUDE.md`
- 画面ごとの詳細設計・実装根拠: `docs/screens/<SCR-ID>_<画面名>/`
- 新規画面を追加する場合: SKILL `migrate-screen`
- 実装修正後の再検証: SKILL `verify-migration`
- 共通基盤方針の変更: `docs/02_foundation_design.md`を更新し影響範囲を洗い出す

## 8. 変更履歴

| 日付 | 内容 |
|---|---|
| | 初版作成 |
