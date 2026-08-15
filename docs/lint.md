# lint の検査表

`AGENTS.md` §12 の lint（健全性チェック）で回す検査の一覧。
**何が健全かを定めるのは規約側（§4 のリンク規約、§6 の source 規約 等）で、
ここはその検査手順だけを持つ。** lint のときだけ読めばよい。
以下の `§` は `AGENTS.md` の節番号を指す。

Cosense では機械的に検出できる。定期的に実行する。

| 検査 | コマンド |
|---|---|
| 未処理の指示 | `cosense list1hopLinks <projectUrl>/ingest` `.../query` `.../lint` の被リンク（§11） |
| マーカーの付け忘れ | `cosense searchFullText <projectUrl> 'yuki.icon'` のうち、行頭にマーカーの無い行。実行はせず次の会話で確認する（§11） |
| 孤立ページ | `cosense listPages <projectUrl> --sort linked` の末尾（`linked: 0`）。入口ページ `[このwikiについて]`、`[log]` の日付ページ、操作ページ `[ingest]` `[query]` `[lint]`、プロフィールページ `[yuki]` は被リンクを持たないのが正常なので除く |
| 取り込み漏れの原文 | 上記のうち source 層のもの（§6 識別）で、対応する `[summary]` が未作成のもの |
| リンク未付与の原文 | `#raw` ページのうち `cosense list1hopLinks` が summary 1 本しか返さないもの。空リンクは現れないので、疑わしければ本文を読む（§8）。`#bookmark` は本文を持たず、bookmark summary はリンクを自身の `takeaways` に持つので、どちらも対象外 |
| 育ちすぎたハブ | `--sort linked` の先頭。pageRank 上位ページは分割を検討する |
| 書くべきページ | `searchVector` の `exists: false`、および空リンクの被リンク数 |
| 未解決の矛盾 | `cosense searchFullText <projectUrl> '⚠'`。source 層は除く（§9） |
| ソースの網羅性 | `cosense browseRelatedPages <projectUrl>/summary` の表を眺める |
| 確信度の陳腐化 | `cosense browseRelatedPages <projectUrl>/thesis` の表で `reviewed` が古いもの |

各検査は `projects/` 下で定義されたプロジェクトごとに実行する（`<projectUrl>` は `https://scrapbox.io/niki-auth` / `https://scrapbox.io/niki-ai` / `https://scrapbox.io/niki-cs`）。

lint では検出だけでなく、**次に読むべきソースと、次に立てるべき問いを提案する。**
