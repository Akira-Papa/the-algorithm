# Tweet Mixer（Out-of-Network 候補の調停）

パス: `tweet-mixer/server/src/main/scala/com/twitter/tweet_mixer`

役割

- Out-of-Network（OON）候補の取得・ミックス・フィルタ・軽ランキングを司る調停層
- 多数の CandidatePipelineConfigFactory（SimClusters/TwHIN/UTG/UVG/UTEG/コンテンツ探索 等）を切替

代表的な構成要素（抜粋）

- functional_component
  - filter: 既視/メディア重複/スコア閾値/可視性/動画特性 等
  - gate: ユーザー条件（低シグナル/直近リクエスト間隔/フォロワー数 等）
  - hydrator: USS/埋め込み/カテゴリ/外部サービス由来の特徴水和
 - side_effect: ログ/計測/埋め込み要求/デバッグ 等
- candidate_pipeline（多数の〜PipelineConfigFactory 類）
  - SimClusters/TwHIN/Embedding 類似/Deep Retrieval/UTG/UVG/UTEG/人気トピック/位置情報/動画系 等

ポイント

- Out-of-Network を広く探索しつつ、重複/スパム/品質/露出制御で質を担保
- 後段（Home Mixer/Recommendation Pipeline）へ渡すための前処理/整形を担う

ファイル別詳細（継続拡充中）

- functional_component 詳細: `guides/services/tweet-mixer-functional-components.md`
- candidate_source 詳細: `guides/services/tweet-mixer-candidate-sources.md`
- candidate_pipeline 詳細: `guides/services/tweet-mixer-candidate-pipelines.md`
- パラメータ/Decider/定数: `guides/services/tweet-mixer-params-and-utils.md`
