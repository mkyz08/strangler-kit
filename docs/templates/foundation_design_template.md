# 共通基盤設計書

- 作成: SKILL `scaffold-foundation`
- 作成日:
- 参照: `docs/00_migration_plan.md`（特に §11 未確定事項）, `docs/01_legacy_architecture_overview.md`

Wave 1（画面移行）着手前に一度だけ確定させる、全画面共通の技術方針を記載する。個別画面の設計書
（`02_screen_design.md` `03_api_design.md`）はここで決めた方針に従う。

各項目には既定値（推奨する標準的な選択肢）を添えている。ただし既定値は**無断で採用せず**、
`scaffold-foundation`の手順に従って着手前にユーザーへ選択肢として提示し、確認を得たうえで本書に記載する
（既定値のまま／ユーザーが選んだ代替案、いずれの場合も選定理由を書き添える）。

## 0. 表示言語

- 多言語対応（現行が英語/ドイツ語等の複数言語に対応している場合）を維持するか（既定値: 維持しない。
  検証・移行スコープの簡素化のため単一言語とする）:
- **単一言語とする場合、表示言語そのもの（日本語 or 英語）を必ずここで確定する。**
  「維持しない」を選んだだけで具体的な言語を確定しないまま進めない（実例: PetClinic検証で
  「日本語（または英語）」という曖昧な決定のままWave1に進んだ結果、画面ごとに担当エージェントが
  異なる言語を選び、ナビゲーション・見出し・ボタンが英語と日本語で混在する状態のままWave1が
  実装完了してしまい、リリース判定時に人間の指摘で判明・事後修正する事態になった）。エラーメッセージ
  （バックエンドの`{code, message, details}`の`message`部分含む）・画面見出し・ボタン・ラベル・
  テーブルヘッダー等、ユーザーに見えるテキスト全般が対象:

## 1. 認証・認可方式

- 現行方式（セッション方式）から変更するか、踏襲するか（既定値: トークン方式（JWT）へ変更する。
  SPA構成にはセッションCookieより適するため）:
- トークン方式を採用する場合: 保存場所（既定値: メモリ上のみ。Reduxの認証state等、`localStorage`/
  `sessionStorage`には永続化しない。XSSでのトークン窃取リスクを避けるため）、リフレッシュ方式
  （既定値: 本検証では未導入。必要であればリフレッシュトークン+httpOnly Cookieを別途検討）:
- 新旧並行稼働期間中のCookie/セッション競合対策（ドメイン分離、パス分離、リバースプロキシ振り分け等。
  プロジェクト固有の並行稼働方針次第のため既定値なし）:
- フロントエンド側の認可ガード方針（既定値: `RequireAuth`（未ログイン時は`/login`へリダイレクト）と
  `RequireRole`（許可ロール外は`/403`へリダイレクト）の2つの共通コンポーネントで実装する）:

## 2. バックエンド共通方針

- ビルドツール・バージョン（既定値: Maven、Java 21（LTS）、Spring Boot 3.x最新安定版）:
- パッケージ構成（既定値: 機能単位パッケージング。`<basePackage>.<screen機能ドメイン>.{controller,service,
  repository,entity,dto}`。レイヤー単位の巨大パッケージにしない）:
- 共通例外ハンドリング方式とエラーレスポンス形式（既定値: `@RestControllerAdvice`に集約。
  レスポンス形式 `{ "code": "string", "message": "string", "details": [] }`）:
- DBマイグレーションツール（既定値: Flyway。シンプルな増分マイグレーションに向き学習コストが低いため）:
- 共通バリデーション方針（既定値: Bean Validation（`jakarta.validation`）をDTOに付与）:
- CORS設定（既定値: フロントエンドの開発用オリジン（例: `http://localhost:5173`）を`CorsConfigurationSource`で
  明示的に許可する。**設定を省略すると、単体テスト・curlはすべて通過するのに実際のブラウザからは接続できない
  という不具合になる**（過去に実例あり。curlはブラウザの同一オリジンポリシーを経由しないため検出できない）:
- テスト基盤（既定値: JUnit 5 + Mockito + AssertJ + `spring-boot-starter-test`。Controller層は
  `@WebMvcTest`によるスライステストを基本とする。**加えて、`@SpringBootTest`によるアプリケーション全体の
  コンテキストロードテストを1件用意する**（H2インメモリDB、`spring.flyway.enabled: false`でDocker不要に
  する。`@WebMvcTest`はBeanをモック化するため、複数画面が並行して別パッケージに同名クラスを作ってしまう
  Spring Bean名衝突等のDI配線ミスを一切検出できない。この種の不具合は実際にアプリを起動して初めて発覚する
  ため、過去に実例あり）:
