# 練習課題3：ToDo アプリのルーティング設計書

題材：ToDo アプリ（Blade の Web アプリ、Breeze のログイン）

## 想定する機能

- 会員登録・ログインしたユーザーが、自分の ToDo を作成・編集・削除できる
- ToDo の項目：タイトル（必須）、メモ（任意）、期限（任意）、完了かどうか
- 一覧では「未完了」「完了」で絞り込める
- 一覧のチェックボックスで、ページを再読み込みせずに完了・未完了を切り替えられる（Ajax）
- 他の人の ToDo は、見ることも操作することもできない

## 方針

- ToDo は個人のデータなので、**すべてのルートでログインが必要**
- ToDo の CRUD は `Route::resource()` の7つのルート（RESTful）で作る
- 完了・未完了の切り替えは「完了かどうか」だけを変える **一部の更新** のため、PATCH メソッドで別のルートを作る
- `/` にアクセスしたら、ToDo の一覧（`/todos`）へリダイレクトする

## ルート一覧

| 画面・操作 | メソッド | URL | ルート名 | コントローラー@アクション | ログイン | 権限（Policy） |
|-----------|---------|-----|---------|--------------------------|---------|---------------|
| トップ | GET | / | - | （`/todos` へリダイレクト） | 必要 | - |
| ToDo 一覧（絞り込み） | GET | /todos?status=undone または done | todos.index | TodoController@index | 必要 | -（自分の ToDo だけ取得する） |
| ToDo 作成フォーム | GET | /todos/create | todos.create | TodoController@create | 必要 | - |
| ToDo の保存 | POST | /todos | todos.store | TodoController@store | 必要 | - |
| ToDo 詳細 | GET | /todos/{todo} | todos.show | TodoController@show | 必要 | 自分の ToDo だけ |
| ToDo 編集フォーム | GET | /todos/{todo}/edit | todos.edit | TodoController@edit | 必要 | 自分の ToDo だけ |
| ToDo の更新 | PUT | /todos/{todo} | todos.update | TodoController@update | 必要 | 自分の ToDo だけ |
| ToDo の削除 | DELETE | /todos/{todo} | todos.destroy | TodoController@destroy | 必要 | 自分の ToDo だけ |
| 完了・未完了の切り替え（Ajax） | PATCH | /todos/{todo}/complete | todos.complete | TodoController@toggleComplete | 必要 | 自分の ToDo だけ |

※ ログイン・ログアウト・会員登録の画面は Breeze が作るルートを使う
※ 完了・未完了の切り替え（Ajax）は画面を返さず、JSON（完了かどうか）を返す

## 補足

### 権限（Policy）

| Policy | メソッド | ルール |
|--------|---------|--------|
| TodoPolicy | view / update / delete | ToDo の user_id がログイン中のユーザーと同じとき |

- 完了・未完了の切り替え（todos.complete）も、update と同じルールで守る
- 一覧（todos.index）は、`auth()->user()->todos()` のように、最初からログイン中のユーザーの ToDo だけを取得する

### Ajax で呼ぶルート（JSON を返す）

**完了・未完了の切り替え PATCH /todos/{todo}/complete**

```json
// レスポンス例（完了にしたとき）
{
  "completed": true
}
```

- 未完了なら完了に、完了なら未完了にする（トグル）
- 他の人の ToDo のときは 403（権限なし）を返す

### PUT と PATCH の使い分け

| メソッド | 使う場面 | このアプリでは |
|---------|---------|---------------|
| PUT | データ全体を置き換える | 編集フォームで、タイトル・メモ・期限をまとめて更新する（todos.update） |
| PATCH | データの一部だけを更新する | 「完了かどうか」だけを変える（todos.complete） |

## routes/web.php のイメージ

```php
Route::middleware('auth')->group(function () {
    Route::redirect('/', '/todos');

    // create より後に書くと /todos/create が show に吸い込まれるため、resource でまとめて定義する
    Route::resource('todos', TodoController::class);
    Route::patch('/todos/{todo}/complete', [TodoController::class, 'toggleComplete'])->name('todos.complete');
});

require __DIR__.'/auth.php';
```

## 確認したこと（教材のチェック項目）

| チェック項目 | 結果 |
|-------------|------|
| 機能がどれかのルートで実現できるか | 作成・編集・削除・一覧（絞り込み）・完了の切り替えのすべてにルートがある |
| 画面と GET のルートが1対1で対応しているか | 一覧（todos.index）・作成（todos.create）・詳細（todos.show）・編集（todos.edit）の4画面 |
| 他人のデータを編集・削除できてしまうルートがないか | {todo} を含むルートはすべて TodoPolicy で守る。一覧は自分の ToDo だけを取得する |
| Ajax で呼ぶルートが少なくとも1つあるか | todos.complete（完了・未完了の切り替え） |
