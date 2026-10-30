# Build 06: パスワードリセットAPI

## やること

- `POST /api/auth/password-reset/request`: メールアドレスを受けてリセットメールを送る。応答はアカウントの有無が分からない内容にする。レート制限を適用する
- `POST /api/auth/password-reset/confirm`: トークンと新しいパスワード（8文字以上）で更新する。トークンは期限付き・使用済み不可。更新後は、そのユーザーの全セッションを失効させる
- リセットメールのテンプレート

## やらないこと

- Web の画面（build-08）
- ログイン中のパスワード変更（スコープ外）

## 対応する受け入れ基準

- US-3.1, US-3.2

## 受け入れ基準

- [ ] 登録済みのメールアドレスを入力すると、リセットメールが Mailpit に届く
- [ ] リンクのトークンで新しいパスワードを設定すると、新しいパスワードでログインでき、古いパスワードではログインできない
- [ ] リセット後、既存のセッションは無効になる
- [ ] 期限切れ・使用済みトークンは拒否される

## 参照する設計ドキュメント

- design-delta.md「認証・メール」
- ADR: 20261030-session-token-auth.md

## BP見積もり

作成するファイル: `apps/api/src/services/password-reset.ts`, `apps/api/src/routes/auth-password-reset.ts`, `apps/api/src/mail/password-reset-template.ts`, `packages/contract/src/password-reset.ts`, `apps/api/src/services/password-reset.test.ts`, `apps/api/src/routes/auth-password-reset.test.ts`, `apps/api/src/mail/password-reset-template.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 7 | 3（ベース） |
| 実装行数（テスト含む） | 550 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 4 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
