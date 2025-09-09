# Home Mixer（ホーム構築の中核）

パス: `home-mixer/README.md`

役割

- For You / Following / Lists 等のホーム系タイムラインを構築して返すメインサービス
- `product-mixer` フレームワーク上でパイプライン（Product/Mixer/Recommendation/Candidate）を編成

主なステージと考慮点

- 候補生成: Earlybird/UTEG/FRS/Tweet Mixer など複数ソース
- 特徴量水和: ランク用に大量の特徴を集約
- ランキング: Light/Heavy の二段（状況により切替/冗長化）
- フィルタ: 多様性/バランス/既視/可視性/ヘルス/重複など
- ミキシング: ツイート/広告/WTF/プロンプト等の異種統合
- サービング: 会話モジュール/ソーシャルコンテキスト/ページング/ログ

代表パイプライン（抜粋）

- ForYouProductPipelineConfig → ForYouScoredTweetsMixerPipelineConfig
  - InNetwork（Earlybird）, TweetMixer, UTEG, FRS 等の CandidatePipeline を束ね、ScoringPipeline でランキング
- FollowingProductPipelineConfig
  - FollowingEarlybirdCandidatePipeline + 会話祖先 + Ads + WTF

ポイント

- フィードの信頼性/安全性/多様性のための後段フィルタが豊富
- 失敗時フォールバック（逆順/会話祖先等）を用意

