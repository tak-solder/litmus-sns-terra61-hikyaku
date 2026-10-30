# ビルド一覧 — 001-mvp

ビルドの完了判定は `PR` 列が非空かどうかで行います。
`PR` 列の更新は当該ビルドの PR に同梱されるため、デフォルトブランチ上で
`PR` 列が埋まっていること自体がマージ済みを意味します。

依存関係のあるビルドは、先行ビルドがデフォルトブランチにマージ済みであることが必須です。
依存関係がないビルドは並行実行できます。

| buildID | title                                                 | BP | dependencies           | issue                        | PR |
| ------- | ----------------------------------------------------- | --- | ---------------------- | ---------------------------- | --- |
| 1       | 開発基盤（モノレポ・compose・CI・テスト基盤）         | 5  | —                      | [issue](./build-01/issue.md) | —  |
| 2       | DB基盤・API共通基盤・レート制限                       | 5  | 1                      | [issue](./build-02/issue.md) | —  |
| 3       | ユーザー登録とメールアドレス確認API                   | 5  | 2                      | [issue](./build-03/issue.md) | —  |
| 4       | ログイン・ログアウトAPI                               | 5  | 3                      | [issue](./build-04/issue.md) | —  |
| 5       | 確認メール再送API                                     | 3  | 3                      | [issue](./build-05/issue.md) | —  |
| 6       | パスワードリセットAPI                                 | 4  | 3, 4                   | [issue](./build-06/issue.md) | —  |
| 7       | Web基盤と登録・ログイン画面                           | 5  | 4, 5                   | [issue](./build-07/issue.md) | —  |
| 8       | Web: メール確認・パスワードリセット画面               | 5  | 5, 6, 7                | [issue](./build-08/issue.md) | —  |
| 9       | フォロー・フォロー解除API                             | 4  | 4                      | [issue](./build-09/issue.md) | —  |
| 10      | 投稿の作成・削除API（本文のみ）                       | 4  | 4                      | [issue](./build-10/issue.md) | —  |
| 11      | 投稿単体取得と閲覧権限の共通判定                      | 4  | 9, 10                  | [issue](./build-11/issue.md) | —  |
| 12      | 公開範囲設定API                                       | 4  | 9, 11                  | [issue](./build-12/issue.md) | —  |
| 13      | フォロー申請の一覧・承認・拒否API                     | 4  | 9, 11                  | [issue](./build-13/issue.md) | —  |
| 14      | フォロワー一覧・フォロワーを外すAPI                   | 4  | 13                     | [issue](./build-14/issue.md) | —  |
| 15      | 画像の形式検証・メタデータ除去ライブラリ              | 5  | 1                      | [issue](./build-15/issue.md) | —  |
| 16      | 画像の保存・アップロード・認可配信API                 | 5  | 10, 11, 15             | [issue](./build-16/issue.md) | —  |
| 17      | いいねAPI                                             | 4  | 11                     | [issue](./build-17/issue.md) | —  |
| 18      | グローバルフィード・タイムラインAPI                   | 4  | 9, 16, 17              | [issue](./build-18/issue.md) | —  |
| 19      | ユーザーページAPI                                     | 4  | 18                     | [issue](./build-19/issue.md) | —  |
| 20      | 検索クエリ解析ライブラリ                              | 3  | 1                      | [issue](./build-20/issue.md) | —  |
| 21      | 検索API（pg_bigm・負荷保護）                          | 4  | 18, 20                 | [issue](./build-21/issue.md) | —  |
| 22      | Web: トップ・グローバルフィード・タイムライン         | 4  | 7, 18                  | [issue](./build-22/issue.md) | —  |
| 23      | Web: 投稿作成・単体ページ                             | 4  | 16, 17, 22             | [issue](./build-23/issue.md) | —  |
| 24      | Web: ユーザーページとフォロー操作                     | 4  | 9, 19, 22, 23          | [issue](./build-24/issue.md) | —  |
| 25      | Web: 公開範囲設定・フォロー申請一覧                   | 4  | 12, 13, 22             | [issue](./build-25/issue.md) | —  |
| 26      | Web: フォロワー一覧                                   | 3  | 14, 22                 | [issue](./build-26/issue.md) | —  |
| 27      | Web: 検索画面                                         | 3  | 21, 22                 | [issue](./build-27/issue.md) | —  |
| 28      | infra: CDK基盤・ネットワーク・RDS                     | 5  | 1                      | [issue](./build-28/issue.md) | —  |
| 29      | infra: S3・SES・ECR・Secrets                          | 5  | 28                     | [issue](./build-29/issue.md) | —  |
| 30      | infra: ECS Fargate・ALB・オートスケール・本番イメージ | 4  | 7, 29                  | [issue](./build-30/issue.md) | —  |
| 31      | infra: デプロイパイプラインと監視                     | 4  | 30                     | [issue](./build-31/issue.md) | —  |
| 32      | 負荷試験基盤とシードデータ                            | 5  | 22, 23, 24, 25, 26, 27 | [issue](./build-32/issue.md) | —  |
| 33      | 性能・費用・画像処理の検証とREADME                    | 5  | 31, 32                 | [issue](./build-33/issue.md) | —  |

