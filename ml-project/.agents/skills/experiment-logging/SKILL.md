---
name: experiment-logging
description: 実験記録の手順と規則。実験を実行するとき・結果を保存するときに自動で使う。MLflow/W&Bへの記録方法と出力ファイルの命名規則を含む。
---

# 実験記録 手順

## 実験IDの命名規則

```
{YYYYMMDD}_{連番}_{手法の略称}
例：
  20240315_001_baseline
  20240315_002_lr_warmup
  20240316_001_transformer_v2
```

## 記録必須項目

### 実験前に記録する
- 実験の目的・仮説
- 変更した設定（前回との差分）
- 使用データセット・バージョン

### 実験後に記録する
- 全評価指標（train / val / test）
- 実行時間・使用リソース
- 気づいたこと・懸念事項

---

## MLflow での記録（MLflow使用時）

```python
import mlflow

with mlflow.start_run(run_name=experiment_id):
    # パラメータの記録
    mlflow.log_params({
        "model_type": config.model.type,
        "learning_rate": config.training.lr,
        "batch_size": config.training.batch_size,
        "seed": config.training.seed,
    })
    
    # 実験の目的をタグで記録
    mlflow.set_tags({
        "purpose": "何を検証するか",
        "hypothesis": "なぜ効くと思うか",
        "based_on": "ベースの実験ID",
    })
    
    # 学習中のメトリクス記録
    for epoch in range(epochs):
        mlflow.log_metrics({
            "train_loss": train_loss,
            "val_loss": val_loss,
            "val_metric": val_metric,
        }, step=epoch)
    
    # 最終結果
    mlflow.log_metrics({
        "test_metric": test_metric,
        "best_val_metric": best_val_metric,
    })
    
    # モデルの保存
    mlflow.pytorch.log_model(model, "model")
```

## W&B での記録（W&B使用時）

```python
import wandb

wandb.init(
    project="[プロジェクト名]",
    name=experiment_id,
    config=config,
    tags=["baseline", "v1"],
    notes="何を検証するか",
)

# 学習中
wandb.log({"train_loss": loss, "val_metric": metric}, step=epoch)

# 終了
wandb.finish()
```

---

## 出力ファイルの保存規則

```
outputs/
└── {experiment_id}/
    ├── config.yaml          # 実験設定（自動コピー）
    ├── metrics.json         # 評価指標の最終値
    ├── model_best.pt        # 最良モデルの重み
    ├── training_log.txt     # 学習ログ
    └── notes.md             # 実験メモ・気づき
```

## metrics.json の形式

```json
{
  "experiment_id": "20240315_001_baseline",
  "timestamp": "2024-03-15T14:30:00",
  "config_hash": "abc123",
  "results": {
    "train": {"loss": 0.123, "metric": 0.456},
    "val":   {"loss": 0.145, "metric": 0.423},
    "test":  {"loss": 0.150, "metric": 0.419}
  },
  "training_time_sec": 3600,
  "notes": "学習は安定していた。val lossが途中から上がり始めた"
}
```

---

## autoresearch との共存

autoresearch は git commit をメモリとして使う。
手動実験の記録（outputs/ + MLflow/W&B）と共存させるためのルール。

### commit メッセージの統一
autoresearch に以下の形式で commit するよう指示する：

```
# autoresearch 呼び出し時に CommitPrefix を指定する
CommitPrefix: {YYYYMMDD}_{連番}_ar

# 例：autoresearch が生成するコミット
20240315_003_ar: reduce learning rate warmup → val_loss: 0.412 (-0.008) ✓
20240315_004_ar: add dropout 0.1 → val_loss: 0.418 (+0.006) ✗ reverted
```

### 記録の対応関係

| 記録先 | 対象 | 読み方 |
|--------|------|--------|
| `outputs/{id}/metrics.json` | 手動実験 | ファイル読み込み |
| MLflow / W&B | 手動実験 | UIまたはAPI |
| `git log --grep="{YYYYMMDD}_ar"` | ARループ | git コマンド |
| `.autoresearch/results.tsv` | ARループ | ファイル読み込み |

### autoresearch 開始前にやること
1. 現在の状態を手動実験として記録しておく（ベースライン確定）
2. `git status` がクリーンなことを確認する
3. autoresearch が触ってよいファイルの Scope を明示する

---

## 再現性のための必須設定

```python
import random, numpy as np, torch

SEED = 42  # CLAUDE.mdの規定に従う

random.seed(SEED)
np.random.seed(SEED)
torch.manual_seed(SEED)
torch.cuda.manual_seed_all(SEED)
torch.backends.cudnn.deterministic = True
torch.backends.cudnn.benchmark = False
```
