# テーブル定義書

oshi-tomo（推し活おでかけ仲間募集アプリ）のテーブル定義書です。
要件定義書（[requirements.md](../requirements.md)）の機能要件をもとに作成しています。

- DB：MySQL 8.4
- 文字コード：utf8mb4
- `created_at` / `updated_at` は Laravel の `$table->timestamps()` で作成するため、NULL を許可する

## テーブル一覧

| No | テーブル名 | 日本語名 | 種類 | 関連する機能 | 優先度 |
|----|-----------|---------|------|-------------|--------|
| 1 | users | ユーザー | メイン（Breeze ＋カラム追加） | F-01, F-02, F-14 | Must |
| 2 | categories | カテゴリ | マスタ（Seeder） | F-03, F-05, F-10 | Must |
| 3 | works | 作品 | マスタ（Seeder） | F-03, F-05, F-09 | Must |
| 4 | prefectures | 都道府県 | マスタ（Seeder） | F-03, F-05 | Must |
| 5 | recruitments | 募集 | メイン | F-03〜F-06 | Must |
| 6 | recruitment_user | 行きたい！ | 中間（users × recruitments） | F-07, F-08 | Must |
| 7 | user_work | プロフィールの推し作品 | 中間（users × works） | F-09 | Should |
| 8 | posts | 投稿 | メイン | F-10 | Should |
| 9 | follows | フォロー | 中間（users × users） | F-15 | Could |
| 10 | reports | 通報 | メイン | F-11 | Could |
| 11 | official_infos | 公式情報 | マスタ（Seeder） | F-12 | Could |
| 12 | contacts | お問い合わせ | メイン | F-13 | Could |

※ 管理者画面は作らない（Won't）ため、公式情報は運営者が Seeder で登録し、お問い合わせは運営者が DB を直接確認する（Week 21 で決定）

### ON DELETE の方針

| 設定 | 使う場面 |
|------|---------|
| CASCADE | ユーザーが退会したら、その人のデータも削除する。中間テーブルのつながりも削除する |
| RESTRICT | マスタ（作品・カテゴリ・都道府県）が募集・投稿で使われている間は削除させない |
| SET NULL | 通報は、退会後も記録を残す |

---

## 1. users（ユーザー）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | ユーザーID（PK） |
| name | VARCHAR(255) | NO | - | ユーザー名 |
| email | VARCHAR(255) | NO | - | メールアドレス（他のユーザーには非表示） |
| email_verified_at | TIMESTAMP | YES | NULL | メール確認日時 |
| password | VARCHAR(255) | NO | - | パスワード（ハッシュ） |
| birthday | DATE | NO | - | 生年月日（他のユーザーには非表示） |
| oshi_character | VARCHAR(50) | YES | NULL | 推しキャラ（任意） |
| bio | TEXT | YES | NULL | 自己紹介・好きな活動（任意） |
| remember_token | VARCHAR(100) | YES | NULL | 「ログインしたままにする」用 |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- UNIQUE KEY (email)

【制約】
- email：UNIQUE 制約
- password：8文字以上（バリデーション）
- birthday：登録日時点で18歳以上（バリデーション）
- name・email・password・remember_token・email_verified_at は Breeze が作成する。birthday・oshi_character・bio を追加する

---

## 2. categories（カテゴリ）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | カテゴリID（PK） |
| name | VARCHAR(50) | NO | - | カテゴリ名 |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- UNIQUE KEY (name)

【初期データ（Seeder）】
- コラボカフェ／写真撮影／ライブ・イベント／グッズ／その他

---

## 3. works（作品）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | 作品ID（PK） |
| name | VARCHAR(100) | NO | - | 作品名 |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- UNIQUE KEY (name)

【備考】
- 推しは作品単位で登録する（表記ゆれを防ぐため、自由入力にしない）
- 初期データは Seeder で登録する。載っていない作品の人は「その他」を選び、お問い合わせで追加を依頼する（US-15）

---

## 4. prefectures（都道府県）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | - | 都道府県ID（PK）。JIS の都道府県コード（1=北海道 〜 47=沖縄県） |
| name | VARCHAR(10) | NO | - | 都道府県名 |

