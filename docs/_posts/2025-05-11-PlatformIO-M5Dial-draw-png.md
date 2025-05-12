---
layout: diary
title: PlatformIOを使ってSPIFFSに画像を保存して、その画像をM5Dialに表示する
category: programming
---

M5Dialに画像を表示しようと思ったら少しハマったのでメモ。

まず、環境とかは以下のような感じ。
* VSCodeのPlatformIO拡張を使って開発
* ターゲットはM5Dial
* M5Unified/M5GFXで描画
* すでに画面サイズ分のM5Canvasが2つ作られている
* 画像は `SPIFFS` に保存する

最初にやるのは `SPIFFS` の有効化なので、 `platformio.ini` に以下を追加。

```
board_build.filesystem = spiffs
board_build.filesystem_size = 1M
```

これで、　`SPIFFS` が1MBほど利用可能になる。

M5GFXには `drawPngFile` という関数があるけど、M5Dialとの組み合わせの問題なのかコンパイルエラーが出た。
なので、以下のようなコードで回避。

```
File file = SPIFFS.open("/path/to/png", "r");
if (file) {
    display.drawPng(file, 0, 0, 100, 100);
}
```

これで実行すると、画像描画しようとして再起動を繰り返してる。
M5Canvasで割と大きくヒープを使ってるため、画像表示用のヒープが確保できずにそのままパニックになって再起動ということらしい。
なので、画面サイズ分のM5Canvasのうちの1つを取りやめて他の方法で直接画面に描画することで、PNG画像の描画も問題なくできた。

最後に画像ファイルの `SPIFFS` への転送は以下の手順。

1. プロジェクトのルート直下に `data` ディレクトリを追加して、そこに画像ファイルを保存。
2. PlatformIO CLIから以下のコマンドを実行。
   
   ```
   pio run -t uploadfs
   ```

これで `SPIFFS` にある画像が表示できるようになる。