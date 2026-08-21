# Open Knowledge Format v0.2 との比較

Google Cloud の [Open Knowledge Format](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) v0.2 と、
`AGENTS.md` が定める現行運用を突き合わせた記録である。

**これは規約ではない。** `docs/` の他の 2 つ（`convert-html.md` / `lint.md`）が操作の手続きを持つのに対し、
ここにあるのは採否を判断するための材料だけで、`AGENTS.md` を上書きしない。
規約を変えるならこの文書ではなく `AGENTS.md` を直す。意図的に `AGENTS.md` からは参照していない。

以下、`§` は `AGENTS.md` の節番号、`OKF §` は SPEC.md の節番号を指す。
比較時点: 2026-08-21。OKF は v0.2（v0.1 を supersede、破壊的変更 2 件を含む minor bump）。
niki 側は 4 プロジェクト計 698 ページ（niki-auth 297 / niki-ai 247 / niki-tech 98 / niki-cs 56）。

## 0. 結論

- **層が違うので「どちらを採るか」ではない。** OKF は組織間で知識を**交換**するためのフォーマットで、
  未知の producer が書いたバンドルを consumer が機械的に読めることを目的にしている。
  niki は書き手が LLM 1 つ・読み手が人間 1 人に閉じた**運用規約**で、
  Cosense の被リンクとリンクグラフという基盤機能に強く依存している。
  OKF が規定しない層（何を書くか、いつ書くか、どう探すか）が niki の大半を占める。
- **思想は同じ karpathy の系譜にあり、重要な設計判断がいくつも一致している**（§3）。
  型を 1 つだけ持たせること、関係をリンクで表しリンク自体には型を持たせないこと、
  壊れたリンクを「まだ書かれていない知識」として許容すること、log を持つこと。
  独立に同じ結論に達している以上、これらは基盤に依存しない性質だと考えてよい。
- **決定的な差は 1 つだけである: メタデータが構文で読めるか、LLM に読ませるか。**
  OKF の frontmatter は producer が書いた値がそのまま consumer に渡る。
  niki の Infobox は、本文に書いた値を Cosense の LLM が**抽出し直す**。
  §5 が列挙する制約（1 分のラグ、リンクしただけのページへの捏造、角括弧落ち、書き換え後の反映漏れ）と、
  型定義ページの Infobox 2 列目に書かれた抽出指示は、すべてこの一点から出た税である。
  実物を見ると分かりやすい ── `summary` 型定義ページの `gist` 行は
  「本文の gist 行をそのまま使う。要約し直さない。書かれていなければ空欄とせよ。null とは書かない。」で、
  **これは frontmatter があれば 1 文字も要らない。**
- **取り込む価値があるのは 3 つ**（§7）: `verified`（人間が確認した印）、
  `stale_after`（陳腐化の判定を判断から比較にする）、ページ単位の `status`。
  いずれも Cosense に置いたまま、Infobox のキーとして足せる。
- **取り込まない方がよいものも 3 つある**: `index.md`（§4 で既に禁止しており、被リンクがあるので不要）、
  Attested Computation（リサーチ wiki に検証すべき計算値が無い）、
  `credibility` の信号分解（niki の値には §12 の運用規則が乗っている）。

## 1. 出自と目的

|  | OKF v0.2 | niki |
|---|---|---|
| 目的 | 知識の**交換**。producer と consumer の契約を決める | 特定テーマの**継続的な深掘り**。thesis を育てる |
| 想定する書き手 | 人・エージェント・エクスポート pipeline（不特定） | LLM 1 つ（§0） |
| 想定する読み手 | エージェント・UI・検索インデックス・決定的なコード（不特定） | 人間 1 人（§0） |
| 実体 | ディレクトリツリーの markdown ファイル。git 配布推奨（OKF §3） | Cosense のページ。リポジトリには置かない（§1） |
| 階層 | ディレクトリ。ドメインからは独立 | 無し。完全にフラットで、階層はリンクで表す（§1） |
| 規定する範囲 | フォーマットのみ。保存・配信・クエリ基盤は non-goal（OKF §1） | フォーマットに加えて操作・探索順序・指示の受け方まで |
| 語彙 | 開いている。`type` は中央登録しない（OKF §4.1） | 閉じている。型 11 種、値の候補も列挙（§0, §2） |
| 準拠の条件 | 3 つだけ（OKF §11） | 準拠という概念を持たない。lint が健全性を見る（§13） |

