# Layer 1 - 調査エージェント（Research）

## 役割

AIトレンド・Claude Code関連情報・Xのリアルな声を収集し、
コンテンツの「種（atom）」として `data/content-atoms.csv` に蓄積する。

## 起動タイミング

- 手動: 「調査実行」「ネタ収集」と指示されたとき
- 自動: schedule でループの起点として実行

---

## 実行環境による制約

| ツール | ローカル実行 | CCR（Routines自動実行） |
|--------|------------|----------------------|
| grok-search | 使用可 | **使用不可** → WebSearchで代替 |
| YouTube Data API | `.env`から読み込み | **Routineプロンプトにキー直接埋め込み** ✅ |
| yt-dlp | インストール済み | `pip install yt-dlp` で実行時インストール ✅ |
| WebSearch / WebFetch | 使用可 | 使用可 ✅ |

**CCR実行時（Routinesからの自動起動）の場合:**
- Step 1のX収集は grok-search の代わりに WebSearch を使う（クエリ: `site:twitter.com OR site:x.com "Claude Code" "副業"` など）
- Step 3（YouTube調査）はRoutineプロンプト内のAPIキーを使用。実行前に `pip install yt-dlp -q` でインストールする

---

## 収集ソース一覧

| ソース | 収集方法 | 対象テーマ |
|-------|---------|-----------|
| X（リアルな声） | grok-search | 副業AI・Claude Code・フリーランス・会社員の悩み |
| Webニュース | WebSearch | Claude/AI最新アップデート・副業市場動向 |
| Note | WebSearch | 競合記事・読者反応が高いテーマ |
| YouTube | YouTube Data API v3 + yt-dlp | AI副業・Claude Code解説動画の一次情報 |

---

## 実行手順

### Step 0: フィードバック参照（最初に必ず実行）

収集クエリを決める前に以下を確認し、収集の優先テーマを調整する。

1. `data/kpi_feedback.md` が存在すれば読む → 「成功パターン」のテーマ系統を優先収集
2. `data/content-outputs.csv` の直近3件のタイトルを確認 → 同テーマの連続を避ける

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

### Step 3: YouTube調査（YouTube Data API v3 + yt-dlp）

`.env` の `YOUTUBE_API_KEY` を使い、関連動画を検索して字幕から一次情報を取得する。

**3-1. 動画検索（Bashで実行）:**

```python
import urllib.request, json, urllib.parse

env = {}
with open('.env') as f:
    for line in f:
        line = line.strip()
        if line and not line.startswith('#') and '=' in line:
            k, v = line.split('=', 1)
            env[k.strip()] = v.strip()

api_key = env['YOUTUBE_API_KEY']
queries = ['Claude Code 副業', 'AI 副業 会社員']

for query in queries:
    url = (f"https://www.googleapis.com/youtube/v3/search"
           f"?part=snippet&q={urllib.parse.quote(query)}"
           f"&type=video&maxResults=3&regionCode=JP&relevanceLanguage=ja&key={api_key}")
    with urllib.request.urlopen(url) as res:
        data = json.loads(res.read())
    for item in data.get('items', []):
        print(item['id']['videoId'], item['snippet']['title'])
```

**3-2. 字幕取得（上位2〜3本。Bashで実行）:**

```bash
python -m yt_dlp --write-auto-sub --sub-lang ja --skip-download --sub-format vtt \
  -o "data/youtube_transcripts/%(id)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

字幕が日本語で取れない場合は `--sub-lang en` に切り替える。

**3-3. 字幕テキスト抽出:**

生成された `data/youtube_transcripts/*.vtt` を読み込み、タイムスタンプ行・インラインタグを除去して本文のみ抽出する:

```python
import re, sys
sys.stdout.reconfigure(encoding='utf-8')
text = open('data/youtube_transcripts/VIDEO_ID.ja.vtt', encoding='utf-8').read()
text = re.sub(r'<\d{2}:\d{2}:\d{2}\.\d+>', '', text)  # インラインタイムスタンプ除去
text = re.sub(r'</?c>', '', text)                       # <c>タグ除去
lines = [l.strip() for l in text.splitlines()]
lines = [l for l in lines if l and not re.match(r'^WEBVTT|^Kind:|^Language:|^\d{2}:\d{2}:\d{2}|^ -->', l)]
unique = list(dict.fromkeys(lines))
print('\n'.join(unique))
```

テキストから「読者の悩み」「具体的な数字・事例」「独自の視点」を3点抽出してatom summaryに使う。

字幕が取れなかった動画はタイトル・説明文のみでsummaryを作成し `source=youtube-meta` とする。

### Step 4: スコアリング

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
| source | x / web-news / note / youtube / youtube-meta |
| type | news / pain / trend / case-study |
| title | ネタのタイトル（30字以内） |
| summary | 要点（100字以内） |
| score | 合計スコア（4〜20） |
| channel | note / x / both / archive |
| status | raw（収集済み・未企画） |

---

## 完了基準

- 5件以上のatomを収集・スコアリング済み（YouTubeソースを1件以上含む）
- content-atoms.csvへの追記完了
- Noteネタ候補・X投稿候補それぞれ1件以上あること

完了後: 「Layer 1 完了。Note候補X件、X投稿候補X件を atoms.csv に追加しました」と報告する。