- 日時・数値型で表現する日時カラムの格納規約（現行が日時を独自形式の数値（Unixエポック秒等）で保持しており、
  新スキーマでもDATE/TIMESTAMP型へ正規化せずBIGINT等の数値型のまま踏襲する場合、**単位（エポック秒かミリ秒か）を
  ここに必ず明記する**。既定値: エポック**ミリ秒**（JavaScriptの`Date`・`Date.now()`との親和性を優先し、
  現行が秒単位でも変換する）。理由: この単位は列の型（`BIGINT`）だけからは判別できず、後続の画面実装で
  各担当エージェントが「現行踏襲だろう」と秒単位を独自に仮定してしまうと、シードデータや既存カラムとの
  単位不一致（一部は秒・一部はミリ秒）が生じ、日時が異常な未来日として表示される不具合になる
  （見た目上はAPIが200を返しJSONも正しく見えるため気づきにくい。過去に実例あり）。共通基盤構築時に
  シードデータ・エンティティの変換ロジック（`Instant.ofEpochSecond`か`Instant.ofEpochMilli`か）を統一し、
  その単位を本項目に明記することで、個別画面の`backend-coder`が独自に単位を仮定するのを防ぐ:

## 3. フロントエンド共通方針

- ビルドツール・React/Redux Toolkitバージョン（既定値: Vite、React 18系（安定重視。19系採用も妨げない）、
  Redux Toolkit最新版）:
- ディレクトリ構成（既定値: 機能単位。`src/features/<screen機能ドメイン>/` ＋ 共通部分
  （`src/app/`, `src/components/common/`））:
- ルーティング方針（既定値: React Router v6、data router方式）:
- 状態管理方針（既定値: RTK Queryを標準採用。`createSlice`は認証状態等UIローカルな状態にのみ使用）:
- 共通APIクライアント設計（既定値: RTK Queryの`baseQuery`。ベースURL・`Authorization`ヘッダ付与・
  401時の自動ログアウトを共通化）:
- 共通レイアウト・共通コンポーネント（既定値: `AppLayout`（ヘッダー・ナビゲーション、ロール別出し分け）、
  `RequireAuth`、`RequireRole`）:
- CSSフレームワーク・スタイリング方針（既定値: Tailwind CSS。ユーティリティクラスでスタイリングし、
  antd/MUI等の独自コンポーネントライブラリは導入しない。既存の手組みDialog/Table/フォーム実装方針と
  親和性が高いため。**代替案**: 現行システムが既にCSSフレームワーク（Bootstrap等）を使用している場合、
  それを踏襲する選択も可。現行踏襲を選ぶ場合、`docs/01_legacy_architecture_overview.md`に記載の
  現行フレームワークをReact向けの形（対応するnpmパッケージ等）で導入する。どちらを選んだか・理由を
  必ず明記する）:
- フォーム実装方針（既定値: 追加ライブラリ（react-hook-form等）は導入しない。`useState`による制御
  コンポーネント＋送信時の手動バリデーション。エラーは共通`FormErrorAlert`コンポーネントでまとめて表示
  （フィールド単位のインライン表示は行わない）。登録・更新成功時の通知は`useFlashMessage`フック＋
  `FlashMessageBanner`で統一する）:
- テスト基盤（既定値: Vitest + React Testing Library）:
- E2Eテスト（既定値: **Playwrightを標準採用する（任意にしない）**。curl/モックテストはブラウザの
  CORS制約を受けないため、CORS設定漏れ等ブラウザでしか顕在化しない不具合を検出できない。
  本基盤構築時に`frontend/playwright.config.ts`・`frontend/e2e/`を用意し、Chromium
  （`npx playwright install chromium`。システム全体へのインストール権限が無い環境でもダウンロード可能）
  で認証フローの疎通を確認してから画面移行に進む）:
- Lint/Format（既定値: ESLint + typescript-eslint + Prettier、または`npm create vite`既定の
  oxlintでも可。どちらでもよい）:

## 4. CI/CD・インフラ

- CI/CDツール（プロジェクトのホスティング・組織標準次第のため既定値なし）:
- デプロイ先・環境（開発/検証/本番）:
- ローカル開発用DB（既定値: Docker Compose、`postgres:16-alpine`。ローカルにPostgreSQLサーバーが
  無くても即座に開発着手できるようにする）:
- 新旧並行稼働時のルーティング方式（画面単位でのカットオーバーをどう実現するか）:

## 5. 未確定事項

- [ ]
