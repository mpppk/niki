# lint の検査表

`AGENTS.md` §13 の lint（健全性チェック）で回す検査の一覧。
**何が健全かを定めるのは規約側（§4 のリンク規約、§6 の source 規約 等）で、
ここはその検査手順だけを持つ。** lint のときだけ読めばよい。
以下の `§` は `AGENTS.md` の節番号を指す。

Cosense では機械的に検出できる。定期的に実行する。

| 検査 | コマンド |
|---|---|
| 未処理の指示 | `cosense list1hopLinks <projectUrl>/ingest` `.../query` `.../lint` の被リンク（§12） |
| マーカーの付け忘れ | `cosense searchFullText <projectUrl> 'yuki.icon'` のうち、行頭にマーカーの無い行。実行はせず次の会話で確認する（§12） |
| 処理済み指示の残骸 | 同じ検索結果のうち、`ingest [yuki.icon]` のように平文化した操作名と署名しか残っていない行。実行後に情報を持たない指示は行ごと消すのが規約なので（§12）、消し忘れとして片付ける。子行に結果リンクがあれば、その行の位置に繰り上げてから消す |
| 孤立ページ | `cosense listPages <projectUrl> --sort linked` の末尾（`linked: 0`）。入口ページ `[このwikiについて]`、`[log]` の日付ページ、操作ページ `[ingest]` `[query]` `[lint]`、プロフィールページ `[yuki]` は被リンクを持たないのが正常なので除く |
| 取り込み漏れの原文 | 上記のうち source 層のもの（§6 識別）で、対応する `[summary]` が未作成のもの |
| リンク未付与の原文 | `#raw` ページのうち `cosense list1hopLinks` が summary 1 本しか返さないもの。空リンクは現れないので、疑わしければ本文を読む（§9）。`#bookmark` は本文を持たず、bookmark summary はリンクを自身の `takeaways` に持つので、どちらも対象外 |
| 育ちすぎたハブ | `--sort linked` の先頭。pageRank 上位ページは分割を検討する |
| 書くべきページ | `searchVector` の `exists: false`、および空リンクの被リンク数 |
| 未解決の矛盾 | `cosense searchFullText <projectUrl> '⚠'`。source 層は除く（§10） |
| ソースの網羅性 | `cosense browseRelatedPages <projectUrl>/summary` の表を眺める |
| 未読の参照先 | `cosense searchFullText <projectUrl> 'references'` で `references` 節を持つ summary を引き、まだ ingest していない URL を「次に読むべきソース」の候補にする（§7） |
| 確信度の陳腐化 | `cosense browseRelatedPages <projectUrl>/thesis` の表で `reviewed` が古いもの |
| アイデアの陳腐化 | `cosense browseRelatedPages <projectUrl>/idea` の表で `status` が `open` かつ `reviewed` が古いもの。`done` / `dropped` は対象外（§8） |
| 根拠の付かないアイデア | 同じ表で `enabled_by` も `blocked_by` も空のまま古いもの。そのアイデアに効くソースをまだ 1 つも読んでいないということなので、「次に読むべきソース」の提案につなげる（§8） |
| 宙に浮いた論点 | `[idea]` の `open_questions` にある空リンクのうち、被リンクがそのアイデア 1 本だけのもの。「次に立てるべき問い」の候補になる（§8） |

各検査は `projects/` 下で定義されたプロジェクトごとに実行する（`<projectUrl>` は `https://scrapbox.io/niki-auth` / `https://scrapbox.io/niki-ai` / `https://scrapbox.io/niki-cs` / `https://scrapbox.io/niki-tech`）。

lint では検出だけでなく、**次に読むべきソースと、次に立てるべき問いを提案する。**
