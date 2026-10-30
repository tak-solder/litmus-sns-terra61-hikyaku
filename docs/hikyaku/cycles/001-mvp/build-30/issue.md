# Build 30: infra: ECS Fargate・ALB・オートスケール・本番イメージ

## やること

- 本番用 Dockerfile（web・api。マルチステージ、非 root、`pnpm` のワークスペースを考慮）。ヘルスチェックの設定
- ECS クラスターと Fargate サービス（web・api、各2タスク以上、複数 AZ）。タスクロール（S3・SES・Secrets）、環境変数、ログ（CloudWatch Logs）
- ALB: HTTPS（ACM 証明書は値で渡す）、パスで `/api/*` を api、それ以外を web に振り分ける。`/api/images/*` を将来専用サービスに分離できるよう、パスルールを独立して定義する。WAF の関連付けは任意
- オートスケール: CPU 目標追跡（低めの閾値）と最小台数。単体ページの急増（毎分5,000件・30分）に備えた設定。ターゲットのスケールアウト／スケールインのクールダウン
- マイグレーション実行用の one-off タスク定義
- スナップショットテスト、費用見積もりの更新（Multi-AZ の RDS・NAT・ALB・Fargate の常時分を含む合計）

## やらないこと

- デプロイパイプライン、アラーム（build-31）
- CloudFront による投稿データのキャッシュ（採用しない。ADR 参照）
- 実環境への `cdk deploy`

## 対応する受け入れ基準

なし（非機能要件: 本番構成・可用性・急増への耐性）

## 受け入れ基準

- [ ] 本番用イメージがビルドでき、コンテナで web・api が起動して `/health` が通る
- [ ] `cdk synth` が通り、ECS（複数 AZ）・ALB（パス振り分け・HTTPS）・オートスケールがテンプレートに含まれる
- [ ] 画像配信のパスルールを独立して定義している（専用サービスへ分離できる）
- [ ] 費用見積もりが更新され、月額300米ドルの目安に対する合計と超過時の対策案が記されている

## 参照する設計ドキュメント

- design-delta.md「本番構成（AWS 東京リージョン）」「性能・費用の検証計画」
- ADR: 20261030-no-cdn-cache-for-posts.md、20261030-image-per-request-authz.md

## BP見積もり

作成するファイル: `docker/web.Dockerfile`, `docker/api.Dockerfile`, `.dockerignore`, `infra/lib/cluster-stack.ts`, `infra/lib/service-web.ts`, `infra/lib/service-api.ts`, `infra/lib/alb.ts`, `infra/lib/autoscaling.ts`, `infra/lib/migration-task.ts`, `infra/test/cluster-stack.test.ts`, `infra/test/alb.test.ts`, `infra/test/autoscaling.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 12 | 3（ベース） |
| 実装行数（テスト含む） | 700 | 3 |
| API操作数 | 0 | 1 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 6 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
