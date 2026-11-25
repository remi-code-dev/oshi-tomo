# oshi-tomo 設計書（初版）

推し活おでかけ仲間募集アプリ「oshi-tomo」の設計書です。
[要件定義書](requirements.md) で決めた「何を作るか」を、「どう作るか」に落とし込んだものです。

- 版：初版（Week 19）。Week 21 で見直して確定する
- 作成者：remi-code-dev

## システムの概要

| 項目 | 内容 |
|------|------|
| 目的 | 推しや好きな活動が合う仲間を、安心して見つけられるようにする |
| 主な機能 | 募集の投稿・検索、「行きたい！」（Ajax）、プロフィール、推し活の投稿 |
| 形式 | Laravel の Blade で作る Web アプリ（Breeze のログイン、routes/web.php のルート） |
| 主な利用環境 | スマートフォンのブラウザ（PC では画面幅を保ったまま中央に表示） |
| 開発環境 | Docker（nginx・app・mysql コンテナ） |
| 本番環境 | AWS（EC2 + Docker + RDS、GitHub Actions でデプロイ） |

## 設計書の一覧

| No | 設計書 | ファイル | 内容 |
|----|--------|---------|------|
| - | 要件定義書 | [requirements.md](requirements.md) | 何を作るか（Week 18、設計に合わせて 1.2 に改訂） |
| 01 | 画面設計 | [01_screen-design/figma.md](01_screen-design/figma.md) | Figma のワイヤーフレーム（5画面）、画面遷移図 [screen-transition.png](01_screen-design/screen-transition.png) |
| 02 | DB設計 | [02_database-design/table-definitions.md](02_database-design/table-definitions.md) | テーブル定義（12テーブル） |
| | | [02_database-design/er-diagram.md](02_database-design/er-diagram.md) | ER図（Mermaid）。画像版 [er-diagram.png](02_database-design/er-diagram.png) |
| | | [02_database-design/migration-plan.md](02_database-design/migration-plan.md) | マイグレーションを作る順番と内容、Seeder |
| 03 | ルーティング設計 | [03_routing-design/routing.md](03_routing-design/routing.md) | 画面と URL・HTTP メソッドの一覧、Policy、Ajax の JSON |
| 04 | 見積もり | [04_estimation/workload.md](04_estimation/workload.md) | 機能別の工数（ストーリーポイント法） |
| | | [04_estimation/schedule.md](04_estimation/schedule.md) | 全体と日別のスケジュール |
| 05 | インフラ構成図 | [05_infrastructure/infrastructure.md](05_infrastructure/infrastructure.md) | 本番（AWS）と開発（Docker）の構成図 |
| - | 練習課題 | [practice/](practice/) | Week 18・19 の練習課題 |

## 設計で決めた主なこと

| テーマ | 決めたこと | 理由 |
|--------|-----------|------|
| ログイン | 見るだけならログイン不要。作成・「行きたい！」などの操作はログインが必要 | 登録する前に、どんなアプリか分かるようにするため |
| 推しの持ち方 | 作品はマスタ（works）から選ぶ。キャラクターは自由入力 | 作品名の表記ゆれをなくし、検索を正確にするため |
| 同担拒否 | 募集に「同担OK／同担拒否」を持たせる | 作品単位では、同じキャラ推しかどうかが分からないため |
| 過去の募集 | トップの一覧には出さず、プロフィールには出す | 一覧を探しやすくしつつ、相手の活動が分かるようにするため |
| 友達の判定 | 保存せず、フォローの2方向の行から判定する | 片方がフォロー解除したときの更新漏れを防ぐため（正規化） |
| 通報の記録 | 退会後もメールアドレスとともに残す | 退会して再登録する迷惑ユーザーに対応するため |
| 公式情報・お問い合わせ | 公式情報は Seeder で登録、お問い合わせは DB に保存して tinker で確認する | 管理者画面を作らず（Won't）、習った技術だけで作れるため |
| デプロイ先 | A：AWS を使い続ける（ブログアプリと同じ EC2・RDS） | 構築済みの環境を使え、データが消えないため |
| 実装の進め方 | 12/9 の最初の提出までに中心の機能、その後に残り | 使える時間（48時間）が全機能の工数より少ないため |

## 制約・注意点

- 実装期間は 2026/12/1〜12/9（最初の提出）。11/28〜11/30 に前倒しで環境構築を行う
- 本番環境へのデプロイは、最初の提出の後に行う
- 会員登録・投稿・通報の画面は、Week 21 の見直しで Figma に追加するかを決める
