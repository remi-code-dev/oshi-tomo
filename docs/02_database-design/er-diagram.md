# ER図

oshi-tomo の ER図です。各テーブルのカラムの詳細は [table-definitions.md](table-definitions.md) を参照してください。
画像版は [er-diagram.png](er-diagram.png) です（この Mermaid を Mermaid Live Editor で書き出したもの）。

## 記号の読み方（カラスの足記法）

| 記号 | 意味 |
|------|------|
| `\|\|` | ちょうど1つ |
| `o\|` | 0か1つ |
| `o{` | 0以上 |

例：`users ||--o{ recruitments` は「1人のユーザーは、0個以上の募集を持つ」

## ER図

```mermaid
erDiagram
    users ||--o{ recruitments : "募集する"
    categories ||--o{ recruitments : "分類する"
    works ||--o{ recruitments : "対象の作品"
    prefectures ||--o{ recruitments : "エリア"

    users ||--o{ recruitment_user : "行きたい！を押す"
    recruitments ||--o{ recruitment_user : "行きたい！を押される"

    users ||--o{ user_work : "推しを登録する"
    works ||--o{ user_work : "推しにされる"

    users ||--o{ posts : "投稿する"
    categories ||--o{ posts : "分類する"

    users ||--o{ follows : "フォローする（follower_id）"
    users ||--o{ follows : "フォローされる（followed_id）"

    users |o--o{ reports : "通報する（reporter_id）"
    users |o--o{ reports : "通報される（reported_id）"

    works ||--o{ official_infos : "公式情報"
    users ||--o{ contacts : "お問い合わせする"

    users {
        bigint id PK
        varchar name
        varchar email UK
        timestamp email_verified_at
        varchar password
        date birthday
        varchar oshi_character
        text bio
        varchar remember_token
        timestamp created_at
        timestamp updated_at
    }

    categories {
        bigint id PK
        varchar name UK
        timestamp created_at
        timestamp updated_at
    }

    works {
        bigint id PK
        varchar name UK
        timestamp created_at
        timestamp updated_at
    }

    prefectures {
        bigint id PK
        varchar name UK
    }

    recruitments {
        bigint id PK
        bigint user_id FK
        bigint work_id FK
        bigint category_id FK
        bigint prefecture_id FK
        varchar title
        varchar oshi_character
        boolean allow_same_oshi
        datetime event_at
        tinyint capacity
        text description
        varchar image_path
        timestamp created_at
        timestamp updated_at
    }

    recruitment_user {
        bigint id PK
        bigint recruitment_id FK
        bigint user_id FK
        timestamp created_at
        timestamp updated_at
    }

    user_work {
        bigint id PK
        bigint user_id FK
        bigint work_id FK
        timestamp created_at
        timestamp updated_at
    }

    posts {
        bigint id PK
        bigint user_id FK
        bigint category_id FK
        text body
        varchar image_path
        timestamp created_at
        timestamp updated_at
    }

    follows {
        bigint id PK
        bigint follower_id FK
        bigint followed_id FK
        timestamp created_at
        timestamp updated_at
    }

    reports {
        bigint id PK
        bigint reporter_id FK
        bigint reported_id FK
        varchar reported_email
        text reason
        timestamp created_at
        timestamp updated_at
    }

    official_infos {
        bigint id PK
        bigint work_id FK
        varchar title
        varchar url
        date start_date
        date end_date
        timestamp created_at
        timestamp updated_at
    }

    contacts {
        bigint id PK
        bigint user_id FK
        varchar work_name
        text body
        varchar status
        timestamp created_at
        timestamp updated_at
    }
```

## 多対多の関係（中間テーブル）

| 関係 | 中間テーブル |
|------|-------------|
| users ⇔ recruitments（行きたい！） | recruitment_user |
| users ⇔ works（推しの作品） | user_work |
| users ⇔ users（フォロー） | follows |
