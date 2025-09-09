# Tweet Mixer functional_component 詳解（ファイル別）

対象ディレクトリ: `tweet-mixer/server/src/main/scala/com/twitter/tweet_mixer/functional_component`

本ドキュメントは Filters / Gates / Hydrators / Side Effects の各ファイルを1つずつ読み、実装に基づく正確な要約を記載します（継続更新）。

参考: `TweetMixerFunctionalComponents.scala` は本層の「束ね役」で、グローバルなクエリ/候補ハイドレータ、各種セレクタ（Blending戦略: RoundRobin/SignalPriority/Weighted など）とグローバルフィルタ（可視性・既視除去）を構成します。

## Filters

- filter/ImpressedTweetsFilter.scala
  - 識別子: `ImpressedTweets`
  - 目的: USS の Tweet 関連シグナル（`USSFeatures.TweetFeatures`）と `HasExcludedIds` を統合し、既視/除外された TweetId または SourceTweetId の候補を除去。
  - 入出力: `Filter[PipelineQuery with HasExcludedIds, TweetCandidate]`。`FilterResult(kept/removed)` を返却。
  - 主な特徴/依存: `USSFeatures.getSignals[Long]`, `SourceTweetIdFeature`。

- filter/ImpressedTweetsBloomFilter.scala
  - 識別子: `ImpressedTweetsBloom`
  - 条件: `EnableImpressionBloomFilterHydrator` が true のときのみ適用。
  - 目的: Bloom Filter ストア（`ImpressionBloomFilter`）とUSSシグナルから既視TweetId/SourceTweetId を抽出し、既視候補を除去。
  - 入出力: `Filter[PipelineQuery, TweetCandidate]`（conditionally）。
  - 主な特徴/依存: `ImpressionBloomFilterFeature`, `USSFeatures`, `SourceTweetIdFeature`。

- filter/MediaIdDedupFilter.scala
  - 識別子: `MediaIdDedup`
  - 条件: `EnableMediaIdDedupFilter` が true。
  - 目的: `MediaIdFeature` を用いて同一メディアIDの重複候補を除去。特定の CandidatePipeline は `candidatePipelinesToExclude` で除外可能。
  - 入出力: `Filter[PipelineQuery with HasVideoType, TweetCandidate]`（conditionally）。

- filter/MediaWatchHistoryFilter.scala
  - 識別子: `MediaWatchHistory`
  - 条件: `EnableVideoBloomFilterHydrator` が true。
  - 目的: `VideoImpressionBloomFilterFeature`（BloomFilterのThrift）に含まれるメディア視聴履歴を用い、既視メディアIDの候補を除去。
  - 入出力: `Filter[PipelineQuery with HasVideoType, TweetCandidate]`（conditionally）。

- filter/TweetVisibilityAndReplyFilter.scala
  - 識別子: `TweetVisibilityPolicy`
  - 条件: `EnableDebugMode` が false のとき適用。
  - 目的: In-Network 候補は常に許可。それ以外は `TweetBooleanInfoFeature` が存在し、かつ `IsReply` が立っていない（返信ではない）場合のみ許可。`FromInNetworkSourceFeature` も参照。

- filter/MinScoreFilter.scala
  - 識別子: `MinScore`
  - 条件: ANN/Embedding 用スコア閾値 Param（`DeepRetrievalTweetTweetANNScoreThreshold`, `DeepRetrievalTweetTweetANNScoreMaxThreshold`, `DeepRetrievalTweetTweetEmbeddingANNScoreThreshold`）のいずれかが制約値を持つ場合。
  - 目的: 対象 Pipeline の候補に対し、`ScoreFeature` の最小/最大閾値でフィルタ。Embedding系 Pipeline の場合は別閾値を適用。
  - 入出力: `Filter[PipelineQuery, TweetCandidate]`（conditionally）。

- filter/MaxViewCountFilter.scala
  - 識別子: `MaxViewCount`
  - 条件: `ViewCountInfoOnTweetFeatureHydratorEnabled` かつ 閾値 Param（`ContentExplorationMaxViewCountThreshold` など）が有効。
  - 目的: コンテンツ探索（DR/Embedding Tier1/2 等）の候補に対し、`UecAggTweetTotalFeature` の `Displayed` 指標を参照して最大ビュー数で除外。

- filter/IsVideoTweetFilter.scala
  - 識別子: `IsVideoTweet`
  - 条件: `EnableVideoTweetFilter` が true。
  - 目的: 返信/複数メディア/URL を除外し、`HasVideo` かつ `IsHighMediaResolution` のツイートのみ許可。候補 Pipeline 範囲の限定も可能。

- filter/IsShortFormVideoFilter.scala / filter/IsLongFormVideoFilter.scala / filter/IsPortraitVideoFilter.scala
  - 目的: それぞれ短尺/長尺/縦長動画である候補のみを許可（VideoType/Paramに応じ条件適用）。

- filter/ShouldIgnoreCandidatePipelinesFilter.scala
  - 目的: `CandidatePipelines` 特徴に基づき、`candidatePipelinesToExclude` に一致するPipelineの候補をフィルタロジックから除外（無視）。

## Gates

- gate/AllowLowSignalUserGate.scala, DenyLowSignalUserGate.scala, MinTimeSinceLastRequestGate.scala, AllowNonEmptySearchHistoryUserGate.scala, MaxFollowersGate.scala, ProbablisticPassGate.scala
  - 目的: クエリ/ユーザー状態に応じてパイプライン実行を統制（低シグナルユーザー許可/拒否、直近リクエスト間隔、検索履歴有無、フォロワー数上限、確率的通過など）。
  - 具体条件・パラメータは各ファイルに定義。詳細は後続版でパラメータを紐付けて補記します。

