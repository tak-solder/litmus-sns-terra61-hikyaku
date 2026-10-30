# Build 09: フォロー・フォロー解除API

## やること

- `follows` テーブルとマイグレーション（status: accepted / pending、created_at、accepted_at）
- `PUT /api/users/{id}/follow`: 相手が公開なら accepted、非公開なら pending で作成する。ゲストは 401、自分自身は 400
- `DELETE /api/users/{id}/follow`: フォロー解除、または申請中の取り消し
- フォロー上限: トランザクション内でフォローする側のユーザー行をロックし、`following_count`（accepted + pending）が5,000未満のときだけ追加する。到達時は理由つきのエラー。`following_count` の増減は、フォロー・申請・解除・取り消しのすべてで同一トランザクション内に行う
- 閲覧者と対象ユーザーの関係（none / pending / accepted / self）を求める共通関数（build-19 のユーザーページ API で使う）

## やらないこと

- 申請の一覧・承認・拒否（build-13）、フォロワーを外す（build-14）
- 公開から非公開、非公開から公開への切り替えと、それに伴う申請の破棄（build-12）
- 投稿の閲覧権限の判定（build-11）

## 対応する受け入れ基準

- US-6.3, US-6.4, US-6.5, US-6.6, US-6.7, US-6.8, US-6.10

## 受け入れ基準

- [ ] 公開ユーザーをフォローでき、フォロー済みの状態が取得でき、解除できる
- [ ] 非公開ユーザーには申請になり、申請中の状態が取得でき、申請者が取り消せる。承認されるまでフォロワーにならない
- [ ] 自分自身はフォローできない。ゲストはフォローできない
- [ ] フォロー中と申請中の合計が5,000に達するとフォロー・申請ができず、その旨のエラーが返る。同時リクエストでも上限を超えない（並行テストで確認する）

## 参照する設計ドキュメント

- design-delta.md「データモデル」`follows`、「閲覧権限の条件」（フォロー上限、`following_count`）
- ADR: 20261030-timeline-fan-out-on-read.md

## BP見積もり

作成するファイル: `apps/api/src/db/schema/follows.ts`, `apps/api/drizzle/0003_follows.sql`, `apps/api/src/services/follow.ts`, `apps/api/src/services/relationship.ts`, `apps/api/src/routes/follow.ts`, `packages/contract/src/follow.ts`, `apps/api/src/services/follow.test.ts`, `apps/api/src/services/follow-limit.concurrency.test.ts`, `apps/api/src/routes/follow.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 9 | 3（ベース） |
| 実装行数（テスト含む） | 700 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 1 | 2 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
