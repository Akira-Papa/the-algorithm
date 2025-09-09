# ガイド: X 推薦アルゴリズム解説インデックス

このフォルダでは、リポジトリ全体を横断的に解析し、X（旧Twitter）の推薦アルゴリズム実装の仕様を日本語で体系的に解説します。まず全体像と主要コンポーネントのガイドから提供し、その後サービス別・モジュール別に詳細を追加していきます。

- 概要と全体像
  - overview.md
  - build-and-repo.md

- プロダクト別の流れ
  - for-you-timeline.md
  - recommended-notifications.md

- データ・シグナル・モデル
  - data-and-signals.md
  - models.md

- 主要サービス（アプリ層）
  - services/home-mixer.md
  - services/product-mixer.md
  - services/tweet-mixer.md
  - services/timelineranker.md
  - services/search-earlybird.md
  - services/tweetypie.md
  - services/graph-feature-service.md
  - services/follow-recommendations-service.md

- 候補生成・グラフ（GraphJet系）
  - recos/graphjet-services.md

今後、各サブシステム（例: representation-manager、representation-scorer、visibilitylib、simclusters-ann、navi など）の詳細ガイドも継続的に追加していきます。

