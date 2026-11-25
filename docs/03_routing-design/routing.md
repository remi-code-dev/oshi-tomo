# ルーティング設計

oshi-tomo の画面・操作と、URL・HTTP メソッドの一覧です。Week 22 で書く `routes/web.php` とコントローラーの設計図になります。

## 方針

- Blade の Web アプリとして作る（`routes/web.php`）。ログイン・会員登録は Breeze のルートを使う
- **見るだけならログイン不要**。募集・投稿の作成、「行きたい！」、フォローなどの操作はログインが必要
- 募集と投稿は、それぞれ `Route::resource()` の形（RESTful）で作る
- トップのタブは、募集一覧（`/recruitments`）と投稿一覧（`/posts`）へのリンクで切り替える
- `/` にアクセスしたら、募集一覧（`/recruitments`）へリダイレクトする
- ログイン後の移動先は、Breeze の `/dashboard` ではなく募集一覧にする（`/dashboard` は使わない）

## ルート一覧

### トップ

| 画面・操作 | メソッド | URL | ルート名 | コントローラー@アクション | ログイン | 権限（Policy） | 優先度 |
|-----------|---------|-----|---------|--------------------------|---------|---------------|--------|
| トップ | GET | / | - | （`/recruitments` へリダイレクト） | 不要 | - | Must |

### 募集（F-03〜F-08）

| 画面・操作 | メソッド | URL | ルート名 | コントローラー@アクション | ログイン | 権限（Policy） | 優先度 |
|-----------|---------|-----|---------|--------------------------|---------|---------------|--------|
| 募集一覧（検索・絞り込み） | GET | /recruitments?keyword=...&work_id=...&category_id=...&prefecture_id=... | recruitments.index | RecruitmentController@index | 不要 | - | Must |
| 募集の作成フォーム | GET | /recruitments/create | recruitments.create | RecruitmentController@create | 必要 | - | Must |
| 募集の保存 | POST | /recruitments | recruitments.store | RecruitmentController@store | 必要 | - | Must |
| 募集の詳細 | GET | /recruitments/{recruitment} | recruitments.show | RecruitmentController@show | 不要 | - | Must |
| 募集の編集フォーム | GET | /recruitments/{recruitment}/edit | recruitments.edit | RecruitmentController@edit | 必要 | 自分の募集だけ | Must |
| 募集の更新 | PUT | /recruitments/{recruitment} | recruitments.update | RecruitmentController@update | 必要 | 自分の募集だけ | Must |
| 募集の削除 | DELETE | /recruitments/{recruitment} | recruitments.destroy | RecruitmentController@destroy | 必要 | 自分の募集だけ | Must |
| 「行きたい！」を押す・取り消す（Ajax） | POST | /recruitments/{recruitment}/want | recruitments.want | WantController@toggle | 必要 | 自分の募集以外 | Must |

### 投稿（F-10）

| 画面・操作 | メソッド | URL | ルート名 | コントローラー@アクション | ログイン | 権限（Policy） | 優先度 |
|-----------|---------|-----|---------|--------------------------|---------|---------------|--------|
| 投稿一覧（カテゴリで絞り込み） | GET | /posts?category_id=... | posts.index | PostController@index | 不要 | - | Should |
| 投稿の作成フォーム | GET | /posts/create | posts.create | PostController@create | 必要 | - | Should |
| 投稿の保存 | POST | /posts | posts.store | PostController@store | 必要 | - | Should |
| 投稿の詳細 | GET | /posts/{post} | posts.show | PostController@show | 不要 | - | Should |
| 投稿の編集フォーム | GET | /posts/{post}/edit | posts.edit | PostController@edit | 必要 | 自分の投稿だけ | Should |
| 投稿の更新 | PUT | /posts/{post} | posts.update | PostController@update | 必要 | 自分の投稿だけ | Should |
| 投稿の削除 | DELETE | /posts/{post} | posts.destroy | PostController@destroy | 必要 | 自分の投稿だけ | Should |

### プロフィール（F-02, F-09, F-14, F-15）

