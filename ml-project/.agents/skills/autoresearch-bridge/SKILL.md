---
name: autoresearch-bridge
description: experiment-designerが設計した実験をautoresearchに委譲するブリッジ。「自律実行して」「夜間に回して」「autoresearchに渡して」「ループで最適化して」と言われたとき、または実験設定のGoal/Metric/Scopeが揃ったときに自動で使う。
---

# Autoresearch ブリッジ

experiment-designerが設計した実験をautoresearchの自律ループに渡す手順。

## 委譲できる条件（すべて揃っているか確認する）

| 条件 | 確認方法 |
|------|----------|
| ✅ Goal が1文で言える | 「〇〇を最小化/最大化する」 |
| ✅ Metric がシェルコマンド1行で取れる | `python eval.py \| grep "val_loss"` |
| ✅ Scope が限定されている | 特定ファイル・ディレクトリ |
| ✅ 実験1回が30分以内 | 長すぎるとループが回らない |
| ✅ autoresearch がインストール済み | `ls ~/.claude/skills/autoresearch/` |

揃っていない項目があれば、委譲せずに experiment-designer に差し戻す。

---

## 委譲プロトコル

experiment-designer の出力（`experiments/{id}_design.yaml`）から
autoresearch の入力形式に変換する。

### Step 1: design.yaml を読む

```yaml
# experiments/{id}_design.yaml から読み取る項目
purpose:    → Goal に変換
config:     → Scope・初期設定として使用
evaluation.metrics[0]: → Metric コマンドの基礎
```

### Step 2: autoresearch 設定に変換する

```
Goal:      {purpose をそのまま使う}
Metric:    {eval コマンド} | grep "{指標名}"
Scope:     {変更を許可するファイル・ディレクトリ}
Direction: higher_is_better / lower_is_better
Baseline:  {design.yaml の baseline 実験の結果値}
```

### Step 3: 実験記録との整合性を取る

autoresearch は git commit をメモリとして使う。
既存の実験管理（MLflow/W&B）と共存させるため、
`experiment-logging` スキルの命名規則を git commit メッセージに埋め込む。

```bash
# autoresearch が使う commit メッセージ形式を上書き指示する
# （autoresearch のデフォルトは "experiment: {description}"）
# → "{YYYYMMDD}_{連番}_ar_{description}" に統一するよう指示する
```

### Step 4: /autoresearch を呼び出す

以下のプロンプトを生成してユーザーに渡す（またはそのまま実行する）：

```
/autoresearch
Goal: {goal}
Metric: {metric_command}
Scope: {scope}
Direction: {direction}
Iterations: {推奨回数 - 夜間なら 50、確認用なら 5}
CommitPrefix: {YYYYMMDD}_{連番}_ar
```

---

## autoresearch がインストールされていない場合

```bash
# インストール方法をユーザーに案内する
git clone https://github.com/uditgoenka/autoresearch.git /tmp/autoresearch
cp -r /tmp/autoresearch/skills/autoresearch ~/.claude/skills/autoresearch
# Claude Code を再起動して /reload-plugins を実行
```

---

## 委譲できない場合の判断

以下の場合は autoresearch への委譲を避け、手動実験を推奨する：

- 実験1回が1時間以上かかる（ループが非効率）
- 評価指標がシェルコマンドで取れない（人間の主観評価が必要）
- data/raw/ への変更が必要（CLAUDE.md の禁止事項）
- 複数の指標を同時に最適化したい（autoresearch は1指標特化）
- まだベースラインが確立していない段階

この場合は「手動実験フロー（experiment-designer → 実行 → results-analyzer）」を継続する。
