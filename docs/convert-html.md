# HTML 記事から Cosense 記法へ

`AGENTS.md` §6 が定める source ページを、HTML 記事から作るときの手続き。
**規約そのものは §6 にある。ここは ingest のときだけ読めばよい。**
以下の `§` は `AGENTS.md` の節番号を指す。

## 取得

**`fetch-proxy`（https://github.com/mpppk/fetch-proxy）で取得してから Cosense 記法に写す。**

`fetch-proxy` は Cloudflare Workers 上で任意の URL を取得するプロキシで、`as=md` で defuddle による本文抽出→Markdown 変換（失敗時は Browser Rendering に自動フォールバック）、`as=meta` でメタデータを JSON で返す。karpathy が勧める Obsidian Web Clipper の抽出エンジン（defuddle）をサーバ側で実行しつつ CORS 対応をまとめて扱えるため、直接 `defuddle.md` を呼ぶより安定する。

```
curl -sSL "https://fetch.nibo.sh/<host>/<path>?as=md"    # Markdown 本文
curl -sSL "https://fetch.nibo.sh/<host>/<path>?as=meta"  # メタデータ JSON（title / ogTitle / ogDescription / ogSiteName / ogImage）
curl -sSL "https://fetch.nibo.sh/<host>/<path>?as=html"  # 必要に応じ生 HTML
# 例: curl -sSL "https://fetch.nibo.sh/example.com/blog/post?as=md"
```

`as=md` は本文のみの Markdown を返す（host 版 defuddle のような frontmatter は付かない）。`as=meta` の `ogTitle`（無ければ `title`）で得たタイトルと合わせ、summary の Infobox に流す。`as=meta` に `author` と `published` は含まれないので、`as=html` で取得した HTML から補う（`<meta property="article:published_time">`、署名行、埋め込み JSON 等）。`source` は `url` に対応する。

**`as=title` は廃止された。** 呼ぶと本文の代わりに `as=title has been removed. use as=meta and read ogTitle, falling back to title` が返る（2026-08-15 実測）。

見出しのアンカー、目次、パンくず、ナビゲーション、コードのフェンス判定は `as=md` の時点で処理済みになる。

## Markdown から Cosense 記法へ

