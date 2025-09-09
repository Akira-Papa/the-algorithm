## 検索インデックス（Earlybird）

パス: `src/java/com/twitter/search/README.md`

概要

- Lucene ベースのリアルタイム検索システム。インデックスを Realtime/Protected/Archive に分割し、シャーディング/複製でスループットとレイテンシを両立
- タイムライン In-Network 候補（Following/For You）の主要ソース

インデクシング

- Kafka からツイート/ユーザー更新を取り込み、特徴抽出し、Earlybird ノードが消費してインデックス
- エンゲージメント等の特徴更新も別経路から供給

サービング

- ルートがクエリをクラスター/パーティションへファンアウトし、応答をマージして返却
- In-Network では「フォロー中ユーザーの最近の投稿」を高速に探索・スコアリング

関連

- Light Ranker: `src/python/twitter/deepbird/projects/timelines/scripts/models/earlybird/README.md`
- Home Mixer/TimelineRanker から呼び出し

