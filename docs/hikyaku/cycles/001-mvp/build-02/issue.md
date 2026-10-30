# Build 02: DB基盤・API共通基盤・レート制限

## やること

- Drizzle の導入（接続、`drizzle.config`、マイグレーションの実行コマンド）。compose 起動時にマイグレーションを実行する
- API 共通基盤: 環境変数の読み込みと検証、構造化ログ、エラー形式（`packages/contract` のエラー型に従う統一レスポンス）、OpenAPI（zod-openapi）の組み込みと仕様書の配信、UUIDv7 の ID 生成、キーセットページングの共通部品
- `/health` に DB 疎通を加える
- レート制限: `rate_limits` テーブル（マイグレーション）と、種別・キー単位で制限をかけるミドルウェア。カウンタは DB に置く（複数タスクで共有するため）
- 認証ミドルウェアの差し込み口（実装は build-04）

## やらないこと

- users などの業務テーブル（各機能のビルドで追加する）
- 認証・セッション（build-04）
- 検索専用の接続プール（build-21）

## 対応する受け入れ基準

なし（基盤整備。ユーザーストーリーに直接対応しない）

## 受け入れ基準

- [ ] `docker compose up` でマイグレーションが適用され、`/health` が DB 疎通込みで ok を返す
- [ ] エラー時に統一形式のレスポンスが返り、OpenAPI の仕様書が取得できる
- [ ] レート制限ミドルウェアが、上限を超えたリクエストを 429 で拒否するテストが通る（複数インスタンスを想定し、カウンタが DB 上にあることをテストで確認する）
- [ ] マイグレーションは追加のみで、再実行しても安全（冪等）

## 参照する設計ドキュメント

- design-delta.md「データモデル」「認証・メール」（レート制限）「品質・運用の基盤」
- ADR: 20261030-rest-openapi-shared-contract.md

## BP見積もり

作成するファイル: `apps/api/src/db/client.ts`, `apps/api/src/db/migrate.ts`, `apps/api/drizzle.config.ts`, `apps/api/src/db/schema/rate-limits.ts`, `apps/api/drizzle/0000_rate_limits.sql`, `apps/api/src/config.ts`, `apps/api/src/logger.ts`, `apps/api/src/errors.ts`, `apps/api/src/openapi.ts`, `apps/api/src/lib/id.ts`, `apps/api/src/lib/pagination.ts`, `apps/api/src/middleware/rate-limit.ts`, `apps/api/src/middleware/rate-limit.test.ts`, `apps/api/src/health.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 14 | 3（ベース） |
| 実装行数（テスト含む） | 750 | 3 |
| API操作数 | 1 | 2 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 1 | 2 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | あり | +1 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
