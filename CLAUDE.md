# まいにち おりがみ — 引き継ぎメモ(Claude Code 用)

## このアプリについて
- 3歳の娘と親子で使う、1日1つの折り紙アプリ。作者は「かっ」(プログラマーではない。説明は専門用語を避けて短く)
- 端末:Fire HD 8 に入れた Chrome で、GitHub Pages から PWA として使う
- 公開URL:https://(ユーザー名).github.io/origami/ (このリポジトリがそのまま公開される)

## ファイル構成(すべてリポジトリ直下)
- `index.html` … アプリ本体(HTML・CSS・JS がこの1ファイルに全部入り)
- `sw.js` … オフライン用 Service Worker
- `manifest.webmanifest` / `icon-192.png` / `icon-512.png` / `icon-maskable-512.png`

## 更新するときの決まり(必ず守る)
1. `index.html` を直したら、`sw.js` の `const VERSION = "vN"` を1つ上げる
2. `index.html` の日付横の表示 `<small ...>vN</small>` も同じ番号にそろえる(かっが Fire で版を確認する目印)
3. コミット → push すれば GitHub Pages に反映。Fire ではアプリを2回開き直すと切り替わる
4. ファイル名は必ず `index.html` / `sw.js` のまま(過去に「index (1).html」で上書きされない事故あり)

## 現在の状態
- 最新は v4(音声読み上げを削除、カメラをアプリ内起動、更新がすぐ反映されるよう sw.js を no-store 化)
- 未解決:Fire で「音声が消えない」報告あり。v4 がまだ反映されていない可能性が高い。まずリポジトリの index.html が v4 か確認する
- 音声読み上げは不要(かっの希望)。`say()` は空関数のまま残してある

## UI の方針
- 子ども向け:大きなボタン、絵文字、文字は少なく、ひらがな中心
- 「🔒 おとな」ボタンを長押し(約1秒)で おとなモード。動画登録・記録削除・作品追加・バックアップ・メモは おとなモードの中だけ
- 画面:きょう(今日の1作品)/おる(プレイヤー)/きろく(写真・満足度・つぎは?)/スタンプ/ずかん

## データ(すべて端末内の IndexedDB、DB名 `maiorigami`)
- `records` … 記録 {day, date, modelId, photoId, face, self, next, memo, createdAt}
- `videos` … キー=作品ID {assetId(blobsのキー), link}
- `models` … おとなが追加した作品。キーは `u_` で始まる
- `blobs` … 写真・動画の実体
- バックアップ:おとなモード → スタンプ画面で JSON 書き出し/読み込み(動画は含めない)
- データ構造を変えるときは、既存の記録が消えないように移行を考えること

## コンテンツ(index.html 内「コンテンツ」ブロック)
- `MODELS` … 作品の一覧 {name, e(絵文字), c(色), lv(1〜3), steps(大人向け文章), tip}
- `COURSE` … 30日コース(作品IDの並び、"free" は自由の日)
- `ANIMS` … 動く説明。正方形の紙(240×240座標、ひし形 or `shape:"square"`)を平らに折るシミュレーション
  - `fold:[[x1,y1],[x2,y2]], side:[x,y]` … 折り線と、動かす側の点
  - `back:true` うしろに折る/`layers:[i,j]` 重なりの一部だけ折る/`multi:[...]` 同時に複数折る
  - `crease:true` 折って戻す/`flip:true` 裏返す
  - `draw:[...]` 顔などを描く/`replace:[...]` 立体的な工程を形の切り替えで表現
- 新しい作品を足すときは MODELS・ANIMS に追加し、必要なら COURSE に入れる。追加後は各ステップの見た目を確認する
