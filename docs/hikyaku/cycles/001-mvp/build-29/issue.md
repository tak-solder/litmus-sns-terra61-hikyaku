# Build 29: infra: S3・SES・ECR・Secrets

## やること

- 画像用 S3 バケット: パブリックアクセスを全面的にブロック、暗号化、バージョニングの方針、ライフサイクル。アプリケーションのタスクロールにだけ読み書きを許可する（CDN や署名 URL は使わない）
- SES: 送信ドメインの ID、DKIM、送信元アドレスの設定（ドメイン名はコンテキスト値で渡す）、送信用の SMTP 資格情報または IAM 権限、バウンス・苦情の通知先
- ECR リポジトリ（web・api）とライフサイクルポリシー
- Secrets Manager: アプリケーションの設定値（外部に出さない値）の置き場所と、タスクへの渡し方
- スナップショットテスト、費用見積もりの更新

## やらないこと

- ECS・ALB（build-30）、デプロイ・監視（build-31）
- ドメインの取得・DNS の本番設定（運用手順として README に書くにとどめる）

## 対応する受け入れ基準

なし（非機能要件: 本番構成・画像の秘匿・メール送信）

## 受け入れ基準

- [ ] `cdk synth` が通り、S3（全面ブロック・暗号化）・SES・ECR・Secrets がテンプレートに含まれる
- [ ] S3 に公開ポリシーが無く、読み書きがアプリケーションのロールに限られることをテストで確認する
- [ ] 費用見積もりが更新されている

## 参照する設計ドキュメント

- design-delta.md「本番構成（AWS 東京リージョン）」「認証・メール」
- ADR: 20261030-image-per-request-authz.md

## BP見積もり

作成するファイル: `infra/lib/storage-stack.ts`, `infra/lib/mail-stack.ts`, `infra/lib/registry-stack.ts`, `infra/lib/secrets-stack.ts`, `infra/test/storage-stack.test.ts`, `infra/test/mail-stack.test.ts`, `infra/test/registry-stack.test.ts`, `infra/test/secrets-stack.test.ts`

| 項目 | 値 | BP |
|------|----|----|
| 新規ファイル数 | 8 | 3 |
| 実装行数（テスト含む） | 450 | 2 |
| API操作数 | 0 | 1 |
| 画面/ページ数 | 0 | 1 |
| DBテーブル/エンティティ数 | 0 | 1 |
| ベースBP（指標の最大値） | — | 3 |
| 影響ファイル数 | 4 | +1 |
| 基盤セットアップを含む | 未指定 | +0 |
| 外部API連携を含む | あり | +1 |
| 既存コードの大規模リファクタ | 未指定 | +0 |
| **合計** | — | **5BP** |
