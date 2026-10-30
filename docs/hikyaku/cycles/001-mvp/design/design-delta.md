# 設計差分

## このサイクルで作るもの

新規開発で、既存の本番データは無い。すべてのテーブル・エンドポイントは追加のみで、旧形式の撤去や移行は発生しない。

### 全体構成

ブラウザ・モバイルは、ALB 越しに同一オリジンで `/api/*` を api サービス、それ以外を web サービスへ振り分ける。web（Next.js）は BFF としてサーバー側から api を呼び、api（Hono）がビジネスロジックと DB アクセスの唯一の入口になる。DB は PostgreSQL、画像は非公開の S3 バケットに置く。

```
apps/web        Next.js。画面と BFF（cookie 付きセッションを Bearer に変換して api を呼ぶ）
apps/api        Hono。REST + OpenAPI、認証・認可、DB、画像、メール送信
packages/contract  zod スキーマ（リクエスト・レスポンス・エラー形式）。web と api が共有
infra/          AWS CDK（TypeScript）。本番の全リソース
```

認可の判定（誰がどの投稿を見られるか）は api の一箇所にまとめ、一覧・検索・単体ページ・画像で共通の条件を使う。

### データモデル

主キーは UUIDv7（時刻順）で、一覧のページングは id のキーセットで行う。

- `users`: id, name, email（小文字化して一意）, password_hash（argon2id）, email_verified_at, visibility（public / private）, following_count（フォロー中 + 申請中。上限5,000の判定用）, created_at
- `email_tokens`: ユーザー、種別（確認 / リセット）、ハッシュ化したトークン、有効期限、使用済み
- `sessions`: ユーザー、ハッシュ化したトークン、有効期限、最終利用
- `posts`: id, author_id, body, search_text（NFKC 正規化 + 小文字）, image_id, like_count, deleted_at。インデックスは `(author_id, id desc)`、`id desc`、`search_text` の pg_bigm GIN
- `images`: id（ランダム）, post_id, s3 キー, content_type, size
- `likes`: `(post_id, user_id)` 主キー。`user_id` のインデックス
- `follows`: `(follower_id, followee_id)` 主キー、status（accepted / pending）、created_at。`(followee_id, status, created_at desc)` のインデックス（フォロワー一覧・申請一覧用）

### 閲覧権限の条件

投稿を見られるのは、削除されておらず、かつ投稿者が公開、または閲覧者が投稿者本人か accepted のフォロワーである場合。公開範囲はユーザー行を参照するため、変更は過去の投稿を含めて即時に反映される。フォロー解除・フォロワーを外す操作・削除も同様に、次のリクエストから反映される。

- タイムライン: `follows.status = accepted` のフォロー先と自分の投稿。pending は含めない
- グローバルフィード・検索・ユーザーページ: 上の閲覧権限の条件でフィルタ。ゲストは公開ユーザーの投稿のみ
- 非公開から公開への切り替え: 同一トランザクションで、そのユーザー宛ての pending を削除し、申請者の `following_count` を減らす
- フォロー上限: トランザクション内でフォローする側のユーザー行をロックし、`following_count` が5,000未満のときだけ追加する

### 認証・メール

ADR「認証は DB 保存の不透明トークンのセッションで行う」に従う。登録時に確認メール、パスワードリセット時にリセットメールを送る。トークンはハッシュ化して保存し、有効期限と使用済みを持つ。パスワードリセットの完了時は、そのユーザーの全セッションを失効させる。ローカルは Mailpit（SMTP）に届け、本番は SES を使う。メール送信は nodemailer の SMTP で共通化し、接続先を環境変数で切り替える。登録済みメールアドレスの有無が応答から分からないようにする（アカウント列挙の防止）。ログイン・登録・リセット要求にはレート制限を設ける。

### 投稿・検索・画像

