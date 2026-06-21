# 記事テンプレート & 粒度ガイド（infra-study Zenn）

infra-study のおまけ記事を **同じ粒度・同じ型** で量産するための規格。
人間も Codex もこの1ファイルに従って書く。基準記事は `articles/proxmox-linux-disk-serial-mapping.md`。

> 目的：①更新頻度を上げる（本数を増やす）②全記事を将来 **Part / Chapter 単位でまとめ直せる**（総集編・書籍化）ように、各記事へ機械可読のメタを持たせる。

---

## 0. 1記事の粒度（最重要）

- **1記事 = 章の中の「1つの山場・1つの判断/作業」**。テーマを広げない。
  - ✅「同容量ディスクが5本あるとき、拡張対象をどう特定するか」
  - ❌「Proxmoxのディスク管理ぜんぶ」
- 作業時間にして **10〜30分相当** の、自己完結した problem → solution。
- **マニアック寄りOK**（"インフラでも意外と知らない" ニッチを歓迎）。
- 文量目安：**markdown 300〜400行**（frontmatter＋本文＋検証ログ＋フッター込み）。

---

## 1. frontmatter 規格

```yaml
---
title: "（下の「タイトルの型」に従う）"
emoji: "💽"                 # 記事内容に合う絵文字1つ
type: "tech"
topics: ["Linux", "server", "infra", "初心者", "command"]  # 最大5・公開表示される
published: false           # 下書き=false。公開判断はオーナー
# --- 書籍化メタ（Zennは無視・まとめ抽出専用）---
book_part: 1               # PART番号（整数）
book_chapter: 1            # CHAPTER番号（整数）
book_order: 3              # 章内の並び順（整数・まとめ時の章内順序）
book_unit: "linux-basics"  # まとめ単位スラッグ（任意・章をまたぐ束ね用）
---
```

- `book_*` は **Zenn が解釈しない独自キー**（公開に影響しない・無視される）。これで後から Part/Chapter を割り出せる。
- 本編の章に紐づかない集客記事などは `book_part: 0 / book_chapter: 0`、`book_unit` で分類（例 `intro`）。
- **抽出例**（まとめ時）：
  ```bash
  # 全記事の Part/Chapter/順序を一覧
  grep -rl '^book_part:' articles/ | while read f; do
    awk -F': ' '/^book_part:/{p=$2}/^book_chapter:/{c=$2}/^book_order:/{o=$2}/^title:/{t=$0}
      END{printf "P%s C%s #%s  %s  %s\n",p,c,o,FILENAME,t}' "$f"
  done | sort
  ```

---

## 2. タイトルの型

> `[読者像]でも[意外性]。[具体的な状況・痛み]。[対象コマンド/技術]。`

- `[読者像]`：インフラエンジニア / 駆け出しインフラ など
- `[意外性]`：迷う / 意外と知らない / 説明できない / つまずく
- `[状況]`：具体的なシーン1つ（数字や現場感を入れる）
- 例：
  - インフラエンジニアでも迷う。100GBの仮想ディスクが5本あるとき、Proxmoxで拡張対象をどう特定する？
  - インフラエンジニアでも意外と知らない。ネットが使えない現場で頼れるLinuxの`man`コマンド。

---

## 3. 本文の構成（11点・★=必須）

1. ★**導入＝具体的な状況と痛み**から入る。抽象論・コマンド羅列で始めない。
2. ★**マクロ→ミクロ**：何を・なぜ、を先に置いてから手順へ。
3. ★**手順は「コマンド＋実際の出力ブロック」を必ずペア**で示す（再現可能に）。
4. ★**「なぜそうするか」を都度説明**。単なる手順書にしない（判断の根拠を書く）。
5. **mermaid 図**で対応関係・流れを可視化（`<br/>` を使う。`\n` は不可）。
6. **コールアウト**で注意を分離：`:::message`（補足）/ `:::message alert`（警告）。
7. ★**扱わない範囲を明示**してスコープを締める（広がりそうな所を切り出す）。
8. **補足セクション**で代替手段・脇道を「任意」に降格。
9. **実機検証ログ**を `:::details` アコーディオンに格納（※下のセキュリティ必須）。
10. ★**まとめは原則レベルの学び**（1文で言い切る）。
11. ★**フッター**（§5 を丸ごとコピペ・Part/Chapter部分だけ差し替え）。

トーン：制作ログ寄りの一人称・会話調。**断定しすぎず、不確実性に正直**。教科書解説ではなく「現場でどう判断したか」。

---

## 4. 公開前セキュリティチェック（push前に必須・grep）

公開リポ（public）なので、下記が**1件でも出たらマスク/除去してから**コミット。

- 内部IP（`192.168.` `172.16.` `10.x`）/ MAC（`94:40:c9` 等）
- ユーザー名・固有名（`nishiken` / `nk` / `ops` / `alice` / `root@`）
- 内部ホスト名（`learner01` `workstation01` `l3router01` `proxmox` 等）
- 認証情報（パスワード・APIキー・`lab-XXXX`）/ Worker URL（`*.workers.dev`）
- 禁止用語（「シミュレーター」）/ 内部連番（`S-01` などの社内ID）

```bash
grep -nEi '192\.168\.|172\.16\.|10\.[0-9]+\.|[0-9a-f]{2}(:[0-9a-f]{2}){5}|nishiken|workers\.dev|\b(nk|ops|alice)\b|root@|lab-[0-9a-z]|S-[0-9]{2}|シミュレーター' articles/<slug>.md
```

実機ログを載せる場合は、識別に不要な実値（IP/ユーザー名/MAC/ホスト名）を**伏せるか汎用化**してから `:::details` に入れる。

---

## 5. フッター雛形（コピペ用）

`{PART}` `{CHAPTER}` `{CHAPTER_URL}` の3か所だけ各記事で差し替える（frontmatter の `book_part`/`book_chapter` と一致させる）。

```markdown
---

## ご紹介

感想をもとにコンテンツを改善しながら、いずれは実機サーバーに触れる演習も用意していく予定です。

まずは Linux の基礎を読んでみてください。

**Linux の基礎:** [infra-study.org/{CHAPTER_URL}](https://infra-study.org/{CHAPTER_URL})

**感想フォーム:** [感想フォーム（Tally）](https://tally.so/r/eqvX8l)

---

この記事は、infra-study「Part {PART} / Chapter {CHAPTER}」でLinuxの基礎を読むときの、おまけとして作成しています。

よかったら見てみてください。

X（@taro3_01）で更新を告知しています。フォローすると次回の通知が届きます。

@[card](https://x.com/taro3_01)
```

例：Part 1 / Chapter 1 → `{CHAPTER_URL}` = `part1-chapter01`。

---

## 6. ファイル名（slug）規約

- `articles/<slug>.md`。slug は **英小文字・数字・ハイフン**、内容が分かる名前（例 `linux-man-command-offline-help`）。
- Zenn 制約：12〜50文字 `^[a-z0-9\-_]{12,50}$`。
- 追加したら `articles/INDEX.md` に wikilink を1行足す（内部管理用・`.gitignore` 済）。

---

## 7. チェックリスト（記事を出す前に）

- [ ] タイトルが §2 の型
- [ ] frontmatter に `book_part`/`book_chapter`/`book_order` がある
- [ ] §3 の★必須を満たす（導入の痛み / コマンド＋出力ペア / なぜ / 扱わない範囲 / まとめ / フッター）
- [ ] フッターの Part/Chapter/URL が frontmatter と一致
- [ ] §4 セキュリティ grep がクリーン
- [ ] `published: false`（公開判断はオーナー）
