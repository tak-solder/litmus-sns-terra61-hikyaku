# Build 19: ユーザーページAPI

## やること

- `GET /api/users/{id}`: 名前、公開範囲、閲覧者との関係（none / pending / accepted / self。build-09 の共通関数）、投稿の閲覧可否（`can_view_posts`）を返す。ゲストも閲覧できる。存在しないユーザーは 404
- `GET /api/users/{id}/posts`: そのユーザーの投稿のうち閲覧者が見られるものを、新しい順に1回最大20件、続きも取得できる形で返す（項目と取得の仕組みは build-18 の部品）。閲覧不可のときは一覧を返さず、`can_view_posts` が false であることで「投稿が非公開」と判別できる（投稿が0件の場合と区別できる）

## やらないこと

- Web のユーザーページ（build-24）
- 他ユーザーのフォロワー一覧・フォロー中一覧（スコープ外）

## 対応する受け入れ基準

- US-6.1, US-6.2, US-6.9

## 受け入れ基準

- [ ] ユーザーの名前と、閲覧者が見られる投稿の一覧が新しい順に最大20件ずつ返る（含める項目・続きの取得はフィードと同じ）
- [ ] 非公開ユーザーのページでも、名前と関係（フォロー・申請中）は返り、投稿を見られない閲覧者には `can_view_posts: false` が返る
- [ ] ゲストも閲覧でき、関係は none、`is_liked_by_me` は false

## 参照する設計ドキュメント

- design-delta.md「閲覧権限の条件」（ユーザーページ API）

## BP見積もり

作成するファイル: `apps/api/src/services/user-profile.ts`, `apps/api/src/services/user-posts.ts`, `apps/api/src/routes/users.ts`, `packages/contract/src/user-profile.ts`, `apps/api/src/services/user-profile.test.ts`, `apps/api/src/routes/users.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 6 | 3（ベース） |
| 実装行数（テスト含む） | 550 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
