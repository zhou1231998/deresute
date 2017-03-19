# マニフェストとリソースの取得方法

デレステで使われているリソースの取得方法を示す。

基本的には http://storage.game.starlight-stage.jp/ 以下の URL を触ることになる。

どうでもいいが https://storages.game.starlight-stage.jp/ でも同じデータに触れるみたい（こちらは HTTPS 用になっている）。

## リソースバージョン

デレステにはリソースにバージョンがついている。たとえば 10024100 などである。

「新しいダウンロードデータがあります」とポップアップされるときは、このリソースバージョンが上がったタイミングである。

リソースバージョンはおおよそ 10 の倍数で、メジャーバージョンが 100 の倍数になる。そのため 10023900 → 10023910 → 10024000 などとバージョンが上がることがある。

過去にはマイナーバージョンが 60 や 70 だったこともあるため、当てずっぽうでリソースバージョンを決める場合でも 10 ずつ上げることには注意したい。

リソースバージョンに関しては併せて [ユーザーデータの通信概要](../user/general.md) を読むとよい。

## マニフェストのマニフェスト

マニフェストの一覧は http://storage.game.starlight-stage.jp/dl/バージョン番号/manifests/all_dbmanifest にある。

中身は CSV で、以下のような内容である（ヘッダ行はない）。

| ファイル名          | MD5                              | OS      | 音質 | 画質 |
|---------------------|----------------------------------|---------|------|------|
| iOS_AHigh_SHigh     | bb6ed46abe758259e1fae80bec66e351 | iOS     | High | High |
| Android_AHigh_SHigh | 9702fccf31ccd1a8839100e3f36e700a | Android | High | High |
| iOS_ALow_SHigh      | 0d910735f6fd5f498a3bfcdb4f4bd15c | iOS     | Low  | High |
| Android_ALow_SHigh  | 235e1809e7325d6ac1422e0fb3904485 | Android | Low  | High |
| iOS_AHigh_SLow      | 7ec7f8cb6130ff4fb6179e0d223fbc5d | iOS     | High | Low  |
| Android_AHigh_SLow  | 38db28d1ffff15eea0fe30280b1d065a | Android | High | Low  |
| iOS_ALow_SLow       | c186d27985ca70a8767653471d06428d | iOS     | Low  | Low  |
| Android_ALow_SLow   | 720ff17b3aa2df32e6c627415562848d | Android | Low  | Low  |

## マニフェスト

マニフェストの一覧の中から適切なマニフェストを選び、それをダウンロードする。解析では Android_AHigh_SHigh が使われることが多い。

URL は http://storage.game.starlight-stage.jp/dl/バージョン番号/manifests/Android_AHigh_SHigh である。

ファイルの中身は LZ4 で圧縮済みの SQLite になっている。またファイルの MD5 はマニフェストの一覧に載っているものと一致する。

展開した SQLite には、manifests テーブルのみが含まれる。

manifests テーブルは以下のような内容になっている。全部でエントリは 25000 ほど。

| name                              | hash                             | attr | category | decrypt_key |
|-----------------------------------|----------------------------------|------|----------|-------------|
| 3d_an_chr_view0001_legacy.unity3d | 83d27a3f8aee5e8ae9970bc66ec5e2d6 | 1    | every    | NULL        |
| 3d_chara_body_0001.unity3d        | 0330325d794cb6e47a288aced3516914 | 1    | every    | NULL        |
| 3d_chara_body_0002.unity3d        | 20633bb8015867cb54abd8e17def1b54 | 1    | every    | NULL        |
| 3d_chara_body_1001.unity3d        | a31f553aa68c99020242348a120679fe | 1    | every    | NULL        |

各列は以下の通り。

- name：ファイル名
- hash：ファイルの MD5（解凍前）
- attr：LZ4 で圧縮しているか（1 なら圧縮）
- category：カテゴリ
- decrypt_key：未使用

## リソースの種類と URL

拡張子によって異なる。

- unity3d：Unity アセットバンドル
- acb / awb：サウンドファイル
- mp4 / ogg：ムービーファイル（未実装）
- bdb：バイナリ DB
- mdb：マスター DB

bdb（Blob DB）と mdb（Master DB）はともに SQLite の形式である。

各リソースの URL はディレクトリで、ファイル名はマニフェストに載っているハッシュ値になる。

またファイル名にディレクトリが入る場合は、URL にこのディレクトリが入る。たとえば v/card_100174.acb であれば、URL は http://storage.game.starlight-stage.jp/dl/resources/High/Sound/Common/v/18f3a52b79f19ce6887110f8642f72f3 となる。

なおマニフェストに載っているものの 404 Not Found が返ってくるデータが一部存在する。

### アセットバンドル

URL は http://storage.game.starlight-stage.jp/dl/resources/High/AssetBundles/Android/ 。

### サウンドファイル

URL は http://storage.game.starlight-stage.jp/dl/resources/High/Sound/Common/ 。

### ムービーファイル

実装されればおそらく URL は http://storage.game.starlight-stage.jp/dl/resources/High/Movie/Common/ 。

### バイナリ DB、マスター DB

URL は http://storage.game.starlight-stage.jp/dl/resources/Generic/ 。
