## Follow Recommendations Service（FRS）

パス: `follow-recommendations-service/README.md`

役割

- アカウント推薦（Who-To-Follow）および、将来フォローが見込まれる著者のツイート推薦（FutureGraph）
- 用途別（表示面）に RecommendationFlow を組成（候補生成→特徴水和→ランク→フィルタ→変換→切詰）

構成の手掛かり

- 表示面（DisplayLocation）: `follow-recommendations-service/common/.../DisplayLocation.scala`
- フロー定義: `follow-recommendations-service/server/.../flows`
- プロダクト: `follow-recommendations-service/server/.../products/home_timeline_tweet_recs`
- 候補ソース: `follow-recommendations-service/common/.../candidate_sources/`（各フォルダに README）
- フィルタ: `follow-recommendations-service/common/.../predicates`
- ランカー: `follow-recommendations-service/common/.../rankers`
- 変換: `follow-recommendations-service/common/.../transforms`

ランク

- ML ランカーは DataRecord を構築し、別サービスの推論を呼び出してスコア（p(follow) とエンゲージの重み付き）で並べ替え

