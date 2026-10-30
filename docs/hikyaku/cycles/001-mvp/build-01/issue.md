# Build 01: 開発基盤（モノレポ・compose・CI・テスト基盤）

## やること

- `pnpm-workspace.yaml` に `packages/*` を追加し、`packages/contract`（zod スキーマ置き場。中身は空の雛形とエラー形式の型）を作る
- `docker/Dockerfile.dev` の COPY を、新しいパッケージを含む形に更新する
- `compose.yaml` に PostgreSQL（pg_bigm を含むカスタムイメージ。`docker/postgres/Dockerfile`）、Mailpit、S3 互換ストレージ（MinIO）を追加し、web・api の環境変数を渡す。`docker compose up` の1コマンドで全体が起動する状態にする
- lint（ESLint）、フォーマット、テスト基盤（api は Vitest + `app.request()`、web も Vitest）、ルートスクリプト（`lint` / `test`）を整える
- CI（GitHub Actions: typecheck / lint / test / build）を追加する
- `.env.example` と、README の最小限の起動手順を書く

## やらないこと

- DB スキーマ・マイグレーション・DB クライアント（build-02）
- 認証、各機能の API・画面
- 本番用の Dockerfile と AWS のコード（build-28 以降）
- README の完成（build-33）

## 対応する受け入れ基準

なし（基盤整備。ユーザーストーリーに直接対応しない）

## 受け入れ基準

- [ ] `docker compose up` で web・api・PostgreSQL（pg_bigm が有効）・Mailpit・MinIO が起動し、Mailpit の画面をブラウザで開ける
- [ ] `pnpm typecheck` / `pnpm lint` / `pnpm test` / `pnpm build` がローカルと CI の両方で通る
- [ ] api の `/health` を叩くテストが Vitest で動く
- [ ] `packages/contract` を web と api の両方から import できる

## 参照する設計ドキュメント

- design-delta.md「全体構成」「既存構造への影響」「品質・運用の基盤」
- codebase-survey.md「既存の規約・パターン」「拡張ポイント」
- ADR: 20261030-rest-openapi-shared-contract.md、20261030-search-pg-bigm.md（pg_bigm を含む DB イメージ）
- `apps/web/AGENTS.md`（Next.js 16 の該当ガイドを読んでから実装する）

## BP見積もり

作成するファイル: `packages/contract/package.json`, `packages/contract/tsconfig.json`, `packages/contract/src/index.ts`, `packages/contract/src/errors.ts`, `docker/postgres/Dockerfile`, `docker/postgres/init.sql`, `eslint.config.mjs`, `.prettierrc`, `.env.example`, `.github/workflows/ci.yml`, `apps/api/vitest.config.ts`, `apps/api/src/app.test.ts`, `apps/web/vitest.config.ts`, `apps/web/app/page.test.tsx`, `README.md`（更新を含めず新規のみ数えて概算）

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 15 | 3（ベース） |
| 実装行数（テスト含む） | 450 | 2 |
| API操作数 | 0 | 1 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 6 | +1 |
| 基盤セットアップを含む | あり | +1 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
