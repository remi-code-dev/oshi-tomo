# oshi-tomo

推し活のおでかけ仲間を安心して見つけるためのアプリです。
コラボカフェや写真撮影の募集を投稿し、推しや好きな活動が合う人とつながれます。

## 誰のためのアプリか

- **募集する人**：コラボカフェに行きたいけど、周りに同じ推しの友達がいない人
- **募集を探す人**：同じ推しの人と出かけたいけど、知らない人に DM を送るのが苦手な人

くわしくは [要件定義書のペルソナ](docs/requirements.md) を参照してください。

## 主な機能（MVP：最初の提出 12/9 まで）

| 機能 | 内容 | 教材の必須の機能 |
|------|------|-----------------|
| 会員登録・ログイン | 18歳以上の人だけ登録できる | ログイン・ログアウト（Breeze） |
| 募集の投稿・一覧・詳細・編集・削除 | おでかけ・グッズ交換の募集 | 主要なデータの CRUD |
| 自分の募集だけ編集・削除 | 他の人の募集は 403 | 権限管理（Policy） |
| キーワード検索 | 推しキャラ・タイトルで募集を探す | 検索または絞り込み |
| 写真アップロード | 募集に写真を1枚付ける | 画像アップロード |
| 「行きたい！」ボタン | ページを再読み込みせずに参加希望を伝える | Ajax の操作 |

MVP の後に、プロフィール・絞り込み・本番環境へのデプロイ、そのあと Should・Could の機能（投稿・フォローなど）を作ります。

## 使用技術

| 分類 | 技術 |
|------|------|
| バックエンド | PHP 8.2 / Laravel 12 / Laravel Breeze / MySQL 8.4 |
| フロントエンド | Blade / Tailwind CSS / Alpine.js |
| インフラ | Docker（開発）/ AWS：EC2 + Docker + RDS（本番） |
| 開発ツール | GitHub Actions / PHPUnit / PHPStan / Figma / Claude Code |

選んだ理由は [技術選定資料](docs/06_tech-stack.md) を参照してください。

## スケジュール

| 期間 | 内容 |
|------|------|
| 11/20〜11/27（Week 18〜21） | 要件定義・設計・スプリント計画 |
| 11/28〜12/9（Week 22〜24） | 実装（MVP） |
| 12/9 | 最初の提出 |
| 12/10〜 | 残りの機能・本番環境へのデプロイ |

## ドキュメント

| ドキュメント | 内容 |
|-------------|------|
| [要件定義書](docs/requirements.md) | ペルソナ・MVP・ユーザーストーリー・機能要件・非機能要件 |
| [設計書の概要](docs/00_overview.md) | 設計書の一覧と、設計で決めたこと |
| [画面設計](docs/01_screen-design/figma.md) | Figma のワイヤーフレーム（6画面）・画面遷移図 |
| [DB設計](docs/02_database-design/table-definitions.md) | テーブル定義（12テーブル）・[ER図](docs/02_database-design/er-diagram.md) |
| [ルーティング設計](docs/03_routing-design/routing.md) | 画面と URL・HTTP メソッドの一覧、Policy・Ajax のルート |
| [見積もり](docs/04_estimation/workload.md) | 機能別の工数・[スケジュール](docs/04_estimation/schedule.md) |
| [インフラ構成図](docs/05_infrastructure/infrastructure.md) | 本番（AWS）と開発（Docker）の構成 |
| [技術選定資料](docs/06_tech-stack.md) | 使用技術とデプロイ先の選定理由 |
| [スプリント計画](docs/sprint-plan.md) | 3週間のスプリントと日別の計画 |
| [見積書](docs/estimate.md) | 架空のクライアント向けの見積書 |

## リンク

- 画面設計（Figma）：[oshi-tomo 画面設計](https://www.figma.com/design/Tku4M3QKBPlf8cIJ8SJPN7/oshi-tomo-%E7%94%BB%E9%9D%A2%E8%A8%AD%E8%A8%88?node-id=14-602&t=ytfOgcBFLJ78UEjM-1)
- タスク管理（GitHub Projects）：[oshi-tomo のボード](https://github.com/users/remi-code-dev/projects/1)

## 開発環境の起動手順

（Week 22 の環境構築で追加する）
