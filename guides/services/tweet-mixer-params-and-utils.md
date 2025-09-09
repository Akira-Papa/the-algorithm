# Tweet Mixer Params / Utils / Scoring 概要

対象: `tweet-mixer/server/src/main/scala/com/twitter/tweet_mixer/param/*`, `utils/*`, `scoring_pipeline/*`

Params（例・抜粋）
- BlendingParam（RoundRobin/SignalPriority/Weighted/Source/RankingOnly）: 混合戦略の切替
- Enable〜（多数）: Hydrator/Filter/SideEffect/Source を有効化する FSParam
- DeepRetrieval/Embedding/SimClusters/UTG/UVG/ContentExploration 系: 各候補ソースのスコア閾値・MaxCandidates・ベクトルDB名など
- Video 関連（Short/Long/Portrait、最小/最大長・解像度・アスペクト比など）
- MaxTweetAge/MinVideoDuration など閾値類

代表Paramグループ（実コードより抜粋）
- Earlybird/Haplo: `HaploliteTweetsBasedEnabled`
- Hydra: `HydraScoringPipelineEnabled`, `HydraBasedSortingEnabled`, `HydraModelName`, `EnableHydraScoringSideEffect`
- SimClusters: `SimclustersInterestedInEnabled`, `SimclustersProducerBasedEnabled`, `SimclustersTweetBasedEnabled`, `SimclustersPromotedCreatorEnabled`
- UTG/UVG: `UTGTweetBasedEnabled`, `UTGProducerBasedEnabled`, `UTGTweetBasedExpansionEnabled`, `UVGTweetBasedEnabled`, `UVGTweetBasedExpansionEnabled`
- DeepRetrieval I2I/U2I: `DeepRetrievalTweetTweetANNEnabled`, `DeepRetrievalTweetTweetEmbeddingANNEnabled`, `DeepRetrievalUserTweetANNEnabled`, `DeepRetrievalI2iVectorDBCollectionName`, `DeepRetrievalTweetTweetANNMaxCandidates`, `DeepRetrievalTweetTweetANNScoreThreshold`, `DeepRetrievalEnableGPU`
- ContentExploration: `ContentExplorationEmbeddingSimilarityEnabled`, `ContentExplorationMediaEmbeddingSimilarityEnabled`, `ContentExplorationMultimodalEnabled`, `ContentExplorationVectorDBCollectionName`, `ContentExplorationEmbeddingANNMaxCandidates`, `ContentExplorationEmbeddingANNScoreThreshold`
- Videoフィルタ: `EnableVideoTweetFilter`, `EnableShortFormVideoFilter`, `EnableLongFormVideoFilter`, `EnablePortraitVideoFilter`, 動画長/解像度/アスペクト比などの Min/Max 群
- Hydratorトグル: `EnableUSSFeatureHydrator`, `EnableUSSGrokCategoryFeatureHydrator`, `EnableUSSDeepRetrievalTweetEmbeddingFeatureHydrator`, `EnableGizmoduckQueryFeatureHydrator`, `EnableVideoBloomFilterHydrator`, `ViewCountInfoOnTweetFeatureHydratorEnabled`, `GrokFilterFeatureHydratorEnabled`, `EnableUserTopicIdsHydrator`
- Scribe/監査: `ScribeRetrievedCandidatesParam`

Utils（例・抜粋）
- CandidatePipelineConstants: パイプライン識別子サフィックスを集中管理
- Transformers: Memcache 値の Buf 変換器（long/double シーケンス等）
- MemCacheStitchClient / ConcurrentMapCache: キャッシュ連携
- BucketSnowflakeIdAgeStats: ID 年齢の分布統計
- PipelineFailureCategories: 失敗分類

Scoring（Hydra）
- scoring_pipeline/HydraScoringPipelineConfig.scala, scorer/HydraScorer.scala
- Hydra 経由のスコア取得と特徴マッピングを定義（詳細は後続で精読のうえ追記）

今後の追記予定
- Param 定義ファイルの全列挙とデフォルト/最小/最大/説明を一覧化
- ScoringPipeline と Home 側 Scoring の境界/IF 整理
- 各 Utils の内部実装要点（メモリ/効率性/エラー処理）
