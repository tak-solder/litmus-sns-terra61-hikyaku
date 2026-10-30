# Build 14: フォロワー一覧・フォロワーを外すAPI

## やること

- `GET /api/me/followers`: 自分の accepted のフォロワーを、名前つきで、フォロー成立日時（`accepted_at`）の新しい順に1回最大20件返し、`(accepted_at, follower_id)` の複合キーセットで続きを取得できる
- `DELETE /api/me/followers/{followerId}`: フォロワーを外す。外された側の `following_count` を同一トランザクションで減らす。外された人は、自分が非公開なら改めて申請でき、公開なら改めてフォローできる
- 結合テスト: フォロワーを外す、またはフォロワー自身が解除（build-09）した直後に、build-11 の単体取得で非公開の投稿が見えなくなることを確認する

## やらないこと

- 他ユーザーのフォロワー一覧、フォロー中一覧（スコープ外）
- Web の一覧画面（build-26）

## 対応する受け入れ基準

- US-14.1, US-14.2, US-14.3, US-11.6

## 受け入れ基準

- [ ] 自分のフォロワーが名前つきで新しい順に最大20件ずつ取得できる
- [ ] フォロワーを外すと、その人は自分をフォローしていない状態になる
- [ ] 外された人は、自分の公開範囲に応じて、改めて申請またはフォローできる
- [ ] フォロー解除・フォロワーを外す操作の直後から、非公開の投稿が見えなくなる（結合テスト）

## 参照する設計ドキュメント

- design-delta.md「データモデル」`follows`、「閲覧権限の条件」

## BP見積もり

作成するファイル: `apps/api/src/services/follower.ts`, `apps/api/src/routes/followers.ts`, `packages/contract/src/follower.ts`, `apps/api/src/services/follower.test.ts`, `apps/api/src/routes/followers.test.ts`, `apps/api/src/test/fixtures/followers.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 6 | 3（ベース） |
| 実装行数（テスト含む） | 500 | 2 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 4 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