【インデックス】
- PRIMARY KEY (id)
- UNIQUE KEY (name)

【備考】
- 47都道府県を Seeder で登録する
- 追加・変更がないため、created_at / updated_at は持たない（モデルに `$timestamps = false`）

---

## 5. recruitments（募集）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | 募集ID（PK） |
| user_id | BIGINT UNSIGNED | NO | - | 募集した人（FK） |
| work_id | BIGINT UNSIGNED | NO | - | 作品（FK） |
| category_id | BIGINT UNSIGNED | NO | - | カテゴリ（FK） |
| prefecture_id | BIGINT UNSIGNED | NO | - | エリア（FK） |
| title | VARCHAR(50) | NO | - | タイトル |
| oshi_character | VARCHAR(50) | YES | NULL | 推しキャラ（任意） |
| allow_same_oshi | BOOLEAN | NO | true | 同担 OK＝true／同担拒否＝false |
| event_at | DATETIME | NO | - | おでかけの日時 |
| capacity | TINYINT UNSIGNED | NO | - | 募集人数 |
| description | TEXT | YES | NULL | 説明（任意） |
| image_path | VARCHAR(255) | YES | NULL | 写真の保存パス（任意） |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- INDEX (created_at)：一覧を新着順に並べるため
- INDEX (event_at)：終わった募集を一覧から除くため
- user_id・work_id・category_id・prefecture_id は外部キーを付けると自動で作成される

【外部キー】
- user_id REFERENCES users(id) ON DELETE CASCADE
- work_id REFERENCES works(id) ON DELETE RESTRICT
- category_id REFERENCES categories(id) ON DELETE RESTRICT
- prefecture_id REFERENCES prefectures(id) ON DELETE RESTRICT

【制約（バリデーション）】
- title：50文字以内
- event_at：作成時は今日以降
- capacity：1〜10
- description：1000文字以内
- image_path：画像ファイル（1枚）

【備考】
- トップの募集一覧には、日時が過ぎた募集を表示しない（`event_at >= 現在`）
- プロフィールの募集一覧には、過去の募集も表示する

---

## 6. recruitment_user（行きたい！）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | ID（PK） |
| recruitment_id | BIGINT UNSIGNED | NO | - | どの募集に（FK） |
| user_id | BIGINT UNSIGNED | NO | - | 誰が押した（FK） |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- UNIQUE KEY (recruitment_id, user_id)：同じ人が同じ募集に2回押せない

【外部キー】
- recruitment_id REFERENCES recruitments(id) ON DELETE CASCADE
- user_id REFERENCES users(id) ON DELETE CASCADE

【制約（バリデーション）】
- 自分の募集には押せない

---

## 7. user_work（プロフィールの推し作品）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | ID（PK） |
| user_id | BIGINT UNSIGNED | NO | - | ユーザー（FK） |
| work_id | BIGINT UNSIGNED | NO | - | 推しの作品（FK） |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- UNIQUE KEY (user_id, work_id)：同じ作品を2回登録できない

【外部キー】
- user_id REFERENCES users(id) ON DELETE CASCADE
- work_id REFERENCES works(id) ON DELETE CASCADE

---

## 8. posts（投稿）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | 投稿ID（PK） |
| user_id | BIGINT UNSIGNED | NO | - | 投稿した人（FK） |
| category_id | BIGINT UNSIGNED | NO | - | カテゴリ（FK） |
| body | TEXT | NO | - | 本文 |
| image_path | VARCHAR(255) | YES | NULL | 写真の保存パス（任意） |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- INDEX (created_at)：一覧を新着順に並べるため

【外部キー】
- user_id REFERENCES users(id) ON DELETE CASCADE
- category_id REFERENCES categories(id) ON DELETE RESTRICT

【制約（バリデーション）】
- body：500文字以内
- image_path：画像ファイル（1枚）

---

## 9. follows（フォロー）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | ID（PK） |
| follower_id | BIGINT UNSIGNED | NO | - | フォローする人（FK → users） |
| followed_id | BIGINT UNSIGNED | NO | - | フォローされる人（FK → users） |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- UNIQUE KEY (follower_id, followed_id)：同じ人を2回フォローできない

