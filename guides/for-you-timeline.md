# For You タイムラインの処理フロー

参考: `home-mixer/README.md`, リポジトリ直下 `README.md`

主なステージ

1) 候補生成（Candidate Generation）
   - In-Network: 検索インデックス Earlybird（`src/java/com/twitter/search`）
   - Out-of-Network: Tweet Mixer（`tweet-mixer/server/...`）、GraphJet 系（UTEG/UTG 等）、FRS
2) 特徴量水和（Feature Hydration）
   - ランキング用に ~6000 の特徴量を収集（ユーザー/ツイート/関係/履歴/埋め込み 等）
3) スコアリング&ランキング（Scoring & Ranking）
   - Light Ranker（Earlybird 側や軽量段階）
   - Heavy Ranker（外部レポ TwHIN/recap 参照）
4) フィルタ/ヒューリスティクス
   - 著者多様性、In/Out バランス、Feedback fatigue、重複/既視除去、Visibility フィルタ等
5) ミキシング（Mixing）
   - 広告、Who-To-Follow、プロンプト等を統合
6) プロダクト機能/サービング
   - 会話モジュール、ソーシャルコンテキスト、ページネーション/カーソル、観測性/ロギング など

Product Mixer 上のパイプライン構成（抜粋）

- Product Pipeline: For You / Following / Lists などの入り口
- Mixer Pipeline: 異種候補（ツイート/広告/ユーザー等）を結合
- Recommendation Pipeline: 同種候補を特徴水和しスコア/ランク
- Candidate Pipeline: 候補の取得・軽フィルタ・装飾・特徴水和

Home Mixer における代表的なクラス（抜粋名）

- ForYouProductPipelineConfig
  - ForYouScoredTweetsMixerPipelineConfig（ツイート/広告/Who-To-Follow を統合）
    - ForYouScoredTweetsCandidatePipelineConfig → ScoredTweetsRecommendationPipelineConfig
      - InNetwork（Earlybird）, TweetMixer, UTEG, FRS 由来の候補を集約
      - ScoredTweetsScoringPipelineConfig にてスコアリング
  - フォールバック逆順パイプライン（会話祖先など）/ Ads / Who-To-Follow パイプライン

注意点

- Earlybird は For You/Following いずれでも In-Network の主力候補ソース
- Out-of-Network は Tweet Mixer/GraphJet/FRS を状況やスイッチで組み合わせ利用

