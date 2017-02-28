# リソースフォーマット

デレステで使われているリソースのフォーマットについて解説する。

## LZ4

デレステでは unity3d ファイルおよび SQLite ファイルは LZ4 で圧縮されている。

バイナリのフォーマットは以下のようになっている。数値はリトルエンディアンである。

```
0x0000 | 64 00 00 00 | 固定?
0x0004 | 00 40 14 00 | 解凍後ファイルサイズ（Byte）
0x0008 | CD 67 08 00 | 圧縮済みファイルサイズ（Byte）
0x000C | 01 00 00 00 | 固定?
0x0010 | F6 12 53... | 実際のlz4のデータ
```

最初の 16 バイトがヘッダになっているので、これを削って liblz4 の `LZ4_decompress_safe` を使用することで解凍できる。[LZ4 ブロック圧縮 API の使い方 - Qiita](http://qiita.com/dearblue/items/65e8526f47dc10a63f04) のページがわかりやすい。

なお Ruby では [extlz4](https://rubygems.org/gems/extlz4/versions/0.2.1) という gem の `LZ4::raw_decode` を用いることで解凍できる。

## SQLite

一般的なファイルフォーマットなので省略。

参考：[SQLite - Wikipedia](https://ja.wikipedia.org/wiki/SQLite)

## unity3d

unity3d ファイルは Unity のアセットバンドルのフォーマットである。[HearthSim/UnityPack](https://github.com/HearthSim/UnityPack) などを使って展開できるが、依然としてドキュメントが存在しないため、今回あらたに執筆した。

詳細は [unity3d ファイルのバイナリフォーマット](unity3d.md) を参照のこと。

その他：

- [unity3d の文字列テーブル](unity3d-string.md)
- [unity3d の Texture2D](unity3d-texture2d.md)

## ACB、AWB、HCA

[CRI Middleware](http://www.cri-mw.co.jp/index.html) の制作したフォーマット。

ドキュメントは未作成。