| 画面・操作 | メソッド | URL | ルート名 | コントローラー@アクション | ログイン | 権限（Policy） | 優先度 |
|-----------|---------|-----|---------|--------------------------|---------|---------------|--------|
| プロフィール（自分・他の人） | GET | /users/{user} | users.show | UserController@show | 不要 | - | Must |
| プロフィールの編集フォーム（推しの登録を含む） | GET | /profile | profile.edit | ProfileController@edit | 必要 | （自分のプロフィールだけ表示される） | Must |
| プロフィールの更新 | PATCH | /profile | profile.update | ProfileController@update | 必要 | （自分のプロフィールだけ更新される） | Must |
| パスワードの変更 | PUT | /password | password.update | PasswordController@update | 必要 | （自分だけ） | Must |
| 退会 | DELETE | /profile | profile.destroy | ProfileController@destroy | 必要 | （自分だけ） | Must |
| フォローする・解除する（Ajax） | POST | /users/{user}/follow | users.follow | FollowController@toggle | 必要 | 自分以外 | Could |

### 通報（F-11）

| 画面・操作 | メソッド | URL | ルート名 | コントローラー@アクション | ログイン | 権限（Policy） | 優先度 |
|-----------|---------|-----|---------|--------------------------|---------|---------------|--------|
| 通報フォーム | GET | /users/{user}/reports/create | users.reports.create | ReportController@create | 必要 | 自分以外 | Could |
| 通報の送信 | POST | /users/{user}/reports | users.reports.store | ReportController@store | 必要 | 自分以外 | Could |

### 会員登録・ログイン（F-01、Breeze）

| 画面・操作 | メソッド | URL | ルート名 | ログイン | 優先度 |
|-----------|---------|-----|---------|---------|--------|
| 会員登録フォーム（生年月日を追加） | GET | /register | register | 未ログインのみ | Must |
| 会員登録（18歳未満はエラー） | POST | /register | - | 未ログインのみ | Must |
| ログインフォーム | GET | /login | login | 未ログインのみ | Must |
| ログイン | POST | /login | - | 未ログインのみ | Must |
| ログアウト | POST | /logout | logout | 必要 | Must |
| パスワード再設定（メール送信・再設定フォーム） | GET / POST | /forgot-password、/reset-password/{token} など | password.request など | 未ログインのみ | Must |

### 公式情報（F-12）・お問い合わせ（F-13）

管理者画面は作らない（Won't）ため、公式情報の登録は Seeder、お問い合わせの確認は tinker で行う。アプリには、見る画面・送る画面だけを作る。

| 画面・操作 | メソッド | URL | ルート名 | コントローラー@アクション | ログイン | 権限（Policy） | 優先度 |
|-----------|---------|-----|---------|--------------------------|---------|---------------|--------|
| 公式情報の一覧 | GET | /official-infos | official-infos.index | OfficialInfoController@index | 不要 | - | Could |
| お問い合わせフォーム | GET | /contacts/create | contacts.create | ContactController@create | 必要 | - | Could |
| お問い合わせの送信 | POST | /contacts | contacts.store | ContactController@store | 必要（`throttle:3,1`：1分に3回まで） | - | Could |

- 公式情報は一覧だけ作る。各情報から公式サイトへリンクする（詳細ページは作らない）
- お問い合わせを送ったら、フォームへリダイレクトして「送信しました」と表示する

## 補足

### ログイン・権限の守り方

- ログインが必要なルートは `auth` ミドルウェアのグループに入れる。ログインしていない人がアクセスすると、ログイン画面へ移動する
- 「自分の募集だけ」などの権限は Policy で守る

| Policy | メソッド | ルール |
|--------|---------|--------|
| RecruitmentPolicy | update / delete | 募集の user_id がログイン中のユーザーと同じとき |
| RecruitmentPolicy | want | 募集の user_id がログイン中のユーザーと **違う** とき（自分の募集には「行きたい！」を押せない） |
| PostPolicy | update / delete | 投稿の user_id がログイン中のユーザーと同じとき |
| UserPolicy | follow / report | 相手がログイン中のユーザー自身ではないとき |

- プロフィールの編集（`/profile`）は URL に ID がなく、ログイン中のユーザー自身のデータだけを扱うため、他人のプロフィールは編集できない（Breeze の仕組み）

### Ajax で呼ぶルート（JSON を返す）

画面を返さず、JSON を返す。ページを再読み込みせずにボタンの表示と件数を切り替える。

**「行きたい！」 POST /recruitments/{recruitment}/want**

```json
// レスポンス例（押した状態になったとき）
{
  "wanted": true,
  "count": 3
}
```

- 押していない状態なら押した状態に、押した状態なら取り消す（トグル）
- 自分の募集のときは 403（権限なし）を返す
- ログインしていないときは 401 を返す。画面側ではボタンの代わりに「ログインして『行きたい！』を押す」リンクを表示する

**フォロー POST /users/{user}/follow**（Could）

