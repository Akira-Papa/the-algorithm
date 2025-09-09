# ビルドとリポジトリ構成

- ビルド: 多くのコンポーネントに Bazel の `BUILD`/`BUILD.bazel` が付属。ただしリポジトリ直下のルート `WORKSPACE`/トップレベル `BUILD` は未同梱。
- 主言語: Scala（Finagle/Scrooge 生態系）、Java（検索/Earlybird）、Rust（`navi`）、Python（学習・軽ランカー等）
- サービス横断依存: Thrift IDL（`src/thrift` 配下など）、埋め込み/特徴ストア、Kafka ストリーム、GraphJet、Manhattan（ストレージ）等

代表ディレクトリ

- ルート
  - `README.md`: 全体アーキテクチャと主要コンポーネント一覧
  - `RETREIVAL_SIGNALS.md`: 候補取得で用いるユーザー行動シグナルの一覧と各コンポーネントでの利用
- アプリ/サービス
  - `home-mixer`, `product-mixer`, `tweet-mixer`, `timelineranker`, `follow-recommendations-service`, `graph-feature-service`, `pushservice`
- データ/モデル/共通
  - `tweetypie`, `unified_user_actions`, `user-signal-service`, `representation-manager`, `representation-scorer`, `visibilitylib`
- 検索/候補
  - `src/java/com/twitter/search`（Earlybird）, `src/scala/com/twitter/recos`（GraphJet群）, `simclusters-ann`

補足

- Bazel の単体ビルド/テストは各サブディレクトリ単位で提供されているものがあります（例: `simclusters-ann`）。
- 一部内部環境依存のコードは削除/サニタイズされている旨が `visibilitylib/README.md` 等で明記されています。

