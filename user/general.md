# ユーザーデータの通信概要

## デレステでの利用

デレステでは `UnityEngine.WWW` を用いている（`Cute.NetworkManager` の `Connect` メソッド参照）。

API通信を担うのは `Cute.NetworkTask` というクラスで、各通信はこのクラスのほかこれを継承した `Stage.BaseTask` を継承して実装されている。

APIの種類は `ApiType` にリスト化されている。

なおメソッドはすべて POST である（これは `UnityEngine.WWW` の実装によるが、どうやら GET でも動きそうである）。

## 自らAPIを叩く

実際に問題になるのは、自分でAPIを叩けるかという問題である。

必要なパラメータは

- UDID
- User ID
- Viewer ID

の3つのみである。

UDID と User ID はリクエストヘッダに暗号化されて書かれているが、容易に復号化できる（後に説明する通称 `lolfuscate` である）。パケットをキャプチャしてしまえばよい。

Viewer ID はいわゆるユーザーIDなのですぐにわかる。

あとはリクエストボディを正しく生成すれば通信が行えるため、非常に容易に通信できるといっても過言ではない。

実際の実装も [API通信の実装](implementation.md) に示した。

## エンドポイント

http://game.starlight-stage.jp/

たとえば VersionCheck であれば http://game.starlight-stage.jp/load/check になる。

API の種類の一覧は `ApiType` から見ることができる。[エンドポイント一覧](endpoints.md) も参照されたい。

## HTTPヘッダ

### リクエストヘッダ

`UDID` 以下はデレステ側で設定しているもの。`X-Unity-Version` 以上は（おそらく）自動付与。

| ヘッダ名             | 説明               | 例                                              |
|----------------------|--------------------|-------------------------------------------------|
| Host                 |                    | game.starlight-stage.jp                         |
| User-Agent           |                    | BNEI0242/2.7.2 CFNetwork/758.3.15 Darwin/15.4.0 |
| Content-Type         |                    | application/x-www-form-urlencoded               |
| Content-Length       |                    | 384                                             |
| Connection           |                    | keep-alive                                      |
| Connection           | 2回目              | keep-alive                                      |
| Accept               |                    | \*/\*                                           |
| Accept-Encoding      |                    | gzip, deflate                                   |
| Accept-Language      |                    | en-us                                           |
| X-Unity-Version      |                    | 5.1.2f1                                         |
| UDID                 | 後述               |                                                 |
| USER_ID              | 後述               |                                                 |
| SID                  | 後述               |                                                 |
| PARAM                | 後述               |                                                 |
| DEVICE               | 引き継ぎ回数       | 1                                               |
| APP_VER              | アプリバージョン   | 2.7.2                                           |
| RES_VER              | リソースバージョン | 10023800                                        |
| DEVICE_ID            | デバイス固有GUID   | D6A3C479-DC23-49C8-8B92-8503894EFB10            |
| DEVICE_NAME          | デバイス名         | iPod7,1                                         |
| GRAPHICS_DEVICE_NAME | GPU名              | Apple A8 GPU                                    |
| IP_ADDRESS           | ローカルIPアドレス | 192.168.0.1                                     |
| PLATFORM_OS_VERSION  | OSバージョン       | iPhone OS 9.3                                   |
| CARRIER              | 通信キャリア       | docomo                                          |
| KEYCHAIN             | キーチェーンID     | 123456789                                       |

詳しく調査していないが、まともな値であるべきフィールドは

- Host
- Content-Length
- UDID
- USER_ID
- SID
- PARAM
- RES_VER

程度であると思われる。

### レスポンスヘッダ

| ヘッダ名       | 例                            |
|----------------|-------------------------------|
| Date           | Fri, 25 Mar 2016 16:16:21 GMT |
| Server         | Apache                        |
| Content-Length | 300                           |
| Connection     | close                         |
| Content-Type   | application/x-msgpack         |

## Certification

デレステでは認証に4つのパラメータが用いられている。

通信時にはこの4つのパラメータを様々な方法でやり取りしている。

### UDID（Unique Device Identifier）

デバイスの識別子か。

`[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}` の形式。GUID を小文字にしたものと同じ。

`219b8ff9-3d02-4df1-b7fc-dc5bbd560003` のような形である。

クライアント側でユーザー登録の際に生成している。

通信時には Cryptographer で暗号化されヘッダの `UDID` フィールドに書かれる。

### Viewer ID

同僚欄などで見ることのできる、いわゆる一般ユーザーからすれば「ユーザーID」。

int型だが、9桁の数字になる。

通信時には暗号化されリクエストボディに含まれる。レスポンスにも平文で書かれる。

### User ID

ユーザー固有のID。int型。

Viewer ID および User ID はユーザー登録時にサーバーから与えられる。

通信時には Cryptographer で暗号化されヘッダの `USER_ID` フィールドに書かれる。レスポンスにも平文で書かれる。

### Session ID

セッションID。

`[0-9a-f]{32}` で与えられる。MD5と同じ形式。

最初の通信は `ViewerId + Udid + "r!I@nt8e5i="` の MD5 で与えている。

その後の通信は直前のレスポンスにある `sid` にまた `r!I@nt8e5i=` を付与したものの MD5 で与える。

## Cryptographer

`UDID` および `USER_ID` の暗号化メソッドについて解説する。

