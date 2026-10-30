# Build 28: infra: CDK基盤・ネットワーク・RDS

## やること

- `infra/`（AWS CDK, TypeScript）のワークスペースを作る。環境（本番）ごとの設定値、スタックの分割方針、タグ付け
- ネットワーク: VPC（東京リージョン ap-northeast-1、複数 AZ）、サブネット、NAT の構成（費用を見積もって決める。S3 は Gateway エンドポイント）、セキュリティグループ
- RDS PostgreSQL: Multi-AZ、pg_bigm を有効化するパラメータグループ、バックアップとポイントインタイムリカバリ、暗号化、削除保護、インスタンスクラス（見積もりで決める）。DB の資格情報は Secrets Manager
- `cdk synth` が通ることと、リソースのスナップショットテスト
- 月額費用の見積もり表（この時点の構成分）を `infra/README.md` に記す

## やらないこと

- S3・SES・ECR（build-29）
- ECS・ALB（build-30）、デプロイ・監視（build-31）
- 実環境への `cdk deploy`（本ビルドでは synth とテストまで）

## 対応する受け入れ基準

なし（非機能要件: 本番構成・可用性・データの保護）

## 受け入れ基準

- [ ] `cdk synth` が通り、VPC・RDS（Multi-AZ・pg_bigm・バックアップ・暗号化）が CloudFormation テンプレートに含まれる
- [ ] DB が公開サブネットに置かれず、アプリケーションのセキュリティグループからのみ接続できる
- [ ] スナップショットテストが通る
- [ ] 費用見積もりが記され、月額300米ドルの目安に対する現時点の合計が分かる

## 参照する設計ドキュメント

- design-delta.md「本番構成（AWS 東京リージョン）」
- ADR: 20261030-search-pg-bigm.md（pg_bigm）、20261030-timeline-fan-out-on-read.md（プライマリの垂直スケール）

## BP見積もり

作成するファイル: `infra/package.json`, `infra/tsconfig.json`, `infra/cdk.json`, `infra/bin/app.ts`, `infra/lib/config.ts`, `infra/lib/network-stack.ts`, `infra/lib/database-stack.ts`, `infra/test/network-stack.test.ts`, `infra/test/database-stack.test.ts`, `infra/README.md`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 10 | 3（ベース） |
| 実装行数（テスト含む） | 600 | 3 |
| API操作数 | 0 | 1 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 1 | 2 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 4 | +1 |
| 基盤セットアップを含む | あり | +1 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
