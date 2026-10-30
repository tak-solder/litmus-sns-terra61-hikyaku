# Build 21: 検索API（pg_bigm・負荷保護）

## やること

- マイグレーション: pg_bigm 拡張の有効化と、`posts.search_text` の GIN インデックス（`gin_bigm_ops`）
- `GET /api/search?q=...` と `POST /api/search`（本文にクエリ。転送上の長さの限界を超える検索のため）: build-20 の構文木と LIKE 条件に、build-11 の閲覧権限と `deleted_at is null` を組み合わせ、新しい順に1回最大20件・続きも取得できる形で返す（項目は build-18 の部品、ゲストの `is_liked_by_me` は false）。条件が成り立たない入力は 400 と理由を返す
- 負荷保護（入力は拒否しない）: 検索専用の小さな DB 接続プール、`statement_timeout`（p95 3秒の目標に合わせた値。超過で 503 と「混み合っています」）、利用者・IP 単位のレート制限（`rate_limits`）
- 既存の投稿に対する `search_text` の欠落が無いことの確認（build-10 以降の投稿は保存時に生成済み。新規構築のため移行は無い）
- 実行計画の確認（EXPLAIN）と、数万件規模での簡易な応答時間の確認

## やらないこと

- Web の検索画面（build-27）
- 語数・文字数への任意の上限（設けない）
- 本番規模での負荷試験（build-32、33）
- OpenSearch など別の検索基盤

## 対応する受け入れ基準

- US-10.1, US-10.2, US-10.3, US-10.4, US-10.5, US-10.6, US-10.7, US-10.8, US-10.9, US-10.10

## 受け入れ基準

- [ ] キーワードを含み閲覧者が見られる投稿が、新しい順に最大20件ずつ返り、続きを取得できる。各項目に本文・画像 URL・投稿日時・投稿者名・合計いいね数・`is_liked_by_me` が含まれる
- [ ] 部分一致で、英字の大小・英数字とカタカナの全角半角を区別しない。AND / 除外 / OR / フレーズが動く
- [ ] 非公開ユーザーの投稿は、権限の無い閲覧者の結果に出ない。削除した投稿は出ない
- [ ] 空・除外のみなどは 400 になる。語数・文字数に任意の上限が無く、数百語の入力も受け付ける
- [ ] 検索がタイムアウトしたとき 503 を返し、他機能の接続を奪わない（専用プール）。レート制限が効く

## 参照する設計ドキュメント

- design-delta.md「投稿・検索・画像」（検索の負荷保護）「性能・費用の検証計画」
- ADR: 20261030-search-pg-bigm.md、20261030-timeline-fan-out-on-read.md

## BP見積もり

作成するファイル: `apps/api/drizzle/0007_search_index.sql`, `apps/api/src/db/search-pool.ts`, `apps/api/src/services/search.ts`, `apps/api/src/routes/search.ts`, `packages/contract/src/search.ts`, `apps/api/src/services/search.test.ts`, `apps/api/src/routes/search.test.ts`, `apps/api/src/services/search.timeout.test.ts`, `apps/api/src/services/search.plan.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 9 | 3（ベース） |
| 実装行数（テスト含む） | 700 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 6 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
