# migration エンドポイント

load/index、load/check の次に行われる通信。ログイン時に必ず行われるものの、なくてもログインできる。

新機能のチュートリアル、ウワサ・1コマ劇場のアップデート情報をやり取りする。

## migration/index

`ApiType` は `MigrationUpdate`。`Stage.MigrationUpdateTask` に実装されている。

前回の通信で追加されたかどうかによって通信内容が異なる。

### リクエスト

- `int get_tips`：前回の通信でウワサ・1コマ劇場が追加されていない場合 1、そうでない場合 0
- `int[] tips_id_list`：ウワサ・1コマ劇場のリスト
- `int get_individual_tutorial`：前回の通信で終了済みのチュートリアルが追加されていない場合 1、そうでない場合 0
- `int[] individual_tutorial_list`：終了済みのチュートリアルのリスト（`get_individual_tutorial` が 0 のときのみ存在）

### レスポンス

必要がなければフィールドが存在しないため、空配列の場合がある。

- `int[] tips_id_list`：追加するウワサ・1コマ劇場のID
- `int[] individual_tutorial_list`：追加する終了済みのチュートリアルのリスト

### individual_tutorial_list のフィールドについて

新機能・データ追加のチュートリアルに関して、終了しているものの一覧になる。

全部で55のチュートリアルの各真偽値を示すために、ここでは55個の真偽値の配列でなく4つの数字の配列で示している。

1つめの数値が最初の16個（15まで）、2つめの数値が次の16個（31まで）、3つめが47まで、4つめが63までを示す。

各数値は2進数で考えて各桁小さい方から順にその真偽値を表している。

たとえば `[true, true, false, true, false] = 0b01011` の要領である。
