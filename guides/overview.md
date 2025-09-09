# 概要（全体像）

このリポジトリは、X の推薦アルゴリズムを構成する複数のサービス群・バッチ/ストリーミング処理・機械学習モデル・共通フレームワークの集合体です。ホームタイムライン（For You / Following）、検索、Explore、通知など様々なプロダクト面で、共通のデータ・モデル・ソフトウェア基盤の上に動作します。

参考: README.md（リポジトリ直下）

- データ系
  - `tweetypie`: ツイートの読み書きを担う中核サービス（コアTweetサービス）
  - `unified_user_actions`: クライアント/サーバイベントを統合したユーザー行動のリアルタイムストリーム
  - `user-signal-service`: 明示/暗黙の行動シグナルを統合して提供

- モデル系
  - `src/scala/com/twitter/simclusters_v2`: コミュニティ検出と疎な埋め込み（SimClusters）
  - TwHIN（外部レポ）: 知識グラフベースの密埋め込み
  - `trust_and_safety_models`: NSFW/Abuse/Toxicityなどの安全性モデル
  - RealGraph（`src/scala/com/twitter/interaction_graph`）: ユーザー間相互作用の確率モデル
  - TweepCred: ユーザーの信用度（PageRank系）

- フレームワーク系
  - `product-mixer`: 推薦/ミキシング用のパイプラインフレームワーク
  - `navi`: Rust 製の高速モデルサービング
  - `timelines/data_processing/ml_util/aggregation_framework`: 特徴量集計フレームワーク
  - `representation-manager`: 埋め込み取得の集約サービス

- 候補生成・ランク・ミキシング（代表）
  - 検索インデックス（`src/java/com/twitter/search` = Earlybird）: In-Network候補の主要ソース
  - `tweet-mixer`: Out-of-Network候補の調停・フィルタ・軽ランク
  - GraphJet 系: UTEG/UUG/UTG など（ユーザー×ツイート/ユーザー×ユーザーの相互作用グラフ）
  - `follow-recommendations-service` (FRS): アカウント推薦/将来フォローを見込んだツイート推薦
  - `home-mixer`: Product Mixer 上に構築されたホーム構築の中核サービス
  - `visibilitylib`: 可視性フィルタ（法令対応/品質/信頼/レベニュー保護等）

For You の高レベルな流れ

1) 候補生成（In-Network/Out-of-Network）
2) 特徴量水和（~6000特征）
3) スコア/ランキング（Light/Heavy Ranker）
4) フィルタ/ヒューリスティクス（多様性、既視、可視性等）
5) ミキシング（広告、WTF、プロンプト等）
6) 応答整形（会話モジュール、ソーシャルコンテキスト、ページネーション等）

本ガイド群は上記を土台に、各サービス/コンポーネントの役割とI/O、主要依存関係、代表的な設定/エンドポイント/拡張点を解説します。

