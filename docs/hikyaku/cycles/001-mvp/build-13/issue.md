# Build 13: フォロー申請の一覧・承認・拒否API

## やること

- `GET /api/me/follow-requests`: 自分宛ての未処理（pending）の申請を、申請者の名前つきで、申請日時（`created_at`）の新しい順に1回最大20件返し、`(created_at, follower_id)` の複合キーセットで続きを取得できる
- `PUT /api/me/follow-requests/{followerId}`: 本文の `decision`（approve / reject）で処理する。承認は accepted にして `accepted_at` を設定する（`following_count` は申請時に加算済みのため変えない）。拒否は申請を削除し、申請者の `following_count` を減らす。拒否後は申請者が改めて申請できる。対象は自分宛ての申請に限り、それ以外は 404
- 結合テスト: 承認後に build-11 の単体取得で申請者が非公開の投稿を見られること、拒否後は見られないことを確認する

## やらないこと

- フォロワー一覧・フォロワーを外す（build-14）
- Web の申請一覧画面（build-25）

## 対応する受け入れ基準

- US-13.1, US-13.2, US-13.3

## 受け入れ基準

- [ ] 自分宛ての未処理の申請が、申請者の名前つきで新しい順に最大20件ずつ取得できる
- [ ] 承認すると申請者がフォロワーになり、投稿を見られるようになる
- [ ] 拒否すると申請が消えて申請者はフォロワーにならず、改めて申請できる
- [ ] 他人宛ての申請は処理できない

## 参照する設計ドキュメント

- design-delta.md「データモデル」`follows`（ページングのキー）、「閲覧権限の条件」（書き込み系の認可）

## BP見積もり

作成するファイル: `apps/api/src/services/follow-request.ts`, `apps/api/src/routes/follow-requests.ts`, `packages/contract/src/follow-request.ts`, `apps/api/src/services/follow-request.test.ts`, `apps/api/src/routes/follow-requests.test.ts`, `apps/api/src/lib/pagination.test.ts`（複合キーセットの追加分）, `apps/api/src/test/fixtures/follow-requests.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 7 | 3（ベース） |
| 実装行数（テスト含む） | 600 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