## 依存グラフ

```mermaid
graph TD
  b01["build-01: 開発基盤（モノレポ・compose・CI・テスト基盤）"]
  b02["build-02: DB基盤・API共通基盤・レート制限"]
  b03["build-03: ユーザー登録とメールアドレス確認API"]
  b04["build-04: ログイン・ログアウトAPI"]
  b05["build-05: 確認メール再送API"]
  b06["build-06: パスワードリセットAPI"]
  b07["build-07: Web基盤と登録・ログイン画面"]
  b08["build-08: Web: メール確認・パスワードリセット画面"]
  b09["build-09: フォロー・フォロー解除API"]
  b10["build-10: 投稿の作成・削除API（本文のみ）"]
  b11["build-11: 投稿単体取得と閲覧権限の共通判定"]
  b12["build-12: 公開範囲設定API"]
  b13["build-13: フォロー申請の一覧・承認・拒否API"]
  b14["build-14: フォロワー一覧・フォロワーを外すAPI"]
  b15["build-15: 画像の形式検証・メタデータ除去ライブラリ"]
  b16["build-16: 画像の保存・アップロード・認可配信API"]
  b17["build-17: いいねAPI"]
  b18["build-18: グローバルフィード・タイムラインAPI"]
  b19["build-19: ユーザーページAPI"]
  b20["build-20: 検索クエリ解析ライブラリ"]
  b21["build-21: 検索API（pg_bigm・負荷保護）"]
  b22["build-22: Web: トップ・グローバルフィード・タイムライン"]
  b23["build-23: Web: 投稿作成・単体ページ"]
  b24["build-24: Web: ユーザーページとフォロー操作"]
  b25["build-25: Web: 公開範囲設定・フォロー申請一覧"]
  b26["build-26: Web: フォロワー一覧"]
  b27["build-27: Web: 検索画面"]
  b28["build-28: infra: CDK基盤・ネットワーク・RDS"]
  b29["build-29: infra: S3・SES・ECR・Secrets"]
  b30["build-30: infra: ECS Fargate・ALB・オートスケール・本番イメージ"]
  b31["build-31: infra: デプロイパイプラインと監視"]
  b32["build-32: 負荷試験基盤とシードデータ"]
  b33["build-33: 性能・費用・画像処理の検証とREADME"]
  b01 -.-> b02
  b02 -.-> b03
  b03 -.-> b04
  b03 -.-> b05
  b04 -.-> b06
  b04 -.-> b07
  b05 -.-> b07
  b06 -.-> b08
  b07 -.-> b08
  b04 -.-> b09
  b04 -.-> b10
  b09 -.-> b11
  b10 -.-> b11
  b11 -.-> b12
  b11 -.-> b13
  b13 -.-> b14
  b01 -.-> b15
  b11 -.-> b16
  b15 -.-> b16
  b11 -.-> b17
  b16 -.-> b18
  b17 -.-> b18
  b18 -.-> b19
  b01 -.-> b20
  b18 -.-> b21
  b20 -.-> b21
  b07 -.-> b22
  b18 -.-> b22
  b22 -.-> b23
  b19 -.-> b24
  b23 -.-> b24
  b12 -.-> b25
  b13 -.-> b25
  b22 -.-> b25
  b14 -.-> b26
  b22 -.-> b26
  b21 -.-> b27
  b22 -.-> b27
  b01 -.-> b28
  b28 -.-> b29
  b07 -.-> b30
  b29 -.-> b30
  b30 -.-> b31
  b24 -.-> b32
  b25 -.-> b32
  b26 -.-> b32
  b27 -.-> b32
  b31 -.-> b33
  b32 -.-> b33
  classDef merged fill:#fbefff,stroke:#8250df,color:#6639ba
  classDef ready fill:#dafbe1,stroke:#1a7f37,color:#116329
  classDef blocked fill:#f6f8fa,stroke:#8c959f,color:#57606a
  class b01 ready
  class b02,b03,b04,b05,b06,b07,b08,b09,b10,b11,b12,b13,b14,b15,b16,b17,b18,b19,b20,b21,b22,b23,b24,b25,b26,b27,b28,b29,b30,b31,b32,b33 blocked
  linkStyle 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48 stroke:#8c959f,stroke-dasharray:5 5
```

紫: 完了（merged） / 緑: 着手可能（ready） / グレー: 待機中（blocked）。点線は先行ビルドが未完了の依存
