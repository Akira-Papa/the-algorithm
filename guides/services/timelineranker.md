# TimelineRanker（レガシー連携層）

パス: `timelineranker/README.md`

役割

- Earlybird（検索インデックス）と UTEG から得たツイート候補に対し、軽いランキング/トランケーションを行い Home へ返すレガシー層
- 現在は Heavy Ranking は行わず、Earlybird のスコアに基づく軽ランクが中心

主な連携先

- Earlybird（検索）: In-Network 候補の取得
- UTEG: 友人がいいねした OON 候補の取得
- Social Graph: フォロー/ブロック/ミュート等の状態
- Tweetypie: ハイドレーションによるポストフィルタ
- Manhattan: 一部特徴の取得

Home Mixer との関係

- For You/Following 双方で候補取得に利用可能
- Home Mixer からの要求に応じて、スコア順に所望件数まで切り詰めて返却