投稿の作成時に、本文の検証（300文字以内、空白・改行のみは本文なしとみなす）、画像の検証（マジックバイトで JPEG / PNG / GIF / WebP を判定、5MB 以内、EXIF 除去）、`search_text` の生成を行う。検索は、入力を空白（全角含む）で分割し、`-` 除外、`OR`、`"..."` フレーズを解釈して LIKE の論理式に変換する（ADR「投稿検索は PostgreSQL の pg_bigm と正規化列で実装する」）。条件として成り立たない入力（除外のみなど）は 400 で返す。

画像は ADR「画像はリクエストごとに認可して配信する」に従う。S3 は全面的にパブリックアクセスをブロックし、`GET /api/images/{imageId}` が認可した上で S3 から読んで返す。

### 本番構成（AWS 東京リージョン）

- ECS Fargate に web と api の2サービス（各2タスク以上、複数 AZ）。ALB でパスを振り分ける
- RDS PostgreSQL（pg_bigm を有効化）。Multi-AZ で月間 99.9% を目標にする。バックアップとポイントインタイムリカバリを有効にする
- S3（画像）、SES（メール）、Secrets Manager、CloudWatch Logs
- 単体ページの急増はオートスケールで受ける（ADR「投稿データは CDN でキャッシュせず…」）。最小台数を保ち、CPU のスケール閾値は低めにする
- 費用は月額300米ドルが目安。Multi-AZ の RDS・NAT・ALB・Fargate の常時分で目安に近づくため、インスタンスサイズと NAT の構成は見積りで決め、収まらなければ報告する

### 品質・運用の基盤

テスト基盤（api は `app.request()` で呼べる単体・結合テスト、DB は実 PostgreSQL）、lint、CI（typecheck / test / build）、DB マイグレーション、構造化ログ、ヘルスチェック（`/health`）、README、OpenAPI 仕様書の生成を整える。

### 性能・費用の検証計画

1年後の規模のデータ（ユーザー10万、投稿500万、いいね5,000万、フォロー500万）を投入した DB に対し、負荷試験で次を実測する。結果が要件を満たさない場合は、満たせない理由と対策案を報告する（要件は緩めない）。

- 一覧・単体ページの p95 1秒、検索の p95 3秒、書き込みの p95 1秒（ピーク時の読み取り毎分500件、書き込み毎分50件）
- 単体ページに毎分5,000件が30分続く間の、表示の継続と他機能への影響
- 画像リクエストごとの認可を含む配信の負荷（秒間約170件の目安）と、ECS・転送の費用見積り

## 永続ドキュメントの更新予定

- `overview` の新規作成: 全体構成、コンポーネントの責務、データフロー、認可の判定の置き場所
- `constraints` の新規作成: 性能目標、可用性、公開範囲の即時反映、画像の認可方針、費用の目安
- `decisions`: 今回追記した6件の ADR を `implemented` に更新（close-cycle）
- `conventions` の新規作成: ディレクトリ構成（`apps/*`・`packages/*`・`infra/`）、命名、エラー形式、テストの書き方
- `security-model`: 認証方式、認可の判定、画像配信、レート制限、アカウント列挙の防止（constraints に収まらなければ分離する）
- `db-schema` / `interfaces`: 参考。正はマイグレーションと OpenAPI

## 既存構造への影響

- `pnpm-workspace.yaml`: `packages/*`（と `infra`）を追加する
- `docker/Dockerfile.dev` と `compose.yaml`: パッケージの COPY を更新し、PostgreSQL（pg_bigm を含む）、Mailpit、マイグレーション実行、S3 互換ストレージ（ローカル用。MinIO など）を追加して、`docker compose up` の1コマンドで全体が起動するようにする
- `apps/api/src/app.ts`: ミドルウェアとルートの接続点。`app` を export する形は維持する
- `apps/web`: Next.js 16 の破壊的変更があるため、実装前に `node_modules/next/dist/docs/` の該当ガイドを読む。`cacheComponents` が有効なので、静的シェルと動的データの境界を明確にする
- 追加のみで既存の本番データは無いため、各ビルドをマージしても本番データを壊さない。未完成の機能は画面からの導線を出さないことで隠す
