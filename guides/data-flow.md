## データフローと依存関係（横断）

参考: `docs/system-diagram.png`、各サービス README

高レベルフロー（For You 例）

- 生成系（ストリーム/バッチ）
  - UUA（行動）→ USS（標準化）→ 特徴/埋め込み生成（SimClusters/TwHIN/RealGraph/RSX/RMS）
  - Recos-Injector → GraphJet（UTEG/UTG/UUG）
  - 検索取込（Kafka）→ Earlybird インデックス
- オンライン照会
  - Home Mixer（Product Mixer）
    - In-Network: Earlybird
    - Out-of-Network: Tweet Mixer（UTEG/UTG/SimClusters/TwHIN/DR 等経由）/ FRS
    - 特徴水和: USS/GFS/RMS/RSX/Tweetypie/可視性等
    - ランキング: Light/Heavy
    - フィルタ/ミキシング/応答整形

主要依存（例）

- `home-mixer` → `tweet-mixer` / `src/java/com/twitter/search` / `src/scala/com/twitter/recos/...` / `follow-recommendations-service` / `visibilitylib` / `graph-feature-service` / `tweetypie`
- `tweet-mixer` → USS/RMS/RSX/各候補ソース（UTEG/UTG/SimClusters/TwHIN/ANN 等）
- `timelineranker` → Earlybird/UTEG/SocialGraph/Tweetypie/Manhattan
- `follow-recommendations-service` → 候補ソース/ランカー/USS/特徴水和
- `graph-feature-service` → Social/Engagement グラフストア
- `src/java/com/twitter/search` → Kafka（インデクシング）/Light Ranker

注意

- 可視性（`visibilitylib`）は法令/安全/品質要件に基づくフィルタ/トリートメントを返し、下流で適用
- 一部内部限定の構成や非公開モデルは除去/簡略化されていることがあります

