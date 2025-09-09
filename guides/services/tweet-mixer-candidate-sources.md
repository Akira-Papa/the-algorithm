# Tweet Mixer candidate_source 詳解（ファイル別）

対象ディレクトリ: `tweet-mixer/server/src/main/scala/com/twitter/tweet_mixer/candidate_source`

本ドキュメントは candidate_source 配下の各ファイルを読み、入力/出力・外部依存・キャッシュ・主要パラメータ/動作を要約します（継続更新）。

- ndr_ann（Deep Retrieval / Embedding / VecDB 系）
  - DeepRetrievalTweetTweetANNCandidateSource.scala: VecDB 検索（`searchById`）を用いて seed（`DRANNKey`）ごとに (tweetId, score) を取得し、`TweetMixerCandidate` へ。Memcached 経由で TTL=~10分。スコア・年齢の統計を記録し、複数シード結果を `interleave` で合成。
  - DeepRetrievalTweetTweetEmbeddingANNCandidateSource.scala: 埋め込みベクトルで VecDB `search` もしくは GPU HTTP 近傍検索を実行。tier フィルタやスコア閾値、maxCandidates を適用。TTL=~3分。Factory で識別子を切替可能。
  - EmbeddingANNCandidateSource.scala, UserInterestANNCandidateSource.scala, DRMultipleANNQuery.scala, DRANNKey.scala: ANN クエリ/キーや他のANN実装（別途追記）

- simclusters_ann（SimClusters ANN）
  - SimClustersAnnCandidateSource.scala: SimClustersANN サービスをモデル/埋め込み種から動的に選択。`getTweetCandidates` の応答を (tweetId, score) で保持→seedId を付与して `TweetMixerCandidate` に。minScore, maxCandidates でフィルタ。Memcached TTL=~10分。
  - SimclusterPromotedCreatorAnnCandidateSource.scala, SimclusterColdPosts*（別途追記）

- UTG（User Tweet Graph）
  - UserTweetGraphTweetBasedCandidateSource.scala: GraphJet ベースの UTG `tweetBasedRelatedTweets` を呼び、(id, score)→seedId を `TweetMixerCandidate` に。Memcached TTL=~10分。
  - UserTweetGraphConsumerBasedCandidateSource.scala: 直近エンゲージユーザー（`RecentEngagedUsersCandidateSource`）を取得→UTG の consumers ベース関連ツイート。シードごとに拡張のためのユーザーリストを最大N件用意。Memcached TTL=~10分。
  - UserTweetGraphProducerBasedCandidateSource.scala（別途追記）

- UVG（User Video Graph）
  - UserVideoGraphTweetBasedCandidateSource.scala: UVG の `tweetBasedRelatedTweets` 呼び出し。Memcached TTL=~10分。tweetId/score を seedId 付き `TweetMixerCandidate` へ変換。
  - UserVideoGraphConsumerBasedCandidateSource.scala: UVG consumer ベース版（別途）

- content_embedding_ann / text_embedding_ann（埋め込み候補）
  - ContentEmbeddingAnnCandidateSource.scala: Strato `RealtimeTopCandidatesBlendedOnTweet` から seed ごとの候補を取得。国/言語/減衰/メディア/テキストソースフラグなど View で制御。InProcessCache で短期キャッシュ。
  - TextEmbeddingCandidateSource.scala: Strato VectorDB `RealtimeTopV8` スキャンによりベクトル近傍を取得。`TextEmbeddingQuery`（ベクトル/limit）を受け取り、`TweetMixerCandidate` 化。

- earlybird_realtime_cg（In-Network リアルタイム）
  - EarlybirdRealtimeCGTweetCandidateSource.scala: Earlybird `search` を呼び、`ThriftSearchResult` 群を返す。必要に応じて Haplolite に Home タイムラインエントリとして書き戻し（`setEntries`）。Tweet メタから `Tweet` 構造体を組み立て、TimelineKind.home でエンコードして保存。Identifier は `EarlybirdRealtimeCGTweets`。

- trends / popular_* / curated_* / events / topic_tweets / user_location など（非ANN系）
  - TrendsCandidateSource.scala: Strato `TopPostsPerCountry` を国コードで取得し、clusterId を seedId に採用。InProcessCache で短期キャッシュ。
  - PopularGeoTweetsCandidateSource.scala, PopGrokTopicTweetsCandidateSource.scala, PopularTopicTweetsCandidateSource.scala, SkitTopicTweetsCandidateSource.scala, CertoTopicTweetsCandidateSource.scala（別途追記）
  - CuratedUserTlsPerLanguageCandidateSource.scala: 言語別のキュレーションユーザーTLS（別途）
  - EventsCandidateSource.scala: イベント由来候補（別途）
  - UserLocationCandidateSource.scala: 位置情報に応じた候補（別途）

- uss_service / engaged_users / cached_candidate_source
  - USSSignalCandidateSource.scala: Strato `Signals` から `(SignalType, Seq[UssSignal])` を取得し返す KeyFetcherSource。下流のパイプラインで信号として利用。
  - RecentEngagedUsersCandidateSource.scala（別途）: seedTweet に対する recent engaged users を返す補助ソース。
  - MemcachedCandidateSource.scala: 共通の Memcache 混載基底。key/value Transformer と TTL、enableCache 切替を提供。

- evergreen_videos / semantic_video
  - TwitterClipV0Short/LongVideoCandidateSource.scala, MemeVideoCandidateSource.scala, HistoricalEvergreenVideosCandidateSource.scala, SemanticVideoCandidateSource.scala（別途）
  - EvergreenVideosSearchBy*Query.scala: 検索条件を保持するクエリ型

補足
- ほぼ全てのソースが seed 単位で (id, score) を取り、`TweetMixerCandidate(tweetId=id, score, seedId)` に正規化して返却。複数 seed の出力は `TweetMixerCandidate.interleave` でインタリーブしています。
- キャッシュは Memcached または InProcessCache を用途に応じて使い分け。キーはコレクション名/しきい値/seedなどを連結して生成。

今後の追記予定
- 各「別途追記」ファイルのコード精読と要約追加
- 参照パラメータ/Decider/制御フラグ（`TweetMixerGlobalParams.scala`）との対応表
- 外部依存（Strato 列/サービス名/VecDBコレクション）を一覧化
