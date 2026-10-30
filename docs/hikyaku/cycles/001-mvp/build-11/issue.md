# Build 11: 投稿単体取得と閲覧権限の共通判定

## やること

- 閲覧権限の共通条件を、api の一箇所に実装する。投稿を見られるのは、削除されておらず、かつ投稿者が公開、または閲覧者が投稿者本人か accepted のフォロワーである場合。一覧・検索・単体ページ・画像で再利用できるクエリ条件（SQL の述語）と判定関数として切り出す。閲覧権限の読み取りは常にプライマリ DB を使う
- `GET /api/posts/{id}`: 本文・画像（有無）・投稿日時・投稿者名・合計いいね数・`is_liked_by_me` を返す。ゲストの `is_liked_by_me` は false（いいね機能は build-17 で追加するまで固定で false）。権限が無い場合も、存在しない場合も 404 を返す（存在を漏らさない）
- 結合テスト: フォロー関係と公開範囲を直接 DB に作り、権限の組み合わせ（ゲスト・本人・フォロワー・申請中・フォローしていない人 × 公開・非公開・削除済み）を網羅して確認する

## やらないこと

- 公開範囲を切り替える API（build-12）。テストでは DB を直接更新して確認する
- 画像の配信（build-16）、いいね（build-17）、各一覧（build-18）

## 対応する受け入れ基準

- US-12.2, US-12.5, US-11.2, US-11.3, US-5.2

## 受け入れ基準

- [ ] 公開ユーザーの投稿は、ゲストを含む誰でも単体取得できる
- [ ] 非公開ユーザーの投稿は、本人と accepted のフォロワーだけが取得でき、ゲスト・申請中・フォローしていない人には 404 になる
- [ ] 公開範囲を DB 上で変えると、過去の投稿を含めて次のリクエストから取得可否が変わる
- [ ] 削除した投稿は取得できない（404）
- [ ] 権限の組み合わせを網羅する結合テストがある

## 参照する設計ドキュメント

- design-delta.md「閲覧権限の条件」
- ADR: 20261030-timeline-fan-out-on-read.md、20261030-no-cdn-cache-for-posts.md

## BP見積もり

作成するファイル: `apps/api/src/services/visibility.ts`, `apps/api/src/services/visibility.test.ts`, `apps/api/src/services/post-read.ts`, `apps/api/src/routes/post-read.ts`, `packages/contract/src/post-read.ts`, `apps/api/src/routes/post-read.test.ts`, `apps/api/src/test/fixtures/visibility-matrix.ts`, `apps/api/src/services/post-read.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 8 | 3（ベース） |
| 実装行数（テスト含む） | 650 | 3 |
| API操作数 | 1 | 2 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 6 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
