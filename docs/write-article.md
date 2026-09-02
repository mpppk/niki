# 骨子から Zenn 記事へ

`AGENTS.md` §9 が定める `[article]` の骨子を、記事として mpppk/zenn-contents に出すときの手続き。
**規約そのものは §9 と §14 draft にある。ここは `status` が `ready` の骨子を記事にするときだけ読めばよい。**
以下の `§` は `AGENTS.md` の節番号を指す。

## 出す先

- リポジトリ: https://github.com/mpppk/zenn-contents（ローカルは `~/ghq/github.com/mpppk/zenn-contents`）
- デフォルトブランチは `master`。**GitHub Actions は無く、Zenn 側の連携が push を検知してデプロイする**
- 記事は `articles/<slug>.md` の 1 ファイル。画像はリポジトリ直下の `images/` だけが対象
- 詳細な制約はあちらの `README.md` が持つ。**ここに写さない**

## 手順

### 1. slug を決める

Zenn の制約は **半角小文字英数字・ハイフン・アンダースコアで 12〜50 字**。
骨子のページ名は日本語なのでそのままでは使えない。主題を英語で表し、
**内容が時点に依存するなら年を入れる**（例: `agent-harness-comparison-2026`）。
公開後は変えられない（URL が変わる）ので、ここで確定させる。

### 2. ブランチを切る

```sh
cd ~/ghq/github.com/mpppk/zenn-contents
git switch master && git pull
git switch -c article/<slug>
```

### 3. 雛形を置く

```sh
cp templates/article-explainer.md articles/<slug>.md   # 使い方と思想の解説
cp templates/article-comparison.md articles/<slug>.md  # 複数を比較する記事
```

どちらにも当てはまらないなら、frontmatter だけ写して**骨子の `outline` から直接組む**。
`yarn new:article` でも作れるが、返るのは frontmatter だけの雛形である。

### 4. frontmatter を埋める

```yaml
---
title: ""       # 骨子の outline で確定させた記事タイトル。ここが正本になる
emoji: "📝"     # 1 文字
type: "tech"    # tech: 技術記事 / idea: アイデア
topics: []      # 小文字英数字。最大 5 つ
published: false
---
```

**`published: false` のまま書き始める。** デプロイされること自体は公開を意味しない。
ここを埋めた時点で、記事タイトル・`emoji`・`topics` の正本は frontmatter に移る。
骨子の `outline` にタイトル案が残っていれば消す（§9）。

### 5. 本文を書く

骨子の `outline` の順に書く。守ることが 4 つ。

- **Cosense 記法を持ち込まない。** `[X]` は Zenn ではただの角括弧になる。
  リンクにしたいものだけ `[X](url)` にし、残りは普通の文にする。強調は `**`
- **引用の裏取りは `📄` raw ページに降りて取る**（§10 手順 5）。
  summary の `quotes` に無い引用を使うときは、必ず raw ページで原文を確認する。
  **孫引きをしない**のが、wiki を持っていることの実利である
- **Infobox の表の値を本文に貼らない。** リンク記法の角括弧が落ちることがある（§5 制約 4）。
  値が要るときはリンク元ページの本文を読む
- 図表を使うなら `images/` に置き、`/images/...` の**絶対パス**で参照する。
  `.png` `.jpg` `.jpeg` `.gif` `.webp` のみ、1 ファイル 3MB 以内

Zenn 独自記法で使うもの: `:::message` / `:::details` の囲み、単独行に置いた URL のリンクカード。

### 6. 「参考」節を作る

骨子で使った `[summary]` の Infobox `url` から作る。
**wiki にある根拠を全部並べない。** 読者がその先を読む価値があるものだけを残す
（`references` の考え方と同じ。§7）。

### 7. 確認して PR を出す

```sh
yarn install   # 初回のみ
yarn preview   # http://localhost:8000
```

```sh
git add articles/<slug>.md
git commit
git push -u origin article/<slug>
gh pr create
```

**PR 本文に骨子ページの Cosense URL を貼る。** どの wiki の知識から起こした記事かが辿れる。

PR を作ると Zenn 側で**下書きデプロイ**が走り、実物のレンダリングを確認できる。
制約は 3 つ ── 同一リポジトリの PR のみ、デプロイされるのは `articles/` と新規 `images/` のみ、
**公開済み・予約公開済みの記事は更新されない**。

### 8. マージ後に wiki を直す

- 骨子ページの `slug` を埋め、`status` を `done`、`reviewed` を当日にする（§14 draft）
- 日付ページに `draft [<article のページ名>]` を 1 行（§12）

### 9. 公開する

**`published: true` にするのは常に別 PR で、ユーザーの明示指示があるときだけ。**
外向きの不可逆な操作であり、`status: done` は「記事になった」であって「公開した」ではない（§9）。

## 書きながら wiki に戻すもの

**記事にしか書かれていない知識を作らない。** 執筆中に次のことが起きたら、記事を書き終える前に戻す。

| 起きたこと | 戻し先 |
|---|---|
| 外部を調べた | 調査結果ページとして ingest する（§13） |
| 新しいソースを読んだ | ingest する（§14）。`references` に URL だけ残して先送りしてもよい |
| 「つまりこう言える」が出た | `[thesis]` に立て、骨子の `claim` から参照する（§2） |
| 根拠が足りないと分かった | 骨子の `open_questions` に「未 ingest」として置く |

執筆は **wiki の穴が最も見えるタイミング**である。見つけた穴を記事の中だけで埋めると、
その知識は次の記事から引けなくなる。

## 貼る前の点検

- [ ] `published: false` になっているか
- [ ] Cosense 記法の `[ ]` が本文に残っていないか
- [ ] 引用は raw ページで原文を確認したか
- [ ] `topics` は 5 つ以内で、Zenn 側に存在する表記か
- [ ] 画像の参照は `/images/` から始まる絶対パスか
- [ ] 骨子の `outline` に書いた `not_covering` を、記事が越えていないか
