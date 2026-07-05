# Raindrop.io 自動分類システム AIワークフロー

## 1. 目的

Raindrop.ioを「第二の脳」として運用するための分類システム。
目的は単なる自動振り分けではなく、以下の状態を作ること。

- 後から探せる
- タグで横断検索できる
- デザイン・実装・生活系資料を再利用できる
- Obsidianへ昇格すべき判断資産を選別できる

---

## 2. 日常分類パイプライン

```text
Raindrop（保存）
  ↓
Make.com（自動化ワークフロー）
  ↓
Jina Reader（URL本文取得）
  ↓
Claude Haiku（分類AI・JSON出力）
  ↓
JSON Parse
  ↓
Raindrop Update（folder / tags / note 書き戻し）
```

- **Raindrop**: 一次倉庫
- **Make**: 日常保存時の自動実行基盤
- **Jina Reader**: URL本文取得
- **Claude Haiku**: 新規保存物の暫定分類
- **Raindrop Update**: 分類結果の書き戻し
- **Obsidian**: 厳選した判断資産の保管先

---

## 3. 正本ファイル

分類ロジックの正本は以下3ファイル。

1. `raindrop_folder_config.md`
   - フォルダ名
   - Collection ID
   - フォルダ分類ルール
2. `raindrop_allowed_tags_v1.md`
   - 許可タグ
   - 禁止タグ
   - 正規化ルール
   - タグ付けルール
3. `raindrop_rules_v3_1.md`
   - 月次棚卸し思想
   - 自動決定木
   - confidence / 要確認ルール
   - Obsidian候補判定
   - 出力方針

矛盾時の優先順位：

```text
1. raindrop_folder_config.md
2. raindrop_allowed_tags_v1.md
3. raindrop_rules_v3_1.md
```

---

## 4. 基本思想

- フォルダ = 住所（1記事につき1つ）
- タグ = 検索索引（フォルダを横断して探すための名詞）
- 日常分類では未整理へ逃げない。情報不足でも暫定分類し、必要なら `要確認` を付ける
- `要確認` は停止フラグではなく低信頼フラグ
- `要確認` 単独付与は禁止。必ず内容タグを併用する

---

## 5. AI役割分担

| AI | 役割 | やらないこと |
|---|---|---|
| チャッピー | 採否判断、収束、スコープ制御、push判断 | ファイル編集、実装、勝手なpush |
| Claude Code | 承認済みdocs修正、prompt draft作成、validation、commit | 独断push、分類判断、未承認の設計変更 |
| Codex | 差分監査、P0/P1/P2分類、commit/push前レビュー | 実装、分類実行 |
| Claude Haiku | Make上の日常自動分類 | 正本変更、ルール設計 |
| Fable5 | 大規模IA監査・構造棚卸し専用 | 月次常設担当、個別URL再分類 |
| Gemini | 必要時の比較検証・外部視点・別案検証 | 現行標準の分類担当ではない |

---

## 6. 月次棚卸しフロー

### Phase 1: CSV棚卸し

入力:
- Raindrop CSV
- 正本3ファイル
- 直近の誤分類メモ

確認対象:
- 許可外タグ
- 禁止タグ残存
- 表記ゆれ
- タグなし
- `要確認` 滞留
- `ネットショップ` 単独タグ
- 重複URL
- フォルダ分布の偏り
- Obsidian候補の過不足

### Phase 2: チャッピー判断

分類:
- 採用
- 修正採用
- 保留
- 却下

出力:
- docs修正指示
- CSV修正指示
- Codex要否
- commit / push判断方針

### Phase 3: Claude Code編集

対象:
- 承認済みdocs修正
- `.gitignore` など安全整備
- Make/Haiku用prompt draft作成
- 必要に応じたCSV整形

禁止:
- 未承認のCSV削除
- 未承認のRaindrop更新
- Make設定変更
- `git add .`
- 独断push

### Phase 4: Codex監査

確認:
- 対象外ファイル混入
- 正本間の矛盾
- 許可タグ外の混入
- 決定木の参照ズレ
- 仕様と差分の一致
- commit / push可否

出力:
- Verdict: PASS / PASS_WITH_NOTES / FAIL
- P0 / P1 / P2
- Final recommendation

---

## 7. このrepoで触ってよいもの

通常の正本更新対象:
- `raindrop_folder_config.md`
- `raindrop_allowed_tags_v1.md`
- `raindrop_rules_v3_1.md`

承認済みの場合のみ触ってよいもの:
- `.gitignore`
- `docs/*.md`
- Make/Haiku用prompt draft
- 棚卸しレポート

触ってはいけないもの:
- 未承認のCSVデータ
- Makeシナリオ設定
- API設定
- Raindrop実データ
- 未承認のプロンプト本番反映

---

## 8. 直近の完了済み変更

### `44fab20 docs: align raindrop classification rules`

反映済み:
1. `ツール` 分岐を決定木に追加
2. カメラ境界を一本化
3. SNS新規保存ルールと月次棚卸し維持ルールの優先順位を明文化
4. 許可外タグ例を修正
5. `ネットショップ` 単独タグ禁止を追加
6. Note欄ルールを現実運用に合わせて緩和

Codex PASS後、GitHub `main` にpush済み。

### `28daa23 chore: add .gitignore and Make/Haiku prompt draft`

反映済み:
1. `.gitignore` に `CSV/` と `.DS_Store` を追加
2. `docs/make_haiku_prompt_draft.md` を新規作成
3. Make/Haiku日常分類用prompt草案を追加

Codex PASS後、GitHub `main` にpush済み。

---

## 9. 禁止事項

- `git add .` 禁止
- CSVを勝手に削除しない
- CSVを勝手にcommitしない
- Make設定を勝手に変更しない
- Raindrop実データを変更しない
- 正本3ファイルを無断で追加修正しない
- P2 polishを混ぜない
- push前Codexレビューなしでpushしない

---

## 10. 他AIレビュー時の起点コマンド

```bash
git status -sb
git log --oneline -3
git show --stat --oneline HEAD
find . -name ".DS_Store" -print
git diff --check
```

stageは明示pathのみ。

```bash
git add path/to/file
```

`git add .` は禁止。
