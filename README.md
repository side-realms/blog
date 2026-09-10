# 検品

Hugo 製の個人サイトです。トップページは背景の絵と「検品」の文字だけ。
検品はタイトルであって、テーマではありません。

## メニュー

| メニュー | 中身 | 置き場所 |
| --- | --- | --- |
| about | 経歴・業績（portfolio） | `content/portfolio.md` |
| blogs | ブログ記事（tech と other をまとめて新しい順に並べる） | `content/tech/*.md` ／ `content/other/*.md` |
| gallery | 入口から illust / music に分岐（ホワイトキューブ様式） | 曲: `assets/music/*.mp3` ／ 絵: `assets/gallery/`。ページは `layouts/gallery/` |
| links | その他のリンク | `config.toml` の `[[params.links]]` |
| life log | 見たものログ（年別） | `content/log/*.md` |

記事の URL は `/tech/...` `/other/...` のまま（旧サイトのリンクが切れないように）。
一覧だけを `/blogs/` にまとめています。

フォントは JetBrains Mono（和文はシステムのゴシック）。タイトル「検品」と
gallery 入口のみ WDXL Lubrifont JP N。

レイアウトは上部ヘッダー（左にサイト名、右にメニュー）＋中央寄せの本文カラム。
トップページのみ、背景の絵＋タイトル＋横並びメニュー。gallery 系は全幅の白。
絵のタイトルは `data/gallery.toml` で付けられます。

## 使い方

```sh
cd mysite
hugo server      # 開発サーバー → http://localhost:1313
hugo --minify    # public/ に静的サイトを出力
```

画像を縮小するので Hugo は **extended** 版が必要です。

## 更新のしかた

- **ブログを書く**：`content/tech/` か `content/other/` に `.md` を置く。

  ```md
  ---
  title: "記事タイトル"
  date: 2026-09-07T12:00:00+09:00
  draft: false
  math: true   # 数式（$...$ / $$...$$）を使うときだけ。KaTeX を読み込む
  ---
  ```

  `{{< youtube ID >}}`、`{{< audio src="foo.mp3" >}}` が使えます。
- **見たものログ**：`content/log/log_2026.md` に追記。年が変わったら新しいファイルを作る。
- **曲を足す**：`assets/music/` に `YYYY_MM_DD.mp3` を置く。波形は `data/waveforms.json`
  に同じ名前のキーで入れる（無ければ平らな波形になる）。
- **絵を足す**：`assets/gallery/` に `YYYYMMDD.png` などを置く。
  どちらもファイル名の日付で新しい順に並びます。
- **リンク**：`config.toml` の `[[params.links]]` を書き換える。
- **トップの絵**：`static/images/` に画像を置き、`assets/css/main.css` の
  `.home-main` の `background-image` のパスを変える（現在は `20260903.png`）。
  絵の明るさに合わせて `.home-title` と `.home-nav` の文字色も調整してください。

## カスタマイズ

- サイト名・メニュー：`config.toml`
- 色：`assets/css/main.css` の `:root`
- テンプレート：`layouts/`（`baseof.html` が全ページの外枠）

## 公開

`main` に push すると GitHub Actions が `hugo --minify` して `gh-pages` ブランチへ
デプロイします（`.github/workflows/pages.yml`）。
