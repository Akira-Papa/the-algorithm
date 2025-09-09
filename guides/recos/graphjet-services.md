## GraphJet 系サービス（UTEG / UUG / UTG）

パス:

- UTEG（UserTweetEntityGraph）: `src/scala/com/twitter/recos/user_tweet_entity_graph/README.md`
- UUG（UserUserGraph）: `src/scala/com/twitter/recos/user_user_graph/README.md`
- UTG（UserTweetGraph）: `src/scala/com/twitter/recos/user_tweet_graph/README.md`

共通点

- GraphJet 上に構築された Finagle Thrift サービス
- Kafka ストリームからエッジを取り込み、メモリ内グラフを維持（UTEG/UTG は24〜48時間、UUGは約1週間の窓）
- 協調フィルタリングの原理で、入力の重み付きフォロー/エンゲージメント近傍から候補を集約

UTEG

- 友人が最近エンゲージしたツイートを提示（Home の「XXさんがいいねしました」）
- 片方向（ユーザー→ツイート）エッジで計算を簡素化

UUG

- フォロー推薦（フォローグラフに基づく協調フィルタ）

UTG

- ユーザー↔ツイートの双方向エッジで多段ホップ（1〜3+）の柔軟なトラバーサルを実現（メモリ要求は高め）
- Recos-Injector 生成のストリームを取り込み

補助/共通ユーティリティ

- `src/scala/com/twitter/recos/graph_common` / `hose/common` / 各 `.../util` にビルダ/カウンタ/フィルタ等