```json
// レスポンス例（フォローした結果、相互フォローになったとき）
{
  "following": true,
  "relation": "friend"
}
```

- relation は `friend`（友達）／`following`（フォロー中）／`followed`（フォローされています）／`none`（関係なし）

### 画面に表示する内容のルール

| 画面 | ルール |
|------|--------|
| 募集一覧 | 日時が過ぎた募集は表示しない。新着順 |
| 募集の詳細 | 「行きたい！」を押した人の一覧（F-08）は、募集した本人にだけ表示する |
| プロフィール | その人の募集（過去の募集も含む）と投稿を表示する |
| プロフィール | 「あなたの年齢 ±○歳」（F-14）と相手との関係（F-15）は、ログインしているときだけ表示する |
| 募集の作成・編集 | 「待ち合わせ場所の詳細（駅名・店名）は書かないでください」と注意書きを表示する（誰でも見られるため） |

## routes/web.php のイメージ

```php
Route::redirect('/', '/recruitments');

// ログインが必要なルート
Route::middleware('auth')->group(function () {
    Route::resource('recruitments', RecruitmentController::class)->except(['index', 'show']);
    Route::post('/recruitments/{recruitment}/want', [WantController::class, 'toggle'])->name('recruitments.want');

    Route::resource('posts', PostController::class)->except(['index', 'show']);

    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');

    Route::post('/users/{user}/follow', [FollowController::class, 'toggle'])->name('users.follow');
    Route::resource('users.reports', ReportController::class)->only(['create', 'store']);
});

// ログイン不要のルート（create より後に書く。先に書くと /recruitments/create が show に吸い込まれるため）
Route::resource('recruitments', RecruitmentController::class)->only(['index', 'show']);
Route::resource('posts', PostController::class)->only(['index', 'show']);
Route::get('/users/{user}', [UserController::class, 'show'])->name('users.show');

require __DIR__.'/auth.php';
```

## 確認したこと（教材のチェック項目）

### 1. 機能要件・ユーザーストーリーが、どれかのルートで実現できるか

| 機能 | ユーザーストーリー | ルート |
|------|-------------------|--------|
| F-01 会員登録・ログイン | US-01, US-22 | register, login, logout（Breeze） |
| F-02 プロフィール | US-02, US-03 | users.show, profile.edit, profile.update |
| F-03 募集の CRUD | US-04, US-06, US-07, US-18 | recruitments.*（7つ） |
| F-04 自分の募集だけ編集・削除 | US-06 | recruitments.edit / update / destroy（RecruitmentPolicy） |
| F-05 募集の検索・絞り込み | US-08, US-09 | recruitments.index（パラメータ） |
| F-06 写真アップロード | US-05 | recruitments.store / update |
| F-07 「行きたい！」 | US-10 | recruitments.want（Ajax） |
| F-08 「行きたい！」を押した人の一覧 | US-11 | recruitments.show（募集した本人にだけ表示） |
| F-09 推しの登録 | US-13 | profile.edit, profile.update |
| F-10 投稿 | US-16, US-17 | posts.*（7つ） |
| F-11 通報 | US-12 | users.reports.create / store |
| F-12 公式情報 | US-14 | official-infos.index |
| F-13 お問い合わせ | US-15 | contacts.create / store |
| F-14 年齢の範囲表示 | US-19 | users.show（ログイン時だけ表示） |
| F-15 フォロー | US-20, US-21 | users.follow（Ajax）, users.show |

### 2. 画面設計（Figma）の画面と、GET のルートが1対1で対応しているか

| 画面（Figma で作る） | GET のルート |
|---------------------|-------------|
| トップ：募集一覧タブ | recruitments.index |
| トップ：投稿一覧タブ | posts.index |
| 募集の詳細 | recruitments.show |
| 募集の作成・編集 | recruitments.create / recruitments.edit（同じレイアウト） |
| 投稿の詳細 | posts.show |
| 投稿の作成・編集 | posts.create / posts.edit（同じレイアウト） |
| プロフィール | users.show |
| プロフィールの編集 | profile.edit |
| 会員登録 | register |
| ログイン | login |
| 通報フォーム（Could） | users.reports.create |

### 3. 他人のデータを編集・削除できてしまうルートがないか

- 募集・投稿の edit / update / destroy は Policy で守る（「権限（Policy）」の列に印あり）
- プロフィールの編集・退会は、ログイン中のユーザー自身のデータだけを扱う

### 4. Ajax で呼ぶルートが少なくとも1つあるか

- recruitments.want（「行きたい！」・Must）
- users.follow（フォロー・Could）
