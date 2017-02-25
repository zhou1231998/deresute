# tool エンドポイント

非常に危険なツール。

## tool/signup

`ApiType` は `SignUp`。`Cute.LoginTask` に実装されている。

新しいユーザーを登録する。

Viewer ID および User ID は 0 で初期化する。UDID はランダムな GUID を生成すること。

なおこの時点で登録された Viewer ID は同僚検索などでは検索できない。

ちなみに登録時のプロデューサー名は空文字列である。

### リクエスト

- `string device_name`：デバイス名。ヘッダの `DEVICE_NAME` と同じ。
- `string client_type`：引き継ぎ回数。ヘッダの `DEVICE` と同じ。
- `string os_version`：OS のバージョン。ヘッダの `PLATFORM_OS_VERSION` と同じ。
- `string app_version`：アプリバージョン。ヘッダの `APP_VER` と同じ。
- `string resource_version`：`os_version` と同じ。リソースバージョンではない。

### レスポンス

空配列。

`data_headers` に含まれる `user_id` および `viewer_id` をクライアント側で保存して利用する。

なお `data_headers` には通常に加えて `udid` が含まれる（これはこちら側で送信した UDID と同じである）。
