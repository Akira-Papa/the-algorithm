# Product Mixer（パイプラインフレームワーク）

パス: `product-mixer/README.md`

概要

- 小さく再利用可能なコンポーネントを組み合わせ、宣言的にパイプライン（Product/Mixer/Recommendation/Candidate/Scoring）を構成
- サービス骨格 + コンポーネントライブラリ + 監視/観測を包括

主な概念

- Product Pipeline: リクエストを受け、実行すべき Mixer/Recommendation Pipeline を選択
- Mixer Pipeline: 異種候補（広告/ツイート/ユーザー等）を統合
- Recommendation Pipeline: 同種候補の特徴水和→スコア→ランク
- Candidate Pipeline: 候補ソース呼び出し、軽フィルタ/装飾/特徴水和
- Scoring Pipeline: モデル推論やスコア集約

拡張点

- 独自の Candidate/Filter/Hydrator/SideEffect/Gate コンポーネントを追加
- 構成クラスでパイプラインを宣言的に組成

