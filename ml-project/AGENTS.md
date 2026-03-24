# [MLプロジェクト名]

## コンテキスト
[研究・開発の目的。例：金融時系列予測モデルの開発 / 文書分類システムの構築]

## 技術スタック
- 言語：Python [バージョン]
- 主要ライブラリ：[PyTorch / TensorFlow / scikit-learn / etc.]
- 実験管理：[MLflow / Weights & Biases / etc.]
- データ処理：[pandas / polars / etc.]
- パッケージ管理：[uv / poetry / etc.]

## コマンド
```bash
# 環境セットアップ
[コマンド]

# データ前処理
[コマンド]

# 実験実行
[コマンド]

# 評価・レポート生成
[コマンド]
```

## ディレクトリ構造
```
project/
├── data/
│   ├── raw/          # 生データ（変更禁止）
│   ├── processed/    # 前処理済みデータ
│   └── external/     # 外部データ
├── notebooks/        # 探索・可視化（実験コードは置かない）
├── src/
│   ├── data/         # データローダー・前処理
│   ├── models/       # モデル定義
│   ├── training/     # 学習ループ
│   └── evaluation/   # 評価指標
├── experiments/      # 実験設定（config）
├── outputs/          # 実験結果（自動生成）
└── papers/           # 参照論文PDF
```

## 実験管理のルール
- 実験はすべて [MLflow/W&B] に記録する
- ハイパーパラメータはconfigファイルで管理（コードに直書きしない）
- 実験結果の再現性を保つためにランダムシードを固定する（seed=42）
- モデルの重みは outputs/{experiment_id}/ に保存する

## プロジェクト固有のルール
- [例：data/raw/ は読み取り専用（変更・削除禁止）]
- [例：Notebookには探索コードのみ、本番コードはsrc/に書く]
- [例：評価指標は必ず [F1/AUC/RMSE etc.] を使う]

## 禁止事項
- data/raw/ の変更・削除
- 実験結果をコードにハードコードすること
- 未記録の実験（必ずMLflow/W&Bに記録する）
