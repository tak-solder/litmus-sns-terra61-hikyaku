# Build 03: ユーザー登録とメールアドレス確認API

## やること

- `users` / `email_tokens` テーブルとマイグレーション（visibility の既定は public、`following_count` を含む）
- `POST /api/auth/register`: 名前・メールアドレス・パスワード（8文字以上、argon2id でハッシュ化）の検証と登録。メールアドレスは小文字化して一意。既存メールでも新しいアカウントは作らず、応答は同じ成功表示にし、既存の持ち主には通知メールを送る（アカウント列挙の防止）。未入力・形式不正は理由つきの 400
- `GET/POST /api/auth/verify-email`: トークンによる確認の完了
- メール送信部品（nodemailer の SMTP。接続先を環境変数で切り替え、ローカルは Mailpit、本番は SES）と、確認メールのテンプレート
- 登録エンドポイントへのレート制限の適用

## やらないこと

- ログイン・セッション（build-04）、確認メールの再送（build-05）
- パスワードリセット（build-06）
- Web の画面（build-07、08）

## 対応する受け入れ基準

- US-1.1, US-1.2, US-1.3, US-1.4, US-1.5, US-1.7

## 受け入れ基準

- [ ] 名前・メールアドレス・パスワードで登録でき、登録直後の公開範囲は「公開」になる
- [ ] 既存のメールアドレスでは新しいアカウントが作られず、応答からアカウントの有無が分からない
- [ ] 未入力・形式不正・8文字未満のパスワードは理由つきで拒否される
- [ ] 登録すると確認メールが Mailpit に届き、リンクのトークンで確認が完了する。期限切れ・使用済みトークンは拒否される
- [ ] トークンはハッシュ化して保存される

## 参照する設計ドキュメント

- design-delta.md「データモデル」「認証・メール」
- ADR: 20261030-session-token-auth.md

## BP見積もり

作成するファイル: `apps/api/src/db/schema/users.ts`, `apps/api/src/db/schema/email-tokens.ts`, `apps/api/drizzle/0001_users.sql`, `apps/api/src/lib/password.ts`, `apps/api/src/lib/mailer.ts`, `apps/api/src/mail/templates.ts`, `apps/api/src/services/registration.ts`, `apps/api/src/routes/auth-register.ts`, `packages/contract/src/auth.ts`, `apps/api/src/services/registration.test.ts`, `apps/api/src/routes/auth-register.test.ts`, `apps/api/src/lib/password.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 12 | 3（ベース） |
| 実装行数（テスト含む） | 900 | 3 |
| API操作数 | 2 | 3 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 2 | 3 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | あり | +1 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
