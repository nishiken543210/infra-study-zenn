# AGENTS.md — infra-study Zenn 記事リポ

このリポで **記事を書く/編集するときは、必ず `ARTICLE-GUIDE.md`（構造）と `VOICE-PROFILE.md`（文体）の両方に従う**こと。

要点（詳細は `ARTICLE-GUIDE.md`）:
- **文体は `VOICE-PROFILE.md` が正本**。書き手は「解説する人」ではなく「やってみた人」。1文40〜60字・1段落2〜4文・心の声を独立行で置く・「〜が重要です」「本記事では〜解説します」等は禁止。
- 🔴 **書く前に `VOICE-PROFILE.md` §8（差し戻された 3 つの型）と §9（実測ベースライン）を読む**。
  ①主語がない／自己紹介から始める ②意図の伝わらない内部用語 ③主題が 1 文で言えない —— の 3 つで実際に 5 回差し戻された。
  **どれも lint は BLOCK 0 で通る。** 段落の目標値（120 字以上を 5% 以上・標準偏差 30 以上）は §9 の実測表から逆算している。
- 読者は infra-study を何も知らない前提で書く。内部ID・内部ファイル名・内部ホスト名・「前に決めたとおり」は書かない（`VOICE-PROFILE.md` §6）。
- 1記事 = 章の中の「1つの山場」粒度（300〜400行）。基準記事 = `articles/proxmox-linux-disk-serial-mapping.md`。
- frontmatter に書籍化メタ `book_part` / `book_chapter` / `book_order`（＋任意 `book_unit`）を必ず入れる（Part/Chapter を後で割り出すため）。
- タイトルは型「[読者像]でも[意外性]。[状況]。[対象]。」に従う。
- 本文：痛みから導入→マクロ→ミクロ／コマンドは出力ブロックとペア／なぜを都度／扱わない範囲を明示／まとめは1文。
- フッターは `ARTICLE-GUIDE.md` §5 を丸ごとコピペし Part/Chapter/URL だけ差し替え。
- **公開前 lint（§4）で BLOCK が 0 になるまで commit しない**（public リポ）:
  `bash outputs/scripts/qa/article-lint.sh zenn-content/articles/<slug>.md`
- 新規は `published: false`（公開判断はオーナー）。`articles/INDEX.md` に1行追記。