OKF の非目標が niki の中身とちょうど噛み合う。OKF は「固定された型の分類を定義しない」「保存・配信・クエリ基盤を規定しない」と明言しており、
niki が持っている型 11 種・3 操作・検索順序・指示記法は、**OKF に寄せても失われない**。上に載るだけである。

## 2. 対応表

| OKF v0.2 | niki | 一致度 |
|---|---|---|
| Knowledge Bundle | Cosense プロジェクト 1 つ（niki は 4 つ） | ○ |
| Concept | ページ | ○ |
| Concept ID（バンドル内のパス） | ページタイトル（§3 で表記ゆれ対策を持つ） | ○ |
| Frontmatter（YAML、構文でパース） | 本文 1 行目の型 + `table:infobox`（LLM が抽出） | **×** |
| Body | 本文 | ○ |
| `type`（唯一の必須） | 本文 1 行目の型 | ○（語彙の開閉が違う） |
| `title` | ページタイトル | ○ |
| `description` | Infobox `gist` | △（summary / idea のみ。thesis・concept は持たない） |
| `resource` | Infobox `url` | ○ |
| `tags` | 相当なし。リンクで代用（§4 でハッシュタグを制限） | △ |
| `sources[]` | `📄` raw / `🔖` bookmark ページ + Infobox `raw` / `url` / `author` / `published` | △（niki は 1 summary = 1 source） |
| `sources[].id` + 本文 footnote による claim 単位の出典 | 相当なし（`quotes` 節が近い） | × |
| credibility signals（`author` / `usage_count` / `last_modified`） | Infobox `credibility`（6 値の判定） | **思想が逆**（§5 参照） |
| `usage_window` | 相当なし | × |
| `generated: { by, at }` | Infobox `ingested`（日付のみ） | △（`by` は規約で LLM に固定されているので書いていない） |
| `verified: [{ by, at }]` | **相当なし** | × |
| trust tier（unverified / machine-confirmed / human-reviewed） | **相当なし**（`confidence` は軸が違う） | × |
| `status: draft / stable / deprecated` | idea の `status: open / done / dropped` のみ | △（意味が違う） |
| `stale_after`（絶対時刻） | 相当なし。lint の「確信度の陳腐化」が代替（`docs/lint.md`） | × |
| actor 規約（`human:<id>` / `<producer>/<version>` / `process:<id>`） | `[yuki.icon]` 署名。**無署名であること自体が署名**（§12） | △（逆向きの解） |
| markdown link（種類は散文が担う、untyped edge） | `[ ]` リンク | ○ |
| 壊れたリンクの許容（"not-yet-written knowledge"） | 空リンク＝次に書くべきページのキュー（§4） | **◎ 完全一致** |
| `index.md`（progressive disclosure） | **禁止**（§4「目次ページ・index ページを作らない」） | **正面から対立** |
| `log.md`（日付見出し、新しい順） | `[log]` 型の日付ページ（1 日 1 ページ、§11） | △（分解の仕方が違う） |
| `references/` 慣習（外部素材をバンドル内の concept にする） | `📄` raw ページ（原文全文を転記、§6） | ○ |
| Attested Computation 一式 | 相当なし | ドメイン外 |
| 予約ファイル名（`index.md` / `log.md`） | 型名と内容ページの衝突回避（§2） | △（同じ問題への別解） |
| — | `supported_by` / `refuted_by` / `enabled_by` / `blocked_by` | **niki のみ** |
| — | 被リンク（Cosense が自動表示） | **niki のみ** |
| — | `⚠` による行単位の矛盾注記（§10） | **niki のみ** |
| — | ingest / query / lint と wiki 上の指示記法（§12, §13） | **niki のみ**（OKF の non-goal） |

## 3. 一致している設計判断

独立に同じ結論へ達している箇所を挙げる。基盤に依存しない性質だと考えてよい部分である。

