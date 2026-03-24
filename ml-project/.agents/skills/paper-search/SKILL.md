---
name: paper-search
description: 機械学習・深層学習の論文検索手順。arXiv・Semantic Scholar・Papers with Codeを使って調査する。paper-researcherエージェントが自動で使う。
---

# 論文検索 手順

## 検索ソースと使い分け

| ソース | URL | 使い道 |
|--------|-----|--------|
| arXiv | https://arxiv.org/search/ | 最新プレプリント・cs.LG / cs.AI / stat.ML |
| Semantic Scholar | https://api.semanticscholar.org/graph/v1/paper/search | 引用数・影響力の確認 |
| Papers with Code | https://paperswithcode.com/sota | SoTA比較・実装コードの確認 |
| Google Scholar | https://scholar.google.com | 網羅的な文献検索 |

## 検索手順

### Step 1: SoTAの把握
Papers with Code でタスク名を検索して、現在のベンチマーク上位手法を確認する。
```
https://paperswithcode.com/task/{task-name}
```

### Step 2: キーワード検索
arXiv で複数のキーワードパターンで検索する：
- 手法名そのまま
- タスク名 + "deep learning"
- 問題設定の説明

### Step 3: 被引用論文の追跡
重要論文が見つかったら、Semantic Scholar でその論文を引用している論文を確認する：
```
https://api.semanticscholar.org/graph/v1/paper/{paperId}/citations
```

### Step 4: 実装の確認
Papers with Code で実装コードが公開されているか確認する。

---

## 論文の読み方（優先順位付き）

### 最初に読む箇所（5分）
1. Abstract（何をやったか）
2. Figure 1（手法の概要図）
3. Results table（どのデータセットで何点か）

### 次に読む箇所（15分）
4. Introduction（なぜこの問題か・何が課題だったか）
5. Method（どうやっているか）
6. Conclusion（何が言えるか・今後の課題）

### 詳細が必要なとき
7. Related Work（先行研究との違い）
8. Experiments（実験設定の詳細）
9. Appendix（実装の詳細・追加実験）

---

## 信頼性の評価

高信頼：
- 査読付き会議（NeurIPS / ICML / ICLR / CVPR / ACL 等）
- 引用数が多い（100以上）
- 公式実装がある

要注意：
- arXivプレプリントのみ（査読未通過）
- 引用数が少ない新しい論文
- 実装がなく再現困難

---

## 調査の落とし穴

- 論文のタイトルや概要だけで判断しない（結果テーブルを確認する）
- データセットの違いに注意（同じ指標でも比較不能な場合がある）
- 存在しない論文をでっち上げない（必ずURLで実在確認する）
