# Build 10: 投稿の作成・削除API（本文のみ）

## やること

- `posts` テーブルとマイグレーション（id は UUIDv7、author_id、body、search_text、image_id（null 可）、like_count、deleted_at。インデックスは `(author_id, id desc)` と `id desc`）。検索用の GIN インデックスは build-21 で追加する
- `POST /api/posts`: 本文（300文字以内、空白・改行のみは本文なし）を検証して作成する。画像は本ビルドでは受け付けない（本文が無ければ拒否）。`search_text` は NFKC 正規化 + 小文字化して保存する。ゲストは 401。理由つきの 400
- `DELETE /api/posts/{id}`: 投稿者本人のみ。論理削除（`deleted_at`）。本人以外・存在しない投稿は 404
- 書き込みへのレート制限の適用

## やらないこと

- 画像の添付（build-15、16）
- 投稿の閲覧・単体取得と閲覧権限の判定（build-11）
- 各一覧への表示（build-18 以降）

## 対応する受け入れ基準

- US-4.1, US-4.2, US-4.4, US-5.1

## 受け入れ基準

- [ ] 300文字以内の本文で投稿でき、本文が空白のみ・301文字以上のときは理由つきで拒否される
- [ ] ゲストは投稿できない
- [ ] 自分の投稿を削除でき、他人の投稿は削除できない
- [ ] `search_text` に NFKC 正規化 + 小文字化した本文が保存される

## 参照する設計ドキュメント

- design-delta.md「データモデル」`posts`、「投稿・検索・画像」
- ADR: 20261030-search-pg-bigm.md

## BP見積もり

作成するファイル: `apps/api/src/db/schema/posts.ts`, `apps/api/drizzle/0004_posts.sql`, `apps/api/src/lib/normalize-search-text.ts`, `apps/api/src/services/post.ts`, `apps/api/src/routes/posts.ts`, `packages/contract/src/post.ts`, `apps/api/src/lib/normalize-search-text.test.ts`, `apps/api/src/services/post.test.ts`, `apps/api/src/routes/posts.test.ts`, `apps/api/src/db/schema/posts.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 10 | 3（ベース） |
| 実装行数（テスト含む） | 750 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 1 | 2 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
