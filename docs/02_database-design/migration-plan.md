# マイグレーション計画

[table-definitions.md](table-definitions.md) のテーブルを、どの順番・どの内容でマイグレーションとして作るかの計画です。
実装は Week 22 から行います。

## 作る順番の考え方

外部キーは「参照先のテーブル」が先にないと作れないため、**参照されるテーブルから順に** 作る。

1. 何も参照しないテーブル（users・マスタ）
2. 1 を参照するテーブル（recruitments・posts など）
3. 2 を参照するテーブル（recruitment_user）

## マイグレーション一覧

| 順番 | マイグレーション | 作るテーブル | 参照するテーブル | 優先度 | 作る時期 |
|------|------------------|-------------|-----------------|--------|---------|
| 0 | Breeze に含まれるもの | users / password_reset_tokens / sessions / cache / jobs | - | Must | Week 22 |
| 1 | add_profile_columns_to_users_table | users にカラム追加 | - | Must | Week 22 |
| 2 | create_categories_table | categories | - | Must | Week 22 |
| 3 | create_works_table | works | - | Must | Week 22 |
| 4 | create_prefectures_table | prefectures | - | Must | Week 22 |
| 5 | create_recruitments_table | recruitments | users, works, categories, prefectures | Must | Week 22 |
| 6 | create_recruitment_user_table | recruitment_user | recruitments, users | Must | Week 22〜23 |
| 7 | create_user_work_table | user_work | users, works | Should | Week 23 |
| 8 | create_posts_table | posts | users, categories | Should | Week 23 |
| 9 | create_follows_table | follows | users | Could | Week 24（余裕があれば） |
| 10 | create_reports_table | reports | users | Could | Week 24（余裕があれば） |

※ 0 は Laravel・Breeze が最初から用意するマイグレーション。cache・jobs などのテーブルも一緒に作られる

## 各マイグレーションの内容

### 1. add_profile_columns_to_users_table

```php
Schema::table('users', function (Blueprint $table) {
    $table->date('birthday')->after('password');
    $table->string('oshi_character', 50)->nullable()->after('birthday');
    $table->text('bio')->nullable()->after('oshi_character');
});
```

- Breeze の users マイグレーションは直接書き換えず、カラム追加用のマイグレーションを別に作る
- down() では、追加した3カラムを削除する

### 2〜4. マスタ（categories・works・prefectures）

```php
// categories（works は文字数を 100 にする）
Schema::create('categories', function (Blueprint $table) {
    $table->id();
    $table->string('name', 50)->unique();
    $table->timestamps();
});

// prefectures（id は JIS コードを入れるため自動採番にしない）
Schema::create('prefectures', function (Blueprint $table) {
    $table->unsignedBigInteger('id')->primary();
    $table->string('name', 10)->unique();
});
```

### 5. create_recruitments_table

```php
Schema::create('recruitments', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('work_id')->constrained()->restrictOnDelete();
    $table->foreignId('category_id')->constrained()->restrictOnDelete();
    $table->foreignId('prefecture_id')->constrained()->restrictOnDelete();
    $table->string('title', 50);
    $table->string('oshi_character', 50)->nullable();
    $table->boolean('allow_same_oshi')->default(true);
    $table->dateTime('event_at')->index();
    $table->unsignedTinyInteger('capacity');
    $table->text('description')->nullable();
    $table->string('image_path')->nullable();
    $table->timestamps();

    $table->index('created_at');
});
```

### 6. create_recruitment_user_table

```php
Schema::create('recruitment_user', function (Blueprint $table) {
    $table->id();
    $table->foreignId('recruitment_id')->constrained()->cascadeOnDelete();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->timestamps();

    $table->unique(['recruitment_id', 'user_id']);
});
```

### 7. create_user_work_table

```php
Schema::create('user_work', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('work_id')->constrained()->cascadeOnDelete();
    $table->timestamps();

    $table->unique(['user_id', 'work_id']);
});
```

### 8. create_posts_table

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('category_id')->constrained()->restrictOnDelete();
    $table->text('body');
    $table->string('image_path')->nullable();
    $table->timestamps();

    $table->index('created_at');
});
```

### 9. create_follows_table

```php
Schema::create('follows', function (Blueprint $table) {
    $table->id();
    // カラム名が user_id ではないため、参照先のテーブルを明示する
    $table->foreignId('follower_id')->constrained('users')->cascadeOnDelete();
    $table->foreignId('followed_id')->constrained('users')->cascadeOnDelete();
    $table->timestamps();

    $table->unique(['follower_id', 'followed_id']);
});
```

### 10. create_reports_table

```php
Schema::create('reports', function (Blueprint $table) {
    $table->id();
    $table->foreignId('reporter_id')->nullable()->constrained('users')->nullOnDelete();
    $table->foreignId('reported_id')->nullable()->constrained('users')->nullOnDelete();
    $table->string('reported_email')->index();
    $table->text('reason');
    $table->timestamps();
});
```

## Seeder の計画

| Seeder | 内容 | 本番でも実行するか |
|--------|------|-------------------|
| CategorySeeder | カテゴリ5件（コラボカフェ／写真撮影／ライブ・イベント／グッズ／その他） | する |
| PrefectureSeeder | 47都道府県（id は JIS コード） | する |
| WorkSeeder | 作品の初期データ（人気作品を20件程度。作品名は Week 21 で決める） | する |
| UserSeeder など | 動作確認用のテストユーザー・募集・投稿（Factory を使う） | しない（開発環境だけ） |

- マスタの Seeder は `updateOrCreate` で書き、何回実行しても重複しないようにする
- 本番では `php artisan db:seed --class=...` でマスタの Seeder だけ実行する

## 注意点

- 外部キーの順番を間違えると `migrate` が失敗するため、ファイル名の日時（マイグレーションの実行順）が上の表の順番になるように作る
- 本番にデプロイした後は、作成済みのマイグレーションを書き換えない。変更は新しいマイグレーションを追加して行う
- Could のテーブル（follows・reports）は、作らないことになった場合はマイグレーションも作らない