なおこの手法は [marcan/deresuteme](https://github.com/marcan/deresuteme) では `lolfuscate` と呼ばれている。

エンコードでは文字列から以下のようにして文字列を生成する。

- 文字列の長さ（16進数で4桁）
- 各文字に対して
    - 2桁の乱数
    - 文字コード+10 にあたる文字
    - 1桁の乱数
- 32桁の乱数

デコードでは逆に操作すればよい。

実装は `Cute.Cryptographer` 内の `encode` および `decode` にある。

## リクエストボディの生成（CryptAES）

リクエストボディは `Cute.NetworkTask` 内の `CreateBody` に生成方法がある。

順序としては

1. MessagePack によりバイナリ列に変換
1. Base64 でエンコード
1. Rijndael で暗号化
1. 最後に32文字の鍵を結合
1. Base64 でエンコード

となる。

### MessagePack について

[MessagePack](http://msgpack.org/) は JSON に似たオブジェクトのシリアライズフォーマットである。

クライアントでは [masharada/msgpack-unity](https://github.com/masharada/msgpack-unity) を用いているようである。

サーバー側の MessagePack は古いバージョンを用いているため、str 8 フォーマットを受け付けない。一部実装では compatibility mode を使用する必要がある。

型は曖昧に実装されているようである（たとえば int 32 であるところを uint 32 で送っても問題ない）。

詳細は [仕様](https://github.com/msgpack/msgpack/blob/master/spec.md) を確認するとよい。

### 暗号化

実際には `Cute.CryptAES` の `EncryptRJ256(string prm_text_to_encrypt)` というメソッドで暗号化を行っている。

なお鍵を結合して2度目の Base64 によるエンコードもこのメソッド内で行っている。

実装には .NET の `System.Security.Cryptography.RijndaelManaged` を用いているが、他実装でも可能である。

条件は以下のとおりである。

- ブロック暗号手法：Rijndael
- 暗号利用モード：CBC（Cipher Block Chaining）
- ブロックサイズ：256 bit
- 鍵サイズ：256 bit
- パディング：ゼロ埋め
- 鍵：適当な32文字の文字列
- 初期化ベクトル：UDID の `-` を取り除いた32文字

鍵は `Base64::encode64((0...32).map{'%x' % rand(65536)}.join)[0...32]` のような実装で作っている。

### 復号化

レスポンスの復号化は同様に行えばよい。これは `Cute.NetworkManager` で行われている。

1. Base64 でデコード
1. 最後の32文字の鍵を取る
1. Rijndael で復号化（条件は暗号化時と同一）
1. Base64 でデコード
1. MessagePack を `Dictionary` などに展開

なお UDID が不明の通信では `"\0"*32` が初期化ベクトルとなる（正規の通信ではこの現象は起こりえない）。

## PARAMの生成

`Cute.NetworkTask` 内の `AddHeaderParam` に実装されている。

以下の4つを順に結合したものの SHA1 になっている。

- UDID
- Viewer ID
- エンドポイントの絶対パス（`/load/check`など）
- MessagePack を Base64 でエンコードしたもの

## リクエストボディの中身

MessagePack によりシリアライズされるリクエストボディは以下のようになっている。ここでは MessagePack を JSON に変換して表記している。

```json
{
    "app_type": 0,
    "campaign_data": "",
    "campaign_sign": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "campaign_user": 123456,
    "timezone": "09:00:00",
    "viewer_id": "43574052975323675801839343555790kTzQvDagcj+2rqehDtsUNn6Wz6MpZoVI7+cjDldkOhU="
}
```

このうち `timezone` と `viewer_id` は必ず存在する（`Cute.NetworkTask` 内の `AddHeaderParam` に実装されている）。

`timezone` は `TimeZone.CurrentTimeZone.GetUtcOffset(DateTime.Now).ToString()` なだけだが、基本的には `09:00:00` でよいだろう。ちなみに以前はなかった。

### Viewer ID の暗号化

Viewer ID は Rijndael で暗号化されている。最初の32文字は初期化ベクタ、後ろが暗号化済みの Base64 になる。

暗号化は `Cute.AES256Crypt` で行っているが、実際には `Cute.CryptAES` とさほど変わらない。

暗号化の条件は以下のとおりである。

- ブロック暗号手法：Rijndael
- 暗号利用モード：CBC（Cipher Block Chaining）
- ブロックサイズ：256 bit
- 鍵サイズ：256 bit
- パディング：ゼロ埋め
- 鍵：`s%5VNQ(H$&Bqb6#3+78h29!Ft4wSg)ex`
- 初期化ベクトル：`[0-9]{32}`

## レスポンスボディの中身

リクエストボディと同様に MessagePack によりシリアライズされたものを JSON に変換して示す。

```json
{
    "data_headers": {
        "sid": "da39a3ee5e6b4b0d3255bfef95601890afd80709a3",
        "user_id": 123456789,
        "viewer_id": 123456789,
        "servertime": 1487834920,
        "result_code": 1
    },
    "data": []
}
```

`data_headers` と `data` からなり、ともに Map の形になる（上記の例では `data` が空のため Array になっている）。

`data_headers` は中身が（ほぼ）固定、`data` は API によって異なる。

### data_headers

`sid` はセッションIDにあたる。次のリクエスト時のヘッダの `SID` に用いられる。生成方法は前述の通り、`sid + "r!I@nt8e5i="` の MD5 である。

`user_id` および `viewer_id` は自分自身のものが返ってくる。リクエストが不正な場合は `false` のことがある。クライアント側ではユーザ登録時（`Cute.LoginTask` で用いられている `SignUp`）以外は使用していないようである。

`servertime` はサーバーの UNIX 時間。無意味。

`result_code` はステータスコードにあたる。`Stage.SkipCuteServerCheckResultCode` にある程度一覧化されている。1 以外はエラーである（ただしエラーの場合でも 1 のことがある）。詳しくは [リザルトコード一覧](result.md) を参照のこと。

リザルトコード 204 の場合は `store_url` が現れ、ここにあるリンクに飛ばされる。

リザルトコード 214 の場合は `required_res_ver` が現れ、このリソースバージョンへの更新を促される。

その他は `Cute.Header` に定義されている構造を参考にされたい。
