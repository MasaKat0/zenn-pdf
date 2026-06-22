# Zenn PDF Tool

公開されているZennの記事URLを指定し，その記事本文をPDFとして保存するCLIツールです。

対象URLは次の形式です。

```text
https://zenn.dev/<user>/articles/<slug>
```

## 事前準備

Node.js 20以上を使います。最初に依存パッケージとPlaywright用のChromiumを入れます。

```bash
npm install
npx playwright install chromium
```

## PDF化の手順

PDF化したいZenn記事のURLを，次のコマンドの末尾に入れて実行します。

```bash
npm run pdf -- https://zenn.dev/headwaters/articles/8bc4e8c3119fa3
```

出力先を指定しない場合，PDFは `pdfs/` に保存されます。ファイル名は，Zennのユーザー名と記事スラッグから自動で決まります。

上の例では次のPDFが作成されます。

```text
pdfs/headwaters_8bc4e8c3119fa3.pdf
```

出力先を指定したい場合は，URLの後ろにPDFファイルのパスを入れます。

```bash
npm run pdf -- https://zenn.dev/headwaters/articles/8bc4e8c3119fa3 pdfs/gbrain.pdf
```

または，オプション形式でも指定できます。`--output` と `--out` は同じ意味です。

```bash
npm run pdf -- --url https://zenn.dev/headwaters/articles/8bc4e8c3119fa3 --output pdfs/gbrain.pdf
```

```bash
npm run pdf -- --url https://zenn.dev/headwaters/articles/8bc4e8c3119fa3 --out pdfs/gbrain.pdf
```

## 動作確認

構文チェックは次のコマンドで実行します。

```bash
npm run check
```

## 実行時の表示

実行すると，次のように進捗が表示されます。

```text
[1/8] Launching Chromium.
[2/8] Opening the Zenn article page.
[3/8] Waiting for the article body.
[4/8] Loading lazy content by scrolling.
[5/8] Waiting briefly for images and fonts.
[6/8] Extracting the article content.
[7/8] Building a printable standalone page.
[8/8] Writing the PDF file.
Done.
```

## 仕様

- PlaywrightでZenn記事を開き，ブラウザのPDF出力機能でA4 PDFを作成します。
- Zennページ全体をそのまま印刷せず，記事本文の `.znc` 要素を中心に抽出して，PDF用の単独HTMLを作ってからPDF化します。
- ヘッダー，フッター，サイドバー，プロフィールカード，コメント欄はPDFに含めません。
- タイトル，出典URL，タグ，記事本文，画像，コードブロック，表をPDFに含めます。
- 画像はページ末尾までスクロールして読み込みを促してからPDF化します。
- 非公開記事，ログインが必要な記事，有料記事の本文は対象外です。

## 表示確認用HTMLを保存する

PDFの体裁を確認したい場合は，PDF化直前のHTMLを保存できます。

```bash
npm run pdf -- https://zenn.dev/headwaters/articles/8bc4e8c3119fa3 --debug-html debug/gbrain.html
```

HTMLをブラウザで開くと，PDFに入る本文の幅，コードブロック，表，画像の状態を確認できます。

## よくあるエラー

### `Executable doesn't exist` と表示される

Chromiumが未導入です。次を実行してください。

```bash
npx playwright install chromium
```

### `The URL must point to a Zenn article` と表示される

ZennのユーザーURLやトップページではなく，記事URLを指定してください。

```text
https://zenn.dev/<user>/articles/<slug>
```

### PDFの本文が細い列になる

古い版では，Zennの画面レイアウトを残したまま `article` 要素を印刷していたため，本文が左側の細い列に押し込まれることがありました。この版では `.znc` の本文を抽出し，PDF用の単独HTMLに組み直してから出力します。

### 途中で止まる

最後に表示されたステップを確認してください。ネットワークが遅い場合は，次のように待機時間を伸ばせます。

```bash
npm run pdf -- https://zenn.dev/headwaters/articles/8bc4e8c3119fa3 --timeout 120000
```
