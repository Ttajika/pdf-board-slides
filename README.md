# PDF Board Slides

PDF のスライドに、**先生の板書・ポインター・ページ送り（＋音声）を記録して、ブラウザ上で再生する**ための小さなビューアです。
動画を作らずに、「スライドがひとりでにめくれて、板書が書かれていく」オンデマンド教材を作れます。

- Beamer・PowerPoint・Keynote など、PDF にできるものなら何でも使える
- 板書は線のデータなので、拡大してもくっきり・動画よりずっと軽い
- 学生は再生中でも自分のペンで上から書き込める。止めて前のページを見ても「⏎ 再生位置に戻る」で続きへ
- サーバ側の処理は不要。静的ファイルを置くだけ（GitHub Pages でそのまま動く）

**デモ**：`index.html?file=slides/sample.pdf` を開き、2 ページ目左下の「▶ このトピックを再生」を押してください。

## ファイル構成

```
index.html        ビューア本体（PDF.js で表示＋学生の手書き）
slide-board.js    板書レイヤー・記録・再生のモジュール（ほかのスライドにも組み込める）
slides/           PDF を置く場所（sample.pdf はデモ）
board/            板書データ <PDF名>.board.json と音声ファイルを置く場所
```

## 使い方

### 学生に見せる

```
index.html?file=slides/lecture01.pdf        # #5 を付けると 5 ページ目から
```

| 操作 | 内容 |
|---|---|
| ◀ ▶ / ← → / スワイプ | ページ送り |
| ✏️ / D | 自分の手書き（ペン・蛍光ペン・消しゴム・ポインター）。Apple Pencil を当てると自動でオン |
| 👁 / T | 先生の板書の表示・非表示 |
| ▶✍ | このページの板書を書き順どおりに再生 |
| ▶ このトピックを再生 | 記録したトピックを再生（ページ送り・板書・ポインター・音声） |
| 💾 / 📂 | 自分の書き込みをファイルに書き出し・読み込み |

学生の書き込みはブラウザに自動保存されます（ブラウザのデータを消すと消えるので、残したいものは 💾 で書き出し）。

### 板書を作る（作成モード）

1. `index.html?file=slides/lecture01.pdf&author=1` を開く
2. 右上のパネルのペン ✏️ / 蛍光ペン 🖍 / 消しゴム 🧽 / ポインター 🔴 で書く
   （👆 にすると先生のペンが止まり、ページ側を普通に操作できる）
3. トピックの最初のページで **● 記録** → 話しながら書く・ポインターで指す・ページをめくる → **■ 停止**
   - 🎤 のチェックを外すと音声なしで記録（板書・ポインター・ページ送りのみ）
   - **⏸ 一時停止中に書いた線は、再生時に一瞬で現れる**（書き写しの時間を飛ばせる）
4. **⬇ 書き出し** で `<PDF名>.board.json` と音声ファイルがダウンロードされるので、`board/` に置く

作成中のデータはブラウザに自動保存されます（同じブラウザで学生モードを開くと、書き出す前でも確認できます）。
音声はメモリ上にしかないので、記録したら早めに書き出してください。

### 公開する（GitHub Pages）

リポジトリの Settings → Pages で `main` ブランチを公開すると、
`https://<ユーザー名>.github.io/<リポジトリ名>/?file=slides/lecture01.pdf` で見られます。
LMS（manaba など）からはこの URL にリンクするだけで使えます。

## 知っておくと良いこと

- **ページの対応**：各ページの文字から ID を作るので、PDF を差し替えてページが増減しても板書は正しいページに付いていきます。
  ただし、**そのページの文字を直すと ID が変わり、そのページの板書は外れます**。録り始めたら文言の修正は控えめに。
- **Beamer の `\pause`**：分割されたページは、記録すればページ送りとして再現されます。PDF のしおり（`\section`）はセクション選択に出ます。
- **音声形式**：Safari では m4a、Chrome では webm で録音されます。webm は古い Safari で再生できないことがあるので、
  学生が iPhone / iPad で見る場合は Safari で録音するか、m4a に変換して `board.json` のファイル名を書き換えてください。
- **ネット接続**：PDF.js を CDN から読み込むので、表示にはネット接続が必要です。
- PDF ファイルは `?file=` で指定した URL から読み込みます（同じサーバ上に置くのが確実です）。

## slide-board.js をほかのスライドに組み込む

`slide-board.js` はビューアに依存しない独立したモジュールです。HTML スライドなどにも、次のように組み込めます。

```html
<script src="slide-board.js"></script>
<script>
  SlideBoard.init({
    deck: 'lecture01',                   // board/<deck>.board.json
    getCanvas: () => myDrawCanvas,       // 学生用の手書き canvas（この上下に板書レイヤーを重ねる）
    getCurrent: () => currentIndex,      // 現在のページ（0 始まり）
    showSlide: (i) => showSlide(i),      // ページ移動（グローバル関数 showSlide / redrawStrokes を包んで連携する）
    getSlideCount: () => slides.length,
    getSlideId: (i) => slides[i].id,     // 任意：ページの安定 ID（無ければ番号）
    getSections: () => sections,         // 任意：[{title, slide}]（トピック名の既定値に使う）
    fixedLayout: false,                  // レイアウトが画面幅で変わる HTML スライドでは true（幅 1100px に固定）
  });
</script>
```

ページ側が `animctrl`（アニメ図の再生・停止・シーク）や `phantomctrl`（空欄の開閉）の CustomEvent を出すと、
それも記録・再生されます。データ形式は `slide-board.js` 冒頭のコメントにあります。

## ライセンス

MIT License（[LICENSE](LICENSE)）。PDF の表示には [PDF.js](https://mozilla.github.io/pdf.js/)（Apache License 2.0）を CDN から読み込んでいます。
サンプル PDF には IPA ゴシック（IPA フォントライセンス）を埋め込んでいます。