- **型を 1 つだけ持たせる。** OKF は `type` を唯一の必須フィールドにし、consumer は未知の型を許容しなければならない。
  niki は本文 1 行目に型を 1 つだけ置き、これがフォルダの代わりになる（§2）。
  どちらも「型がルーティングの単位、それ以外は自由」という同じ骨格を選んでいる。
- **リンク自体には型を持たせない。** OKF は「A から B へのリンクは関係を主張するが、その種類は周囲の散文が伝える」と明示する（OKF §6.1）。
  niki の §4 も同じで、リンクは「そこから辿りたいもの」に限る、としか言っていない。
- **壊れたリンクは異常ではない。** OKF は「consumer は壊れたリンクを許容しなければならない。それは単にまだ書かれていない知識かもしれない」と書く。
  niki の空リンク＝キューはこれと完全に同じ発想で、しかも niki は lint の「書くべきページ」検査でそれを積極的に使っている（`docs/lint.md`）。
  **OKF が許容に留めた性質を、niki は機能に変えている。**
- **階層はドメインから独立。** OKF は「ディレクトリ構造はドメインから独立で、producer が好きに整理する」と言い、
  niki は「階層・分類はすべてリンクで表現する」と言う（§1）。表現が逆に見えるが、
  どちらも「ファイルの置き場所に意味を持たせない」という同じ制約である。この一致のおかげで §4 の写像が成立する。
- **機械が持てるものは機械に持たせる。** OKF は index.md を producer が自動生成してもよい／consumer がその場で合成してもよいとし、
  niki は「手書きの目次は更新漏れで必ず腐る」として被リンクと Infobox に任せる（§4）。
  **原則は同じで、基盤の違いから結論だけが逆になっている。** Cosense には被リンクがあるので index が要らず、
  ファイルツリーには被リンクが無いので index.md が要る。
- **生成物と一次資料を区別する。** OKF は `generated.by` に actor を書かせ、`human:` prefix で trust tier を分ける。
  niki は `credibility: generated` を置き、さらに「generated なソースを thesis の `supported_by` / `refuted_by` に入れてはならない」という
  強い運用規則を乗せている（§12）。**動機は同じで、niki の方が踏み込んでいる。**

## 4. 構造的に写せないもの

- **Cosense のページに YAML frontmatter を置く場所が無い。** したがって
  **Cosense に置いたままでは、niki は原理的に OKF 非準拠である。**
  準拠は書き出し時の変換でしか達成できない（§8）。これは運用の欠陥ではなく、基盤の選択の帰結である。
- **変換するとき、Infobox は本文から直接パースしなければならない。**
  型定義ページの表（`browseRelatedPages` が返すもの）を使ってはならない。
  §5 の実測どおり、**本文には正しい値が書かれており、壊れるのは抽出結果の方だから**である。
  角括弧が落ちた `author`、リンクしただけのページに生えた偽の `gist` / `raw` / `url`、
  指示を書き換えた後に古い値が残った行 ── いずれも本文を読めば起きない。
- **フラットな名前空間は障害にならない。** OKF はディレクトリ構造を要求しないので、
  全ページをバンドル直下に並べても conformance の 3 条件を満たす。
- **タイトル先頭の `📄` / `🔖` は写し方に選択が要る。** OKF の concept ID はファイルパスなので絵文字を含められるが、
  素直なのは `type: raw` / `type: bookmark` で層を表し、絵文字付きの原題は `title` に置く形である。
  ただしそれをすると §3 の「タイトルが実質的な ID」が変換後に崩れるので、
  変換器がタイトルとファイル名の写像表を持つ必要がある。

## 5. OKF が持っていて niki に無いもの

1. **`verified` と trust tier。** OKF は「誰が書いたか」（`generated`）と「誰が確認したか」（`verified`）を明確に分け、
   `human:` actor による確認があるかどうかで unverified / machine-confirmed / human-reviewed を導出させる。
   **niki にはこれが無い。** §0 で「wiki は LLM が書き、人間は読む」と決めた以上、
   OKF の目で見れば **698 ページすべてが unverified** である。
   thesis と idea の `reviewed` は「確信度／状態を見直した日」であって、見直すのは LLM なので `verified` ではない。
   人間が読んで裏を取ったページと、まだ誰も見ていないページが、wiki 上で区別できない。