| Markdown | Cosense |
|---|---|
| `[text](url)` | `[text url]`（**外部リンク記法**） |
| `![alt](url)` | `(image) alt` の次行に `[url]`（埋め込み表示。§6 ファイル） |
| ` ``` ` フェンス | `code:<連番>.<拡張子>` + 各行をインデント |
| 表（`\|` 区切り） | `table:<連番>` + 各行をインデントし、セルをタブ区切り。`\|---\|` の区切り行は落とす |
| `**太字**` | `[* 太字]`。ただし中に `[` を含むときは記号だけ落とす（入れ子を避ける） |
| `*斜体*` | 記号を落として平文にする |
| `## 見出し` | 平文行。Cosense に見出し記法は無い |
| `> 引用` / `- 項目` | インデント 1 段 |

`[text](url)` を**外部リンク記法**に写すのが要点である。本文中のリンク先 URL を
保持したまま、リンクグラフには乗らないので concept ページの被リンクを汚さない（§4）。

`*斜体*` を忘れやすい。`**太字**` だけ処理すると、画像キャプションが生のアスタリスクで残る。

## fetch-proxy でも補う必要があるもの

`fetch-proxy` の `as=md` も内部では defuddle を使うため、2026-08-05 に 2 記事で実測した次の落としは同様に自分で拾うことがある。

- **タイトルが短縮されることがある。** `as=md` の先頭行ではなく **`as=meta` の `ogTitle`（無ければ `title`）** を使う。
  実測: `Your agent needs a computer, not a container` と出るが、
  og:title には `— introducing @cloudflare/computer` まで含まれる。§3 の「原題そのまま」に反する。
  `title` はサイト名が付いた `... | Cloudflare Blog` の形になることがあるので、`ogTitle` がある限りそちらを正とする。
- **引用の組織名が落ちる。** 発言者名と肩書は残るが、組織名はロゴ画像の
  `aria-label` / `alt` にあり、defuddle はロゴを装飾として捨てる。
  実測: 引用 17 件すべてで組織名が消えた。元 HTML（`as=html`）から拾い直し、
  `— 氏名 / 肩書 / 組織` の形に整える。
  **誰が言ったかは `credibility` と `caveats` に直結する**ので、ここは省略しない。
- **著者名は `as=md` からは取れない。** `as=md` は本文のみを返すので、`author` は常に
  元 HTML（`as=html`）の署名やメタタグから拾い直す。frontmatter が付く前提で書かない。
  本文に署名行が残っていれば `as=md` からも読めるが、当てにはできない。
  実測: Cloudflare 記事は本文の署名行ごと落ちた。Zenn の 3 記事は defuddle を直接呼んでいた頃も
  frontmatter に `author` を持たず、いずれも元 HTML から拾う必要があった。
- **埋め込みが `<iframe>` タグのまま残ることがある。** defuddle は中身を展開せず、`fetch-proxy` は Browser Rendering にフォールバックするが、それでも残る場合がある。
  実測: Zenn の mermaid 図とリンクカードが計 10 件、`<iframe src="embed.zenn.studio/...">` の
  1 行として残った。**図がそのまま失われる**ので、元 HTML（`as=html`）から拾い直す。
  Zenn の場合は iframe の `data-content` 属性に URL エンコードで実体が入っている。
  **mermaid は `[` を含むので必ず `code:<連番>.mermaid` ブロックに入れる**（下の「貼る前の点検」）。

## 貼る前に戻すエスケープ

**defuddle はコードブロック内のバックティックを `` \` `` とエスケープして出力する。**
`fetch-proxy` の `as=md` も内部では defuddle を使うため同様に発生する。
フェンス内でのエスケープは不要なので、これは defuddle 側の不具合である。
そのまま貼るとコードが壊れるので、**コードブロック内に限り `` \` `` を `` ` `` に戻す。**
実測: Cloudflare 記事で 6 箇所。テンプレートリテラルを含むコードがあると出る（MCP 記事では 0 件）。

**`\n` は戻さない。** JavaScript の文字列リテラルなど、原文が本来持っているエスケープである。
実測: 同じ記事に 4 箇所あり、戻すとコードの意味が変わる。

## fetch-proxy でも取得できないとき

`fetch-proxy` は `as=md` 取得失敗時に Browser Rendering にフォールバックするため、通常の JS レンダリングはそこで解決する。それでも取得できない場合（ログイン必須等）は自前で変換する。
素朴にタグを剥がすと次が静かに落ちる。

- 見出し末尾のアンカー（`<a class=anchor>#</a>` が残り `What changed#` になる）
- 引用の帰属（`blockquote` の外、`figcaption` / `img` の `alt` / `aria-label` にある）
- リンクのみの箇条書きの URL

**記事本体でないものは落とす。** 目次、パンくず、翻訳版へのリンク、"On this page" のような
ページ内ナビゲーション。これらはページの付属物であって原文の内容ではないので、
「省略しない」（§6 改変の範囲）の対象外である。

## 貼る前の点検

経路によらず、**変換後のテキストに `[` `]` バッククォート 行頭 `#` `#数字` が
残っていないか機械的に確認する。**

- **`[` `]` を含むコード片は `code:<名前>` ブロックに入れる。** インデントだけでは足りない。
  `backends: [` のような行がリンク記法として解釈され、コードが壊れた上に
  無意味な空リンクが大量に生成される。バッククォートも同じくコード記法と衝突する。
  `code:` の後の名前は記法上必須なので、原文に無くても付けてよい。これは構造であって改変ではない。
- 散文中に `[` が出たら個別にバッククォートで囲む。
- **散文中の対になったバッククォートは残す。** Cosense のインラインコード記法として正しく描画される。
  実測: `` `Workspace` `` `` `node:fs` `` などが defuddle から降りてくる。
  手でタグを剥がす経路ではここが平文に潰れていた。

## 画像 URL の扱い

**画像は拡張子で終わる URL を使う。** クエリパラメータは落とす。

- CDN のリサイズエンドポイント（`/_image?href=...&w=1430&f=webp` のような形）は
  拡張子がクエリの中にあるため、画像と認識されず表示されない。
  多くの場合 `href` パラメータに元画像の URL が入っているので、URL デコードして取り出す。
- パスが拡張子で終わっていてもクエリが後続する形（`.../xxx.png?sha=...` など）は、
  **クエリを落としてから使う。** 認識されるか賭けずに済む。
  落とす前に、クエリ無しの URL が画像として取得できることを確認する。
  実測: Zenn の `?sha=` は最適化版を指しており、有無で応答サイズが変わるが同じ図である。

いずれの場合もクエリパラメータが落ちる分、リンク切れにも強くなる。
