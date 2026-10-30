# Build 07: Web基盤と登録・ログイン画面

## やること

- Web の基盤: レイアウト、日本語 UI、日時の日本時間表示、PC とスマートフォンの画面幅に対応するスタイルの土台
- API クライアント（`packages/contract` を使った型付き）と、Server Components / Server Actions から api を呼ぶ仕組み。httpOnly・Secure・SameSite=Lax の cookie にセッショントークンを持ち、Bearer に変えて api を呼ぶ
- 登録画面とログイン画面（エラーの理由表示、確認前のログインで「確認が必要」の表示と確認メールの再送導線）。ログアウト操作
- ログイン状態を取得する共通部品（ゲストとログインユーザーでの出し分け）

## やらないこと

- メール確認結果・パスワードリセットの画面（build-08）
- トップページ・フィード（build-22）
- 画像配信用の cookie 受け入れ（build-16）。ブラウザは api を直接呼ばない

## 対応する受け入れ基準

- US-1.1, US-1.3, US-1.6, US-1.8, US-2.1, US-2.2, US-2.3

## 受け入れ基準

- [ ] 登録画面から登録でき、未入力・形式不正の理由が表示される
- [ ] 確認前のログインで確認が必要な旨が表示され、そこから確認メールを再送できる
- [ ] ログインに成功するとセッション cookie が設定され、ログアウトでゲストに戻る
- [ ] ブラウザから api への直接のリクエストが無い（Server Actions / Server Components 経由）
- [ ] スマートフォン幅と PC 幅で崩れない

## 参照する設計ドキュメント

- design-delta.md「全体構成」「認証・メール」
- ADR: 20261030-session-token-auth.md、20261030-rest-openapi-shared-contract.md
- `apps/web/AGENTS.md`（Next.js 16 の該当ガイドを読んでから実装する）、codebase-survey.md の `cacheComponents`

## BP見積もり

作成するファイル: `apps/web/app/layout.tsx` の更新を除く新規: `apps/web/lib/api-client.ts`, `apps/web/lib/session.ts`, `apps/web/lib/format-date.ts`, `apps/web/app/(auth)/register/page.tsx`, `apps/web/app/(auth)/login/page.tsx`, `apps/web/app/(auth)/actions.ts`, `apps/web/components/forms/*`（3ファイル）, `apps/web/components/header.tsx`, `apps/web/app/styles/*.css`, テスト 3ファイル

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 14 | 3（ベース） |
| 実装行数（テスト含む） | 900 | 3 |
| API操作数 | 0 | 1 |
| 画面/ページ数 | 2 | 3 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | あり | +1 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
