# Tweet Mixer candidate_pipeline 詳解（ConfigFactory 別）

対象ディレクトリ: `tweet-mixer/server/src/main/scala/com/twitter/tweet_mixer/candidate_pipeline`

ここでは CandidatePipelineConfigFactory を中心に、パイプライン識別子、ゲート、Query 変換、CandidateSource、Feature 変換、Result 変換の流れを要約します（継続更新）。

- SimclustersProducerBasedCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `SimClustersProducerBased`
  - Gate: `SimclustersProducerBasedEnabled`
  - Query変換: `SANNQueryTransformer`（ProducerBasedClusterParamMap, FavBasedProducer, MinScore, MaxCandidates(Paramゲート)）で `SANNQuery` を構成。
  - CandidateSource: `SimClustersAnnCandidateSource`
  - Features: `TweetMixerCandidateFeatureTransformer` を適用
  - Result変換: `TweetCandidate(id = candidate.tweetId)`

- EarlybirdInNetworkCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `EarlybirdInNetwork`
  - Gates: `HaploliteTweetsBasedEnabled`（ParamGate）, `EmptySeqFeatureGate(HaploliteFeature)`
  - Query変換: `EarlybirdInNetworkQueryTransformer(identifier, Some(ClientId.name + ".prod"))` により `InNetworkRequest` を構成
  - CandidateSource: `EarlybirdRealtimeCGTweetCandidateSource`
  - Features: `EarlybirdInNetworkResponseFeatureTransformer`
  - Result変換: Earlybird `ThriftSearchResult.id` を `TweetCandidate` に射影

- UTGTweetBasedCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `UTGTweetBased`
  - Gates: `UTGTweetBasedEnabled`（ParamGate）, `MaxFollowersGate`
  - Query変換: `UTGTweetBasedQueryTransformer`（signalsFn, isExpansion=false, `TweetBasedMinScoreParam`, `TweetBasedDegreeExponentParam`）→ `UTGTweetBasedRequest`
  - Query特徴水和: Paramゲートで `OutlierDeepRetrievalEmbeddingQueryFeatureHydrator` → Phase2 で `UTGOutlierSignalsQueryFeatureHydrator`
  - CandidateSource: `UserTweetGraphTweetBasedCandidateSource`
  - Features: `TweetMixerCandidateFeatureTransformer`
  - Result変換: `TweetCandidate(id = candidate.tweetId)`

- UVGTweetBasedCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `UVGTweetBased`
  - Gates: `UVGTweetBasedEnabled`（ParamGate）, `MaxFollowersGate`
  - Query変換: `UVGTweetBasedQueryTransformer`（signalsFn, isExpansion=false, `TweetBasedMinScoreParam`, `TweetBasedDegreeExponentParam`）→ `UVGTweetBasedRequest`
  - Query特徴水和: Paramゲートで `OutlierDeepRetrievalEmbeddingQueryFeatureHydrator` → Phase2 で `UVGOutlierSignalsQueryFeatureHydrator`
  - CandidateSource: `UserVideoGraphTweetBasedCandidateSource`
  - Features: `TweetMixerCandidateFeatureTransformer`
  - Result変換: `TweetCandidate(id = candidate.tweetId)`

- DeepRetrievalTweetTweetSimilarityCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `DeepRetrievalTweetTweetSimilarity`
  - Gate: `DeepRetrievalTweetTweetANNEnabled`
  - Query変換: signalsFn から `DRMultipleANNQuery` を構成（`DRANNKey(tweetId, None, collectionName, maxCandidates, scoreThreshold)`）
  - CandidateSource: `DeepRetrievalTweetTweetANNCandidateSource`
  - Features: `TweetMixerCandidateFeatureTransformer`
  - Result変換: `TweetCandidate(id = candidate.tweetId)`

- TwHINTweetSimilarityCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `TwhinTweetSimilarity`
  - Gate: `TwhinTweetSimilarityEnabled`
  - Query変換: signalsFn から `Seq[TweetId]`
  - CandidateSource: `TwHINANNCandidateSource`
  - Features: `TweetMixerCandidateFeatureTransformer`
  - Result変換: `TweetCandidate(id = candidate.tweetId)`

- PopularGeoTweetsCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `PopularGeoTweets`
  - 対応クライアント Param: `PopularGeoTweetsEnable`
  - Query変換: `TripStratoGeoQueryTransformer`（GeoSourceIds/MaxPerSource/MaxCandidates）→ `TripStratoGeoQuery`
  - CandidateSource: `PopularGeoTweetsCandidateSource`
  - Features: `TripTweetFeatureTransformer`
  - Result変換: `TweetCandidate(id = TripTweet.tweetId)`

- ContentExplorationEmbeddingSimilarityCandidatePipelineConfigFactory.scala
  - 識別子: `HomeRecommendedTweets` + `ContentExplorationEmbeddingSimilarity`
  - Gates: `ContentExplorationEmbeddingSimilarityEnabled`, `MinTimeSinceLastRequestGate`
  - Query特徴水和: `ContentExplorationEmbedding/ContentMediaEmbedding/MultimodalEmbedding` 各 QueryHydrator
  - Query変換: ベクトルDBコレクション名/MaxCandidates/Score閾値 + 各埋め込みマップから `EmbeddingMultipleANNQuery(EmbeddingANNKey...)` を構成（Media埋め込み/Multimodal の切替ロジックあり）
  - CandidateSource: `EmbeddingANNCandidateSource`
  - Features: `TweetMixerCandidateFeatureTransformer`
  - Result変換: `TweetCandidate(id = candidate.tweetId)`

- その他（多数）
  - UTG/UVG Producer/Tweet/Consumer Based、Embedding/DR/TwHIN 系、Popular/Topic/Geo/Trends 系、Pinned/Curated など、多数の ConfigFactory が存在（順次追記）。

共通パターン
- identifier は `identifierPrefix`（例: HomeRecommendedTweets） + 固有サフィックス（`CandidatePipelineConstants`）で構成
- Gate で FSParam/Decider を条件化
- CandidatePipelineQueryTransformer で Query 型へ変換
- CandidateSource から `TweetMixerCandidate` を取得し Feature 変換
- CandidatePipelineResultsTransformer で最終 `TweetCandidate` に射影

今後の追記予定
- 各 ConfigFactory（〜PipelineConfigFactory.scala）ごとのパラメータと変換詳細
- `CandidatePipelineConstants` に列挙される識別子と対応表
- tweet-mixer/param/ 下の Param との紐付け
