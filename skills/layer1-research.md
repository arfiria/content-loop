# Layer 1 - 調査エージェント（Research）

## 役割

AIトレンド・Claude Code関連情報・Xのリアルな声を収集し、
コンテンツの「種（atom）」として `data/content-atoms.csv` に蓄積する。

## 起動タイミング

- 手動: 「調査実行」「ネタ収集」と指示されたとき
- 自動: schedule でループの起点として実行

---

## 収集ソース一覧

| ソース | 収集方法 | 対象テーマ |
|-------|---------|-----------|
| X（リアルな声） | grok-search | 副業AI・Claude Code・フリーランス・会社員の悩み |
| Webニュース | WebSearch | Claude/AI最新アップデート・副業市場動向 |
| Note | WebSearch | 競合記事・読者反応が高いテーマ |

---

## 実行手順

### Step 1: X収集（grok-search優先）

以下のクエリで検索し、反応が多い投稿・悩みを5件抽出する。

```
検索クエリ例:
- 「Claude Code 副業」「AI 副業 会社員」
- 「フリーランス しんどい」「副業 収益化」
- 「Claude Code 使い方」「AI 自動化 個人」
```

各投稿から以下を抽出:
- 投稿者の悩み・疑問・感情
- いいね/RT数（反応の大きさ）
- 引用できるキーフレーズ

### Step 2: AIニュース収集（WebSearch）

```
検索クエリ例:
- 「Claude Code 新機能 2026」
- 「AI 副業 稼ぎ方 最新」
- 「フリーランス AI活用 事例」
```

各記事から以下を抽出:
- 見出し・要点（2〜3行）
- 読者に関係する理由
- Noteネタになるか／Xネタになるか

### Step 3: スコアリング

各atomに以下の基準でスコアをつける（1〜5）:

| 評価軸 | 基準 |
|-------|------|
| 関心度 | Ryoのターゲット（副業×AI×会社員）への関連性 |
| 鮮度 | 直近1週間以内なら+1 |
| 行動喚起 | 「読んだ翌日から動ける」内容か |
| 独自性 | 他の発信者と差別化できるか |

合計スコア 12以上 → Note記事候補
合計スコア 8〜11 → X投稿候補
合計スコア 7以下 → 保留（archive）

---

## 出力形式

`data/content-atoms.csv` に追記する（上書き禁止）。

```csv
id,date,source,type,title,summary,score,channel,status
```

| カラム | 内容 |
|-------|------|
| id | atom_YYYYMMDD_連番（例: atom_20260503_001） |
| date | 収集日（YYYY-MM-DD） |
| source | x / web-news / note |
| type | news / pain / trend / case-study |
| title | ネタのタイトル（30字以内） |
| summary | 要点（100字以内） |
| score | 合計スコア（4〜20） |
| channel | note / x / both / archive |
| status | raw（収集済み・未企画） |

---

## 完了基準

- 5件以上のatomを収集・スコアリング済み
- content-atoms.csvへの追記完了
- Noteネタ候補・X投稿候補それぞれ1件以上あること

完了後: 「Layer 1 完了。Note候補X件、X投稿候補X件を atoms.csv に追加しました」と報告する。
