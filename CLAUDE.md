# content-loop - コンテンツ自動生成システム

## 概要

「会社員 × 副業 × AI」をテーマにしたNote記事・X投稿を毎日1本生成するコンテンツループ。
6レイヤーのエージェントがCSVを介してバトンを渡す仕組み。

## 発信者プロフィール（コンテンツ執筆時に参照）

- **X**: @UMInoOTO_26
- **Note**: https://note.com/umioto_lab
- **立ち位置**: 通信業界の会社員 + Claude Code副業フリーランス（兼業）
- **発信テーマ**: AI副業・Claude Code活用・会社員が収入を増やす方法
- **読者像**: 副業に興味があるが最初の一歩が踏み出せていない会社員
- **文体**: 結論ファースト、敬語なし、具体例・数字必須、読んだ翌日から動ける内容

## ディレクトリ構成

```
content-loop/
├── CLAUDE.md              ← このファイル
├── skills/                ← 各レイヤーの実行手順
│   ├── layer1-research.md
│   ├── layer2-planning.md
│   ├── layer3-creation.md
│   ├── layer4-qa.md
│   ├── layer5-analytics.md
│   └── layer6-weekly-review.md
├── data/                  ← CSVデータ（ループの状態管理）
│   ├── content-atoms.csv      ← 調査済みネタ一覧
│   ├── content-pipeline.csv   ← 企画〜公開の進捗管理
│   ├── content-outputs.csv    ← 公開記録
│   └── pending-approval.md    ← Layer 2 の承認待ち企画案
├── projects/              ← 生成された記事・X投稿・修正版
└── reviews/               ← 週次レポート・SEOレビュー
```

## 実行フロー

```
【毎朝5:00 自動起動】
Layer 1 調査    → data/content-atoms.csv に追記
Layer 2 企画    → data/pending-approval.md に保存して停止
        ↓ Ryo が status を approved に変更
        ↓ 「Layer 3 実行」で再開
Layer 3 制作    → projects/ にファイル生成
Layer 4 QA      → pipeline.csv の status 更新
Layer 5 公開    → content-outputs.csv に追記・公開チェックリスト完了

【毎週日曜 自動起動】
Layer 6 レビュー → 過去7日の記事をSEO採点
              → projects/revised-note-*.md をコピペパッケージで生成
              → reviews/ に週次SEOレポート保存
```

## 承認ルール

**自動実行OK:**
- Layer 1: atoms.csv への追記（常に）
- Layer 2: pending-approval.md への保存（スケジュール自動モード時）
- Layer 3: projects/ へのファイル生成（status = approved のみ）
- Layer 4: QAチェック（読み取り・判定のみ）
- Layer 5: outputs.csv への追記（status = qa-passed のみ）
- Layer 6: SEOスコアリング・修正ドラフト生成（常に）
- reviews/ へのレポート保存（常に）

**確認必須（自動実行禁止）:**
- Layer 2: 企画案の採用（手動モード時）→ AskUserQuestion で確認
- Layer 3: X投稿の最終文面確定
- QA rejected 時の修正方針
- 既存ファイルの上書き・削除（常に禁止、追記のみ）

## 起動コマンド

```
「コンテンツループ実行」     → Layer 1〜5 を順番に実行
「Layer [N] 実行」           → 指定レイヤーのみ単独実行
「今週のコンテンツ状況」     → pipeline.csv を読んで進捗サマリー表示
「週次レビュー実行」         → Layer 6 を手動実行
「週次レポート生成」         → Layer 5 の分析モードを実行
```