2. **`stale_after`。** OKF は絶対時刻を書かせ、「`now >= stale_after` なら stale」という単純比較にしている。
   相対 TTL を避けたのは、読んだ時刻を参照しないで済ませるためだと明記されている。
   niki の陳腐化検査は「`reviewed` が古いもの」（`docs/lint.md`）で、**閾値が規約のどこにも無い。**
   毎回 lint する側が判断していることになる。
3. **ページ単位の lifecycle。** niki が持つ失効の表現は `⚠` の矛盾注記だけで、これは**行単位**である（§10）。
   ソースそのものが撤回された、仕様が後継に置き換わった、という**ページ単位**の失効を表せない。
   OKF の `deprecated`（「リンクと履歴のために残すが、もう現行ではない」）は、
   niki の「古い記述を消さない」という方針とそのまま噛み合う。
4. **summary 以外の provenance。** OKF ではどの concept も一様に `sources` を持てる。
   niki で Infobox を持つのは summary / thesis / idea だけで、**concept / synthesis / question / person / organization は出自を持たない。**
   とくに synthesis は定義上 2 つ以上のソースを跨ぐ型なのに（§2）、
   どのソースから来たかは本文のリンクを読むしかない。
5. **claim 単位の出典。** OKF は footnote のラベルを `sources[].id` に一致させ、
   「エージェントが常に書き換えるので位置ではなくキーで参照する」と理由まで書いている。
   niki は 1 summary = 1 source なので summary 内では不要だが、
   複数ソースを跨ぐ synthesis と question では、この仕組みが効く場所がある。

## 6. niki が持っていて OKF に無いもの

1. **極性を持つリンク。** `supported_by` / `refuted_by` / `enabled_by` / `blocked_by` が niki の中心機能である。
   OKF はリンクの種類を散文に委ねているので（OKF §6.1）、**標準の語彙では表現できない。**
   producer 定義の追加キーとして書けば conformant のままだが（consumer は未知のキーを拒否してはならない）、
   交換相手には意味が伝わらない。
2. **被リンク。** OKF に概念が無い。consumer が全ファイルを走査すれば作れるが、フォーマットは何も約束しない。
   niki の「目次を作らない」「関連ページ節を手書きしない」「未処理の指示はマーカーの被リンクで引く」は、
   **すべて被リンクが自動で見えることに依存している。**
3. **`⚠` の矛盾注記。** 古い記述を消さずに矛盾を隣に積む運用（§10）と、`searchFullText '⚠'` による棚卸し。
   OKF には矛盾という状態が無い。
4. **人間の指示を corpus 自体に置く記法。** `[query] ... [yuki.icon]` と操作ページの被リンクキュー（§12）。
   OKF は producer と consumer が分離した世界を前提にしており、
   **読み手が知識ベースの中に指示を書き込む**という運用を想定していない。
5. **3 操作。** ingest / query / lint。OKF の non-goal に明示的に含まれる領域である。
6. **閉じた語彙と、それを守るための規則。** 型名を小文字・単数形の英語に固定し、表記ゆれを別名ページに寄せる（§0, §3）。
   OKF は「`type` は中央登録しない」と正反対を選んでいるが、これは交換規格として当然の選択で、
   **書き手が 1 つしかいない niki では閉じられる**という違いに過ぎない。

## 7. 取り込むなら

提案であって規約ではない。採否はユーザーが決める。優先順に並べる。

**(a) summary に `verified` を足す（最も効く）**

値は `YYYY-MM-DD` の 1 つでよい。actor は 1 人しかいないので OKF のような `by` は要らない。
空欄が「まだ人間が見ていない」を意味し、OKF の unverified / human-reviewed の区別がそのまま入る。
日付は壊れにくい方の値である。§5 が記録している破損（捏造・角括弧落ち・`null` の残留）は、いずれも日付キーでは報告されていない。
**ただし Infobox のキーを増やすと、その型にリンクしている全ページの抽出が走り直す**（§5）。
反映は遅く、全行には及ばない。足すなら値の揺れが落ち着くまで表を検証に使わない。

**(b) thesis と idea に `stale_after` を足す**

