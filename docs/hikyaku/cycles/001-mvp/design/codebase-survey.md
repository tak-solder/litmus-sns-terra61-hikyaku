# 既存コードベース調査結果

永続ドキュメント（overview 等）は未作成のため、全体を調査した。

## リポジトリ概要

- **技術スタック**: TypeScript / Next.js 16.4.0（React 19.3.0, `cacheComponents` と `partialPrefetching` が有効）/ Hono 4.13.0 + `@hono/node-server`
- **パッケージマネージャ**: pnpm 11.5.2（Node 26.12.0 固定）。ワークスペースは `apps/*`
- **未導入**: DB、ORM、認証、テスト基盤、lint、CI、IaC、メール、画像ストレージ。すべて新規に決める

## ディレクトリ構成

```
apps/api/src/        Hono。app.ts（GET /health のみ）と index.ts（serve、PORT 既定 8080）
apps/web/app/        Next.js App Router の雛形（layout / page / globals.css）
docker/Dockerfile.dev  web と api で共用の開発用イメージ（compose の command で切替）
compose.yaml         web(3000) と api(8080) の2サービスのみ。DB・メールキャッチャーは無い
```

## ユーザーストーリーとの関連箇所

既存コードはほぼ雛形で、ストーリーに直接関わる実装は無い。

- `apps/api/src/app.ts` — ルートの追加先。`app` を export しており、テストから `app.request()` で呼べる形
- `compose.yaml` — DB（PostgreSQL）・メールキャッチャー・マイグレーション実行を足す対象（成果物要件の `docker compose up` 1コマンド起動）
- `docker/Dockerfile.dev` — `COPY` が `apps/web` と `apps/api` の package.json 個別指定。パッケージを増やすとここの更新が要る

## 既存の規約・パターン

- **命名規約**: 雛形のみで未確立
- **ディレクトリ構成方針**: モノレポ `apps/*`。共有パッケージ用の `packages/*` は未定義（`pnpm-workspace.yaml` は `apps/*` のみ）
- **エラーハンドリング**: 未確立
- **テスト**: 未導入。`typecheck` と `build` のみがルートスクリプトにある
- **ESM**: api は `"type": "module"` で、相対 import に `.js` 拡張子を付ける（`./app.js`）
- **Next.js**: `apps/web/AGENTS.md` により、`node_modules/next/dist/docs/` の該当ガイドを読んでから実装する（破壊的変更あり）

## 拡張ポイント

- `apps/api/src/app.ts` — Hono アプリ。ルート・ミドルウェアの接続点
- `pnpm-workspace.yaml` — `packages/*` を追加すれば、API 契約や DB スキーマを web / api で共有できる
- `compose.yaml` — 依存サービスの追加先
