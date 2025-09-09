# データとシグナル

Unified User Actions（UUA）と User Signal Service（USS）を中心に、ユーザー行動シグナルを統合・標準化して候補生成やランキング特徴に供給します。

- `unified_user_actions/README.md`: クライアント/サーバの行動イベントを統合し Kafka/GCP 等へ配信
- `user-signal-service/README.md`: 明示（フォロー、いいね等）と暗黙（クリック、視聴、プロフィール訪問等）の標準化シグナルをオンライン提供
- 参考: ルート `RETREIVAL_SIGNALS.md` に候補取得段階でのシグナル一覧と、各コンポーネント（USS/SimClusters/TwHIN/UTEG/FRS/LightRanker）での活用表

主なシグナル例（抜粋）

- フォロー/アンフォロー、ミュート/ブロック
- ツイートお気に入り/取り消し/RT/引用/返信/共有/ブックマーク
- クリック/ビデオ視聴
- 「興味なし」/通報
- 通知オープン/Ntab クリック
- アドレス帳（関係性推定）

用途

- 学習ラベル（例: Heavy Ranker/FRS/TwHIN/SimClusters）
- オンライン特徴（水和で参照）
- 候補生成のスコアリング（GraphJet/ANN/Embedding 類似など）

