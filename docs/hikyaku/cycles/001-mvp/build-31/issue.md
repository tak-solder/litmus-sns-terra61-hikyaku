# Build 31: infra: デプロイパイプラインと監視

## やること

- デプロイ（GitHub Actions）: イメージのビルド、ECR への push、マイグレーションの one-off ECS タスク実行、ECS サービスのローリング更新、失敗時のロールバック手順。マイグレーションは新旧のコードが同時に動く時間があるため後方互換（追加してから切り替え、最後に削除する）に限る、という規約を README に記す
- CloudWatch アラーム: ALB の 5xx 率と応答時間、ECS のタスク数と CPU、RDS の CPU・接続数・空き容量・レプリカ遅延以外の主要指標。通知先（SNS）
- ALB ターゲットの `/health` 死活監視、月間稼働率（99.9%）を ALB のメトリクスから算出する方法（ダッシュボードまたは手順）
- 構造化ログの検索用クエリ（CloudWatch Logs Insights）の例
- スナップショットテスト、運用手順（Runbook: デプロイ、ロールバック、マイグレーション、障害時の確認）

## やらないこと

- 実環境への初回デプロイの実行（手順の整備まで）
- 負荷試験（build-32、33）

## 対応する受け入れ基準

なし（非機能要件: リリースの繰り返し・可用性の計測）

## 受け入れ基準

- [ ] デプロイワークフローが、ビルド → push → マイグレーション → 更新の順で定義され、ドライランまたは構文検証が CI で通る
- [ ] 主要なアラームが CDK で定義され、`cdk synth` が通る
- [ ] 稼働率の算出方法と、ロールバック・マイグレーションの手順が Runbook に書かれている

## 参照する設計ドキュメント

- design-delta.md「品質・運用の基盤」（デプロイ・監視・稼働率）

## BP見積もり

作成するファイル: `.github/workflows/deploy.yml`, `infra/lib/monitoring-stack.ts`, `infra/lib/dashboards.ts`, `infra/test/monitoring-stack.test.ts`, `docs/runbook/deploy.md`, `docs/runbook/rollback.md`, `docs/runbook/migration.md`, `docs/runbook/incident.md`, `scripts/deploy/*.sh`（スクリプト 1ファイル）

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 9 | 3（ベース） |
| 実装行数（テスト含む） | 550 | 3 |
| API操作数 | 0 | 1 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 5 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | 未指定 | +0 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **4BP** |
