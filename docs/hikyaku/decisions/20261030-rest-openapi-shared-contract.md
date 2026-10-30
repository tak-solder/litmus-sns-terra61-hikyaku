---
status: accepted
---

# API は REST と OpenAPI で作り、契約を共有パッケージに置く

- **決定**: Hono に zod-openapi を使い、スキーマ定義から OpenAPI の仕様書を生成する。リクエスト・レスポンスのスキーマは `packages/contract` に置き、web と api が共有する。
- **文脈**: モバイルアプリからも使える API が必要。成果物として API 仕様書が要る。API の方式は自由。
- **検討した案**:
  - 案A: REST + OpenAPI
  - 案B: GraphQL
  - 案C: tRPC
- **採用理由**: 仕様書を実装から自動生成できる。モバイルを含むどのクライアントからも標準的に使える。HTTP の標準的な振る舞い（認可の失敗は 401/403/404）をそのまま使える。
- **トレードオフ**: 契約パッケージの追加に伴い、ワークスペース設定と `docker/Dockerfile.dev` の COPY の更新が要る。画面ごとに必要なデータを一度に取りたい場面で、複数リクエストになりやすい。GraphQL の柔軟性は手放す。
- **影響範囲**: `pnpm-workspace.yaml`、`docker/Dockerfile.dev`、`apps/api` のルート定義、`apps/web` の API クライアント。