【外部キー】
- follower_id REFERENCES users(id) ON DELETE CASCADE
- followed_id REFERENCES users(id) ON DELETE CASCADE

【制約（バリデーション）】
- 自分自身はフォローできない

【備考】
- 「友達」（相互フォロー）は保存せず、2方向の行があるかどうかで判定する
  - 自分→相手 と 相手→自分 の両方がある：友達
  - 自分→相手 だけある：フォロー中
  - 相手→自分 だけある：フォローされています

---

## 10. reports（通報）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | ID（PK） |
| reporter_id | BIGINT UNSIGNED | YES | NULL | 通報した人（FK → users）。退会したら NULL |
| reported_id | BIGINT UNSIGNED | YES | NULL | 通報された人（FK → users）。退会したら NULL |
| reported_email | VARCHAR(255) | NO | - | 通報時点の、通報された人のメールアドレス |
| reason | TEXT | NO | - | 通報の理由 |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- INDEX (reported_email)：再登録した人の過去の通報を探すため

【外部キー】
- reporter_id REFERENCES users(id) ON DELETE SET NULL
- reported_id REFERENCES users(id) ON DELETE SET NULL

【制約（バリデーション）】
- 自分自身は通報できない

【備考】
- 退会して同じメールアドレスで再登録しても、過去の通報が分かるように、通報の記録は退会後も残す
- 退会後も通報の記録（メールアドレスを含む）を残すことを、利用規約・プライバシーポリシーに書く
- 管理者画面は作らない（Won't）ため、通報は開発者が DB を直接確認する

---

## 11. official_infos（公式情報）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | ID（PK） |
| work_id | BIGINT UNSIGNED | NO | - | どの作品の情報か（FK → works） |
| title | VARCHAR(100) | NO | - | 情報のタイトル（例：「〇〇 コラボカフェ 渋谷」） |
| url | VARCHAR(255) | NO | - | 公式サイトの URL |
| start_date | DATE | NO | - | 開催の開始日 |
| end_date | DATE | NO | - | 開催の終了日 |
| created_at | TIMESTAMP | YES | NULL | 作成日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- INDEX (end_date)：終わった情報を除いて一覧を出すため

【外部キー】
- work_id REFERENCES works(id) ON DELETE RESTRICT

【備考】
- 管理者画面は作らない（Won't）ため、運営者が OfficialInfoSeeder にデータを書き、`php artisan db:seed --class=OfficialInfoSeeder` で登録する
- 一覧には、終了日が今日以降の情報だけを、開始日が近い順に表示する
- 情報の詳細は公式サイトで見てもらう（アプリには詳細ページを作らない）

---

## 12. contacts（お問い合わせ）

| カラム名 | 型 | NULL | デフォルト | 説明 |
|----------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO | ID（PK） |
| user_id | BIGINT UNSIGNED | NO | - | 送った人（FK → users） |
| work_name | VARCHAR(100) | NO | - | 載せてほしい作品名 |
| body | TEXT | YES | NULL | 内容（任意、1000文字以内） |
| status | VARCHAR(20) | NO | '未対応' | 対応状況（未対応／対応済み） |
| created_at | TIMESTAMP | YES | NULL | 送った日時 |
| updated_at | TIMESTAMP | YES | NULL | 更新日時 |

【インデックス】
- PRIMARY KEY (id)
- INDEX (status)：未対応のお問い合わせを探すため

【外部キー】
- user_id REFERENCES users(id) ON DELETE CASCADE

【制約（バリデーション）】
- work_name：必須、100文字以内
- body：1000文字以内
- 連続送信を防ぐため、送信のルートに throttle（1分に3回まで）を付ける

【備考】
- 管理者画面は作らない（Won't）ため、運営者は `php artisan tinker` で未対応のお問い合わせを確認する（例：`Contact::where('status', '未対応')->get()`）
- 作品を追加したら、WorkSeeder に書き足して実行し、tinker で status を「対応済み」にする
