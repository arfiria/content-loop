# Layer 5 - 公開・分析エージェント（Publish & Analytics）

## 役割

QAパス済みコンテンツの公開準備を完了させ、
公開記録を `data/content-outputs.csv` に残す。
蓄積データを分析して次の企画（Layer 1/2）にフィードバックする。

## 起動タイミング

- Layer 4 の qa-passed 確認後に自動または手動で起動
- 「分析して」「週次レポート」「パフォーマンス確認」と指示されたとき

---

## 公開準備（qa-passed コンテンツに対して実行）

### Note記事の公開チェックリスト

```markdown
- [ ] タイトル最終確認（SEOキーワードが含まれているか）
- [ ] 冒頭フックの確認（最初の3行で掴めているか）
- [ ] 見出し構造の確認（H1 > H2 > H3 が正しいか）
- [ ] 内部リンク（過去記事への誘導）の追加
- [ ] CTA（次のアクション）が末尾にあるか
- [ ] サムネイル画像の準備（/thumbnail で生成済みか）
- [ ] タグ・カテゴリの設定（副業 / AI / Claude Code / 会社員 等）
```

### X投稿の公開チェックリスト

```markdown
- [ ] 140字チェック（単発ツイートの場合）
- [ ] スレッドの場合: 1投稿目のフックが強いか
- [ ] ハッシュタグ: 2〜3個（多すぎない）
- [ ] Note記事との連携（記事公開後に誘導Xを投稿する順番か）
- [ ] 投稿最適時間: 平日7〜8時 or 21〜22時 / 休日10〜12時
```

---

## 公開記録（content-outputs.csv への追記）

公開または公開準備完了時に追記する。

```csv
id,content_id,published_date,type,title,platform,url,status
```

| カラム | 内容 |
|-------|------|
| id | output_YYYYMMDD_連番 |
| content_id | pipeline.csv の content ID |
| published_date | 公開日（YYYY-MM-DD）または予定日 |
| type | note-article / x-post / x-thread |
| title | 公開タイトル |
| platform | note / x |
| url | 公開後に手動で記入（空欄でOK） |
| status | ready（公開準備完了）/ published（公開済み）|

pipeline.csvの該当IDのstatusを `published` または `ready-to-publish` に更新する。

---

## 分析（週次・月次）

### 週次分析（毎週月曜に実行）

`data/content-outputs.csv` を読み込み以下を集計:

```markdown
## 週次コンテンツレポート: YYYY-W[XX]

### 制作実績
- Note記事: X本（公開済み X本 / 準備中 X本）
- X投稿: X件
- 合計制作物: X点

### atoms.csv → outputs.csv の転換率
- 収集atom数: X件
- 企画採用数: X件（採用率 XX%）
- 公開数: X件（完走率 XX%）

### ボトルネック
- Layer [N] で止まっているコンテンツ: X件
- 差し戻し（QA rejected）: X件

### 来週への引き継ぎ
- 継続制作中: [content_id一覧]
- 優先度高atomで未企画: [atom_id一覧]
```

分析結果を `reviews/YYYY-W[XX].md` に保存する。

### 月次収益連携

月末に以下を `revenue/YYYY-MM.md` に追記する:

```markdown
## コンテンツ × 収益 月次サマリー: YYYY-MM

### 発信実績
- Note記事公開: X本
- X投稿: X件

### 収益への影響（手動記入欄）
- Note収益: ¥
- 副業問い合わせ件数: X件（X件成約）
- フォロワー増加数: +X人

### 来月のコンテンツ方針
[分析から得た示唆を記載]
```

---

## フィードバックループ

分析後、以下を実行して次のサイクルに繋げる:

1. `data/content-atoms.csv` の `status = planned` を `completed` に更新
2. 特に反応が良かったテーマを `ideas/` に「高反応テーマ」としてメモ保存
3. Layer 1 への示唆をINBOXに追記:
   - 「〇〇系のネタは反応が良い傾向 → 次回の調査クエリに追加」

---

## 完了基準

- 全 `qa-passed` コンテンツの公開チェックリスト完了
- content-outputs.csv への追記完了
- pipeline.csvのstatus更新完了
- 週次の場合: reviews/ にレポート保存済み

完了後: 「Layer 5 完了。公開準備X件完了。週次レポートを reviews/ に保存しました」と報告する。
