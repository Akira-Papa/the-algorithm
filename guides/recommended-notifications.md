# 推薦通知（Recommended Notifications）

参考: リポジトリ直下 `README.md`

- サービス: `pushservice`
- ランキング:
  - Light Ranker: `pushservice/src/main/python/models/light_ranking/README.md`
  - Heavy Ranker: `pushservice/src/main/python/models/heavy_ranking/README.md`

パイプライン概要

1) 候補生成（通知対象の候補ツイート/イベント）
2) 特徴量水和（ユーザー/イベント/履歴/埋め込み 等）
3) Light Ranking（巨大候補から高関連のみを事前選抜）
4) Heavy Ranking（開封/エンゲージ予測の多タスク学習）
5) フィルタ/トランスフォーム（品質/安全/既視/頻度 等）
6) 送達制御・観測（メトリクス/ロギング）

備考

- Home/For You 同様、USS/埋め込み/RMS/RSX 等の共通基盤を特徴計算に活用
- Heavy Ranker は開封確率とエンゲージ確率の結合を最適化