`reviewed` の隣に絶対日付を置く。lint の「確信度の陳腐化」「アイデアの陳腐化」が、
`reviewed` が古いかどうかの**判断**から、`stale_after <= today` の**比較**になる。
`docs/lint.md` の 2 行がそのまま機械的になる。

**(c) ページ単位の `status` は保留でよい**

`⚠` の注記で足りている面があり、値を増やすと更新されない欄が増える。
§8 で idea の `status` に対して「進捗は表さない、lint が絞るための最小限」と既に同じ判断をしているので、
入れるなら `deprecated` 相当の 1 値だけを足す形が規約の一貫性を保つ。

**見送るもの**

- **`index.md` 相当** ── §4 で既に禁止済み。被リンクがある以上、根拠が変わっていない。
- **Attested Computation** ── リサーチ wiki に「サンクションされた計算で出した数値」が存在しない。
  ただし動機は無関係ではない。**「エージェントの出力ではなく一次証拠に遡れること」**は
  niki が `📄` に原文全文を転記して引用の裏取りを可能にしている理由と同じである（§6）。実装が違うだけで思想は近い。
- **`credibility` の信号分解** ── OKF は「スコアは主観的で、consumer 間で可搬でなく、陳腐化する」として
  客観的な信号（`author` / `usage_count` / `last_modified`）だけを記録する立場を取る。
  一見 niki の `credibility` を否定しているように見えるが、
  niki の 6 値は**スコアではなくソースの種別**であり、しかも `generated` には §12 の運用規則（thesis の根拠にしてはならない）が乗っている。
  信号に分解すると、この規則を書く場所が消える。
- **`tags`** ── §4 でハッシュタグ記法を `#raw` / `#bookmark` に限定した判断を覆すことになる。リンクで代替できている。

## 8. OKF バンドルとして書き出すなら

まだ実装しない。写像だけ置く。

```
bundles/niki-auth/
  index.md                     # okf_version: "0.2" を持つ唯一の frontmatter
  log.md                       # [log] の日付ページを 1 ファイルに畳む
  summaries/<title>.md         # type: summary
  concepts/<title>.md          # type: concept
  theses/<title>.md            # type: thesis
  ...
  references/<title>.md        # 📄 raw / 🔖 bookmark
```

| niki | OKF frontmatter |
|---|---|
| 本文 1 行目の型 | `type:`（`summary` / `concept` / `thesis` / … をそのまま使う。OKF は型を登録しない） |
| ページタイトル | `title:` |
| Infobox `gist` | `description:` |
| Infobox `url` | `resource:` |
| Infobox `ingested` | `generated: { by: <LLM の actor>, at: <ingested> }` |
| Infobox `raw` | `sources: [{ id, resource: /references/…, title }]` |
| Infobox `author` / `published` / `credibility` | `sources[].author` / `sources[].last_modified` ＋ 拡張キー `credibility` |
| Infobox `supported_by` / `refuted_by` / `enabled_by` / `blocked_by` | 拡張キーとしてそのまま（標準では表現できない。§6） |
| Infobox `confidence` / `status` / `reviewed` | 拡張キーとしてそのまま |
| `[ ]` リンク | バンドル相対の markdown リンク（`/concepts/….md`。OKF が推奨する形） |
| 空リンク | 壊れたリンクのまま出す。**OKF は明示的に許容している** |

注意点は 3 つ。

1. **Infobox は本文からパースする。**`browseRelatedPages` の表を使わない（§4 の理由）。
2. **conformance の条件は 3 つだけ**（frontmatter がパースできる / `type` が非空 / `index.md` と `log.md` が規定の形）。
   `type` さえ書けば準拠するので、残りは段階的に足せる。最初の変換器で全部を写す必要はない。
3. **`index.md` と `log.md` は予約名で、concept にしてはならない。**
   niki 側に `index` や `log` というタイトルの内容ページを作ると衝突する（§2 の「型名と内容ページの衝突」と同じ問題）。

## 参照

- [Open Knowledge Format SPEC.md v0.2](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) — 本比較の対象。自己完結した仕様書
- [GoogleCloudPlatform/knowledge-catalog](https://github.com/GoogleCloudPlatform/knowledge-catalog) — OKF を含むリポジトリ
- [karpathy の gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — niki と OKF の共通の出自
