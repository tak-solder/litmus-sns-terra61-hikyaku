# Build 12: 公開範囲設定API

## やること

- `GET /api/me/visibility` と `PUT /api/me/visibility`: 自分の公開範囲（公開 / 非公開）を取得・変更する
- 非公開から公開への切り替え: 同一トランザクションで、そのユーザー宛ての pending のフォロー申請をすべて破棄し、申請者の `following_count` を減らす
- 公開から非公開への切り替え: 既存の accepted のフォロワーはそのまま残す
- 結合テスト: 切り替え後に、build-11 の閲覧権限（単体取得）が過去の投稿を含めて即時に変わることを、公開 → 非公開 → 公開の往復で確認する

## やらないこと

- 申請の承認・拒否（build-13）、フォロワーを外す（build-14）
- Web の設定画面（build-25）

## 対応する受け入れ基準

- US-11.1, US-11.3, US-11.4, US-11.5

## 受け入れ基準

- [ ] 現在の公開範囲を取得でき、公開と非公開を切り替えられる
- [ ] 非公開に切り替えても、その時点のフォロワーは残り、投稿を見られる
- [ ] 非公開から公開に切り替えると、未処理の申請がすべて破棄され、申請していたユーザーの `following_count` が減る。改めてフォローすればフォロワーになれる
- [ ] 切り替え直後のリクエストから、過去の投稿を含めて閲覧可否が変わる（結合テスト）

## 参照する設計ドキュメント

- design-delta.md「閲覧権限の条件」（非公開から公開への切り替え、読み取り元）
- ADR: 20261030-no-cdn-cache-for-posts.md

## BP見積もり

作成するファイル: `apps/api/src/services/visibility-setting.ts`, `apps/api/src/routes/visibility-setting.ts`, `packages/contract/src/visibility.ts`, `apps/api/src/services/visibility-setting.test.ts`, `apps/api/src/routes/visibility-setting.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 5 | 2 |
| 実装行数（テスト含む） | 400 | 2 |
| API操作数 | 2 | 3（ベース） |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
