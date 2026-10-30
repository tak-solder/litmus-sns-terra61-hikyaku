# Build 17: いいねAPI

## やること

- `likes` テーブルとマイグレーション（`(post_id, user_id)` 主キー、`user_id` のインデックス）
- `PUT /api/posts/{id}/like`: いいねする。重複はできない（冪等）。自分の投稿は 400。閲覧権限の無い投稿は 404（build-11 の判定）。ゲストは 401。`posts.like_count` を同一トランザクションで増やす
- `DELETE /api/posts/{id}/like`: 取り消し。`like_count` を同一トランザクションで減らす
- `GET /api/posts/{id}` の `is_liked_by_me` を実装する（ゲストは常に false）

## やらないこと

- 一覧 API でのいいね情報（build-18 以降。ただし共通の取得関数は本ビルドで用意する）
- Web のいいねボタン（build-23）

## 対応する受け入れ基準

- US-7.1, US-7.2, US-7.3, US-7.4, US-7.5

## 受け入れ基準

- [ ] 他ユーザーの投稿にいいねでき、合計いいね数が1増え、`is_liked_by_me` が true になる
- [ ] 取り消すと合計が1減り、false に戻る
- [ ] 同じ投稿への重複いいねで合計は増えない（同時リクエストでも1人1回。並行テストで確認する）
- [ ] 自分の投稿とゲストは拒否され、閲覧権限の無い投稿にはいいねできない

## 参照する設計ドキュメント

- design-delta.md「データモデル」`likes`、「閲覧権限の条件」（書き込み系の認可）

## BP見積もり

作成するファイル: `apps/api/src/db/schema/likes.ts`, `apps/api/drizzle/0006_likes.sql`, `apps/api/src/services/like.ts`, `apps/api/src/routes/likes.ts`, `packages/contract/src/like.ts`, `apps/api/src/services/like.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 6 | 3（ベース） |
| 実装行数（テスト含む） | 450 | 2 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 1 | 2 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
