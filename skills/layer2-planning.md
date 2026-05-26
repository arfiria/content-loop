# Layer 2 - 企画エージェント（Planning）

## 役割

`data/content-atoms.csv` のrawデータを読み込み、
Note記事・X投稿の企画案を作成してユーザー承認を取る。
承認後に `data/content-pipeline.csv` へ書き込む。

## 起動タイミング

- Layer 1 完了後に自動または手動で起動
- Routines（スケジュール）からの自動起動
- 「企画して」「コンテンツ決めて」と指示されたとき

## 実行モード

**スケジュール自動モード**（Routinesから起動 または「自動モードで企画して」と指示された場合）:
- Step 1〜3 を実行して企画案を `data/pending-approval.md` に保存
- pipeline.csv に `status = pending-approval` で追記
- AskUserQuestion は使わない
- 完了後に「企画案を pending-approval.md に保存しました。確認・承認後に Layer 3 を実行してください」と報告して停止

**手動インタラクティブモード**（デフォルト）:
- 従来どおり AskUserQuestion で承認を取ってから pipeline.csv に書き込む

---

## 実行手順

### Step 1: atoms.csv の読み込み

`data/content-atoms.csv` を読み込み、`status = raw` のものを抽出する。

### Step 2: Note記事の企画（channel = note / both）

スコア上位のatomからNote記事企画を最大3案作成する。

各案に以下を含める:

```markdown
## 企画案 [N]
- タイトル案（3パターン）
- ターゲット読者: （具体的な人物像）
- 記事の構成（H2見出し3〜5本）
- 冒頭フック（読者が「これ自分のことだ」と感じる一文）
- 結論・読者へのアクション
- 推定文字数: X,000字
- 元atom ID: atom_XXXXXXXX_XXX
```

タイトルは以下の型から選ぶ:
- 「〇〇している人が△△を実現した方法」
- 「〇〇だった私が△△できるようになった理由」
- 「〇〇するための△△ステップ」

### Step 3: X投稿の企画（channel = x / both）

スコア上位のatomからX投稿企画を最大5案作成する。

各案に以下を含める:

```markdown
## X企画案 [N]
- 投稿の核心メッセージ（一言）
- フック（1行目。スクロールが止まる一文）
- 投稿形式: 単発ツイート / スレッド（N枚） / 引用RT
- 元atom ID: atom_XXXXXXXX_XXX
```

### Step 4: 承認フロー（モードによって分岐）

**手動インタラクティブモードの場合:**

企画案を提示し、AskUserQuestion で承認を取る。

承認パターン:
- 「このまま進める」→ 全案をapprovedに
- 「1と3だけ」→ 指定した案のみapprovedに
- 「1を修正して」→ 修正後に再確認
- 「全部やり直し」→ Step 2 からやり直し

**スケジュール自動モードの場合:**

企画案を `data/pending-approval.md` に以下の形式で保存して停止する。

```markdown
# 企画案（承認待ち）: YYYY-MM-DD

承認したい案の `status` を `approved` に変更後、「Layer 3 実行」と入力してください。

## 企画案 1
status: pending
[企画案の内容]

## 企画案 2
status: pending
[企画案の内容]
```

### Step 5: pipeline.csv への書き込み

承認された企画を `data/content-pipeline.csv` に追記する（上書き禁止）。

- 手動モード: 承認後すぐに `status = approved` で追記
- 自動モード: `status = pending-approval` で追記（Ryoが承認後に手動で `approved` に変更）

```csv
id,atom_id,date,type,title,target,structure,hook,status,assigned_layer
```

| カラム | 内容 |
|-------|------|
| id | content_YYYYMMDD_連番（例: content_20260503_001） |
| atom_id | 元のatom ID |
| date | 企画日（YYYY-MM-DD） |
| type | note-article / x-post / x-thread |
| title | 確定タイトル |
| target | ターゲット読者の一言説明 |
| structure | 記事構成（セミコロン区切り） |
| hook | 冒頭フック文 |
| status | approved（Layer 3 待ち） |
| assigned_layer | 3 |

---

## 完了基準

- 承認済みコンテンツが1件以上 pipeline.csv に追記されている
- atoms.csvの該当atomのstatusを `planned` に更新済み

完了後: 「Layer 2 完了。Note記事X件、X投稿X件を pipeline.csv に追加しました」と報告する。
