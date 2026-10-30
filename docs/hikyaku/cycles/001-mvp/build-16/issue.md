# Build 16: 画像の保存・アップロード・認可配信API

## やること

- `images` テーブルとマイグレーション（ランダム ID、post_id、S3 キー、content_type、size）
- S3 クライアント（ローカルは MinIO）。バケットは全面的にパブリックアクセスをブロックする前提
- `POST /api/posts` を multipart に拡張して画像を1枚添付できるようにする: build-15 のライブラリで検証・メタデータ除去し、S3 に保存して `images` と投稿を関連付ける。本文が空でも画像があれば投稿できる。2枚以上、4形式以外、5MB 超は理由つきで拒否する（投稿の作成・削除は build-10）
- `GET /api/images/{imageId}`: リクエストごとに認可する（投稿が存在するか、投稿者が公開か、閲覧者が本人またはフォロワーか。build-11 の共通判定を使い、プライマリ DB を読む）。権限が無い場合も存在しない場合も 404。S3 から取得して返し、`Cache-Control: private, no-cache`、ETag（画像 ID 由来）、`Content-Type`（検出した型）、`X-Content-Type-Options: nosniff` を付ける。`If-None-Match` で 304 を返す。署名 URL は発行しない
- この GET に限り、同一オリジンのセッション cookie を受け付ける（`<img>` は Authorization ヘッダを付けられないため）
- `GET /api/posts/{id}` の応答に、画像の URL（`/api/images/{imageId}`）を含める
- 結合テスト: 公開範囲の変更、フォロー解除、投稿の削除の直後に、画像が取得できなくなることを確認する

## やらないこと

- 一覧 API の応答に画像を含める（build-18、19、21）
- 画像の Web 表示（build-22、23）
- CDN や署名 URL による配信

## 対応する受け入れ基準

- US-4.5, US-4.6, US-4.7

## 受け入れ基準

- [ ] 画像を1枚添えて投稿でき、本文が空でもよい。2枚以上・4形式以外・5MB 超は理由つきで拒否される
- [ ] 保存された画像からメタデータが除去されている
- [ ] 画像は権限のある閲覧者だけが取得でき、権限が無い人・ゲスト・存在しない ID は 404 になる
- [ ] 公開範囲の変更・フォロー解除・投稿の削除の直後から、画像が取得できなくなる（結合テスト）
- [ ] 単体取得の応答に画像 URL が含まれ、cookie 認証でも画像 GET が通る（Bearer でも通る）

## 参照する設計ドキュメント

- design-delta.md「投稿・検索・画像」「性能・費用の検証計画」
- ADR: 20261030-image-per-request-authz.md、20261030-session-token-auth.md

## BP見積もり

作成するファイル: `apps/api/src/db/schema/images.ts`, `apps/api/drizzle/0005_images.sql`, `apps/api/src/storage/s3.ts`, `apps/api/src/services/post-image.ts`, `apps/api/src/routes/images.ts`, `apps/api/src/middleware/cookie-auth.ts`, `packages/contract/src/image.ts`, `apps/api/src/services/post-image.test.ts`, `apps/api/src/routes/images.test.ts`, `apps/api/src/routes/posts-image.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 10 | 3（ベース） |
| 実装行数（テスト含む） | 700 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 1 | 2 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 6 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | あり | +1 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
