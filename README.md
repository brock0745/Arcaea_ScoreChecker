# Arcaea 理論値チェッカー

非公式の Arcaea 理論値・99.5%・PM・EX+ 達成状況トラッカーです。GitHub Pages などの静的ホスティングにそのまま配置できます。

## 構成

```
arcaea-tracker/
├── index.html       アプリ本体（ロジック・UI）。基本的に更新頻度は低い。
├── songs.json        全譜面データ（曲名・難易度・理論値難易度・ノーツ数・定数・ジャケット）。曲追加時はここだけ編集すればよい。
├── jackets/          ジャケット画像フォルダ
├── update.py         songs.json 更新用スクリプト（xlsx取込 / 手動追加 / tier更新）
├── manifest.json     PWA設定
├── sw.js             オフライン対応用 Service Worker
├── icon-192.png / icon-512.png   PWAアイコン
└── README.md         このファイル
```

## デプロイ方法（GitHub Pages）

1. このフォルダの中身をリポジトリのルート（またはお好きなサブフォルダ）にそのまま置いて push
2. リポジトリの Settings → Pages で公開ブランチを設定
3. `https://ユーザー名.github.io/リポジトリ名/` でアクセス可能になる

`index.html` は起動時に同じ階層の `songs.json` を `fetch()` で読み込みます。ファイルを丸ごと同じ場所に置いておけば動作します。

## 曲を追加する3つの方法

### 方法A: songs.json を直接編集（最も手軽）

`songs.json` はただのテキストです。GitHub 上で直接開いて「Edit」ボタンで編集し、配列の末尾に以下の形式でオブジェクトを1つ追加して commit するだけです。

```json
{"title":"英題","disp":"和題（なければ英題と同じでOK）","diff":"FTR","tier":0,"notes":1234,"cc":11.0,"jacket":null}
```

- `tier` が分からない場合は `0` にしておくと「未確定」欄に表示されます。後で分かり次第書き換えてください。
- `jacket` はジャケット画像を `jackets/` に置いた場合そのファイル名を指定します（`null` のままでもグラデーション表示で動作します）。

### 方法B: コンサルタントシート(xlsx)から一括更新

```bash
pip install openpyxl --break-system-packages
python3 update.py --xlsx "新しいコンサルタントシート.xlsx"
```

シートの `DataM` タブから曲名・ノーツ数・定数を読み取り、`songs.json` に自動マージします（既存曲は更新、新曲は tier=0 で追加）。

### 方法C: コマンドで1曲だけ追加

```bash
python3 update.py --add --title "曲名" --diff FTR --tier 25 --notes 1234 --cc 11.0
```

理論値難易度（tier）だけ後から更新する場合:

```bash
python3 update.py --set-tier --title "曲名" --diff FTR --tier 25
```

## CSV機能

フッターの「入力用CSVを入手」で全譜面（曲名・難易度・レベル・スコア）のCSVをダウンロードできます。スコア入力後にそのファイルを「CSVから取込」で読み込むと、一括でスコアを反映できます。既存のスコアがある場合はCSVにも反映された状態で出力されるので、Excel等で編集してそのまま再取込することも可能です。

CSV形式（1行目はヘッダー、UTF-8 BOM付き）:

```
曲名,難易度,レベル,スコア
Testify,BYD,12.0,10002221
Testify,FTR,10.9,
```

## HTML版とアプリ版の違い

- **HTML版（ブラウザ / GitHub Pages）**: 最初に注意事項ページが表示され、スクロールするとトラッカー本体に切り替わります。
- **アプリ版（APK / WebView）**: 注意事項ページは表示されず、代わりにツールバーの「注意事項」ボタンからモーダルで確認できます。

判定は URL のホスト名（`appassets.androidplatform.net` かどうか）で自動的に行われます。

## データの保存場所について

達成マーク・スコア・アップロードした画像・追加した譜面は、すべて閲覧者の端末のブラウザ内（IndexedDB）に保存されます。`songs.json` やこのリポジトリには一切含まれないため、公開しても個人の記録が他人に見えることはありません。
