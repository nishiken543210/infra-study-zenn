# AGENTS.md — infra-study Zenn 記事リポ

このリポで **記事を書く/編集するときは、必ず `ARTICLE-GUIDE.md` に従う**こと。

要点（詳細は `ARTICLE-GUIDE.md`）:
- 1記事 = 章の中の「1つの山場」粒度（300〜400行）。基準記事 = `articles/proxmox-linux-disk-serial-mapping.md`。
- frontmatter に書籍化メタ `book_part` / `book_chapter` / `book_order`（＋任意 `book_unit`）を必ず入れる（Part/Chapter を後で割り出すため）。
- タイトルは型「[読者像]でも[意外性]。[状況]。[対象]。」に従う。
- 本文：痛みから導入→マクロ→ミクロ／コマンドは出力ブロックとペア／なぜを都度／扱わない範囲を明示／まとめは1文。
- フッターは `ARTICLE-GUIDE.md` §5 を丸ごとコピペし Part/Chapter/URL だけ差し替え。
- **公開前セキュリティ grep（§4）をクリアするまで commit しない**（public リポ）。
- 新規は `published: false`（公開判断はオーナー）。`articles/INDEX.md` に1行追記。
