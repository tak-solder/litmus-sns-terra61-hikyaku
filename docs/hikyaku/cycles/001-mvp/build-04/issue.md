# Build 04: ログイン・ログアウトAPI

## やること

- `sessions` テーブルとマイグレーション（ハッシュ化したトークン、有効期限、最終利用）
- `POST /api/auth/login`: メールアドレスとパスワードの検証。確認前のユーザーは、正しい資格情報でも拒否して「確認が必要」を返す。誤りは区別のつかない汎用エラー。成功時に不透明トークンを発行する
- `POST /api/auth/logout`: 該当セッションの失効
- `GET /api/auth/me`: 認証済みユーザーの取得
- 認証ミドルウェア（Bearer）。ゲストは未認証として扱う。最終利用の更新は一定間隔に間引く
- ログインへのレート制限の適用

## やらないこと

- 画像配信用の cookie 認証（build-16）
- 確認メールの再送（build-05）、パスワードリセット（build-06）
- Web の画面とセッション cookie（build-07）

## 対応する受け入れ基準

- US-1.6, US-2.1, US-2.2, US-2.3

## 受け入れ基準

- [ ] 確認済みのユーザーは正しい資格情報でログインできる
- [ ] 確認前のユーザーは、正しい資格情報でもログインできず、確認が必要な旨が返る
- [ ] 誤ったメールアドレスまたはパスワードはエラーになる（どちらが誤りかは分からない）
- [ ] ログアウト後はそのトークンが無効になり、以降はゲストとして扱われる

## 参照する設計ドキュメント

- design-delta.md「認証・メール」
- ADR: 20261030-session-token-auth.md

## BP見積もり

作成するファイル: `apps/api/src/db/schema/sessions.ts`, `apps/api/drizzle/0002_sessions.sql`, `apps/api/src/services/session.ts`, `apps/api/src/middleware/auth.ts`, `apps/api/src/routes/auth-session.ts`, `packages/contract/src/session.ts`, `apps/api/src/services/session.test.ts`, `apps/api/src/routes/auth-session.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 8 | 3 |
| 実装行数（テスト含む） | 600 | 3 |
| API操作数 | 3 | 5（ベース） |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 1 | 2 |
| ベースBP（指標の最大値） | — | 5 |
| 影響ファイル数 | 3 | +0 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
