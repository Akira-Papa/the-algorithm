## Tweetypie（コアTweetサービス）

パス: `tweetypie/server/README.md`

役割

- クライアント/サービスからのツイート読み書きを処理
- 読み取りでは Manhattan/Twemcache から取得し、バックエンド群からハイドレーション
- 書き込みでは各ストア（Manhattan 等）への更新を調整

主なレイヤ

- backends: 下位 Thrift サービスのラッパ
- repository: backend を隠蔽した取得用インターフェース
- hydrator: Tweet の補助情報の取得/合成
- handler: エンドポイント処理（例: get_tweets / post_tweet）

代表的な処理経路

- Read: GetTweetsHandler → Manhattan/Caching Repo → Hydration Pipeline（`TweetHydration.scala`）
- Write: PostTweet → Builder/Validation → WritePathHydration → 各 Store へ反映