## Hydrators（Query/Candidate Feature Hydrators）

- hydrator/USSGrokCategoryFeatureHydrator.scala
  - 識別子: `USSGrokCategory`
  - 条件: `EnableUSSGrokCategoryFeatureHydrator` が true。
  - 目的: Strato `PostAnnotationsOnTweet` からツイートのカテゴリ（Grok）を取得し、USS の Tweet ベース信号をカテゴリ多様化ロジックで分配して更新。
  - 出力: `USSGrokCategoryFeature` と、多様化後の `TweetFeatures` マップ。

- hydrator/USSDeepRetrievalTweetEmbeddingFeatureHydrator.scala
  - 識別子: `USSDeepRetrievalTweetEmbedding`
  - 条件: `EnableUSSDeepRetrievalTweetEmbeddingFeatureHydrator` が true。
  - 目的: Strato `CachedEgsTweetEmbedding` からツイート埋め込みを取得。ネガティブ埋め込み群とコサイン類似度を比較し、しきい値以上に近いポジティブ信号を除去（全体フィルタ率を統計に記録）。
  - 主パラメータ: `USSDeepRetrievalI2iEmbModelName`, `USSDeepRetrievalStratoTimeout`, `USSDeepRetrievalSimilarityTweetTweetANNScoreThreshold`。

- hydrator/MultimodalEmbeddingQueryFeatureHydratorFactory.scala
  - 識別子: `MultimodalEmbedding`
  - 目的: Strato `PostMultimodalEmbeddingMh` から対象ツイートのマルチモーダル埋め込みを取得し、`Map[Long, Option[Seq[Double]]]` として提供。

- hydrator/GizmoduckQueryFeatureHydrator.scala
  - 識別子: `Gizmoduck`
  - 条件: `EnableGizmoduckQueryFeatureHydrator` が true。
  - 目的: Gizmoduck から `Counts` を取得し、`UserFollowersCountFeature`（フォロワー数）を水和。

- hydrator/SGSFollowedUsersQueryFeatureHydrator.scala
  - 識別子: `SGSFollowedUsers`
  - 目的: SocialGraph からフォロー中（ミュート除外）ユーザーIDを取得し、クエリ特徴に追加。

- そのほか（後続で追記）
  - `TwhinUserPositiveEmbeddingQueryFeatureHydrator.scala`, `ContentExplorationDRUserEmbeddingQueryFeatureHydrator.scala`, `UVGOutlierSignalsQueryFeatureHydrator.scala`, `LastNonPollingTimeQueryFeatureHydrator.scala` など。

## Side Effects

- side_effect/ScribeServedCandidatesSideEffect.scala
  - 識別子: `ScribeServedCandidates`
  - 条件: 本番環境かつ `ScribeRetrievedCandidatesParam` が有効、かつ選択候補が非空。
  - 目的: Kafka `tweet_mixer_retrieved_candidates` に、選択/残余候補の情報（スコア/ソース信号など）を `ServedCandidatesForRequest` として書き込み（監視・分析用途）。

- side_effect/HydraScoringSideEffect.scala
  - 識別子: `HydraScoring`
  - 条件: `EnableHydraScoringSideEffect` が true。
  - 目的: Hydra Root へのダークトラフィックで TweetRanking を呼び、候補のスコアリング応答を取得（副作用として計測・比較）。

- side_effect/SelectedStatsSideEffect.scala
  - 識別子: `SelectedStats`
  - 目的: 選択候補の属性（動画/長尺/返信/インネット等）や未スコア件数、HighQuality ソース信号の有無等を Stats へ記録。

- そのほか（後続で追記）
  - `PublishGroxUserInterestsSideEffect.scala`, `RequestMultimodalEmbeddingSideEffect.scala`, `EvergreenVideosSideEffect.scala`, `DeepRetrievalAdHocSideEffect.scala` など。

## TweetMixerFunctionalComponents.scala（束ね/選抜ロジック）

- グローバル Query/Candidate ハイドレータ
  - TweetypieCandidateFeatureHydrator（安全レベル: Recommendations）
  - MediaMetadataCandidateFeatureHydrator（Param で有効化）
  - UecAggTweetTotalFeatureHydrator（表示回数など、コンテンツ探索系 Pipeline のみ）
  - GrokFilterFeatureHydrator（Param で有効化）
  - USS/HighQualitySource/PlaceIds/TopicIds などの Query 特徴

- グローバルフィルタ
  - `TweetVisibilityAndReplyFilter`, `ImpressedTweetsFilter`

- セレクタ（Blending 切替）
  - RoundRobinBlending: `InsertAppendWeaveResults(AllPipelines, bucketer)`
  - SignalPriorityBlending: `InsertAppendPriorityWeaveResults(..., priorityFn)`
  - WeightedPriorityBlending: `InsertAppendWeightedSignalPriorityWeaveResults(..., weightedFn)`
  - RankingOnly: `InsertAppendResults(...)` と重複除去の有無（`DropDuplicateResults`）

---

今後の追記予定

- Gates 各ファイルの条件式/パラメータの詳細列挙
- Hydrator/SideEffect の未記載ファイルの精読結果
- candidate_pipeline / candidate_source のファイル別詳細は別ドキュメントで追加

