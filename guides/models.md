# モデル群の概要

SimClusters / TwHIN / Trust & Safety / RealGraph / TweepCred などのモデルは、候補生成・特徴水和・ランキングの各段階で重要な信号を提供します。

- SimClusters（疎埋め込み）
  - パス: `src/scala/com/twitter/simclusters_v2`（README は直下 README.md 参照）
  - コミュニティ検出とユーザー/ツイートのクラスタ表現
  - オンライン: 類似性、近傍候補（`simclusters-ann` 参照）

- TwHIN（密埋め込み）
  - 外部レポ: the-algorithm-ml / projects/twhin
  - ユーザー/ツイートの知識グラフ埋め込み。類似度は Retrieval/Ranking 特徴として利用

- Trust & Safety モデル
  - パス: `trust_and_safety_models/README.md`
  - pNSFWMedia, pNSFWText, pToxicity, pAbuse など
  - 可視性/ダウンランク/フィルタ/注意喚起の判断材料

- RealGraph（相互作用確率）
  - パス: `src/scala/com/twitter/interaction_graph`（`scio` ベースの特徴/学習処理）
  - ユーザー間の将来相互作用の確率推定（フォロー/いいね/RT/返信など）

- TweepCred（信用度/PageRank）
  - パス: `src/scala/com/twitter/graph/batch/job/tweepcred`（README 記載）
  - 著者の信頼性/影響度指標として利用

- 表現取得/スコアリング周辺
  - `representation-manager/README.md`: 埋め込みの一元取得（RMS）
  - `representation-scorer/README.md`: 埋め込みベースのスコア計算（RSX）

