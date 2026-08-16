# projects

このディレクトリで管理する Cosense プロジェクトを定義する。各サブディレクトリが 1 プロジェクトに対応する。

## 一覧

- `niki-auth/` — https://scrapbox.io/niki-auth — 認証認可に関するナレッジを管理するプロジェクト
- `niki-ai/` — https://scrapbox.io/niki-ai — AIエージェントやLLMに関するナレッジを管理するプロジェクト
- `niki-cs/` — https://scrapbox.io/niki-cs — カスタマーサポートを行うSaaSに関するナレッジを管理するプロジェクト
- `niki-tech/` — https://scrapbox.io/niki-tech — 認証認可やAI以外の技術一般に関するナレッジを管理するプロジェクト

## 構成

```
projects/
  niki-auth/
    config.json  # プロジェクトのメタデータ（URL等）
  niki-ai/
    config.json
  niki-cs/
    config.json
  niki-tech/
    config.json
```

- `AGENTS.md` が全プロジェクト共通の規約。プロジェクト固有の事情があれば `projects/<name>/` 下に追記する（例: `projects/niki-auth/AGENTS.md` や `notes.md`）。
- wiki 本体は常に Cosense 側にある。このリポジトリには置かない。
- 新しい Cosense プロジェクトを管理対象に加える時は、このディレクトリに `<name>/config.json` を追加し、上の一覧と `AGENTS.md` 冒頭の一覧に追記する。projectUrl を列挙している箇所（`AGENTS.md` §4 のリンク規約と §12 lint、`docs/lint.md` 末尾）も併せて直す。
