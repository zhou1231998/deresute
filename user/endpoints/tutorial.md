# tutorial エンドポイント

ユーザー登録時のチュートリアル処理を行うエンドポイント。

## tutorial/update_step

`ApiType` は `Tutorial`。`Stage.TutorialTask` に実装されている。

チュートリアルの進行状況を通知する。

### チュートリアルについて

このエンドポイントではサーバー側のチュートリアル処理の更新を行う。`Stage.TutorialDefine.eTutorialServerStep` を見るとよい。

順序は以下のとおりである。

1. （10）ストーリー（Cast a spell on me!）
1. （20）センターアイドル選択
1. （30）ストーリー（センターアイドル選択後）
1. （40）ローカルガシャ
1. （50）練習 LIVE
1. （55）練習 LIVE のリザルト
1. （60）特訓
1. （70）LIVE（お願い! シンデレラ Debut）
1. （73）LIVE 開始
1. （77）LIVE のリザルト
1. （80）ルーム
1. （90）プロデューサー名の設定

チュートリアルはガシガシスキップできるが、すべて通信は省略しないようである。

ちなみに同僚検索から見れるようになるのは 90 を送ったあとのようである。0 と 100 は送らなくてよい（送ってもエラーになる）。

### リクエスト

LIVE のリザルトは送っていない。

- `int step`：ステップ（10 から 100、`Stage.TutorialDefine.eTutorialServerStep` を確認のこと）
- `int type`：0（ただしセンターアイドル選択のときのみ 1〜3）
- `int skip`：スキップする場合は 1（スキップできるのは `Story_1`、`Story_2`、`End` のみ）
- `map room_info`：ルーム配置情報（ルーム設定のとき以外はそれぞれ空配列）
    - `array floor`
        - `int serial_id`
        - `int index`
        - `int reversal`
        - `int sort_num`
    - `array wall`
        - `int serial_id`
        - `int index`
        - `int reversal`
        - `int sort_num`
    - `array theme`
        - `int serial_id`
        - `int index`
        - `int reversal`
        - `int sort_num`
- `map storage_info`：倉庫情報（ルーム設定のとき以外はそれぞれ空配列）
    - `int[] diff_in`
    - `int[] diff_out`
- `string name`：空文字列（プロデューサー名設定のときのみプロデューサー名）

### レスポンス

- `int step`：次のステップ（クライアント未使用）

#### センターアイドル選択直後（20を送ったとき）

この時点で内部データとしてはプロデューサーLvは 2 である（お願い! シンデレラの直後のデータになっている）

- `array card_list`：load/index の `user_card_list` にほぼ同じ（`viewer_id` だけ抜けている）
    - `int serial_id`
    - `int card_id`
    - `int level`：クライアント未使用
    - `int exp`
    - `int step`
    - `int love`
    - `int skill_level`
    - `int protect`：クライアント未使用
- `array gacha_result`：gacha/exec の `gacha_result` よりはかなり情報が少ない
    - `int reward_id`
    - `int reward_type`：6（クライアント未使用）
    - `int is_new`：true（クライアント未使用）
- `array user_unit_list`：要素は1つだけ
    - `int serial_id_0`
    - `int serial_id_1`
    - `int serial_id_2`
    - `int serial_id_3`
    - `int serial_id_4`
    - `string name`
    - `int viewer_id`：クライアント未使用
- `map user_info`：load/index の `user_info` にほぼ同じ
    - `string name`：空文字列
    - `string comment`：よろしくお願いします
    - `int max_card_num`：100
    - `int max_room_storage_num`：150
    - `int friend_pt`
    - `int jewel`
    - `int free_jewel`
    - `int gold`
    - `int level`
    - `int exp`
    - `int fan`
    - `int stamina`
    - `long stamina_heal_time`
    - 他多数（クライアント未使用）

#### 練習 LIVE 開始直後（50を送ったとき）

- `int add_love`：獲得合計親愛度
- `array love_list`：親愛度一覧
    - `int serial_id`：アイドルID
    - `int before_love`：直前親愛度
    - `int after_love`：現在親愛度
- `array fan_list`：ファン数一覧
    - `int chara_id`：キャラクターID
    - `int before_fan`：直前ファン数
    - `int after_fan`：現在ファン数

#### お願い! シンデレラ 開始直後（73を送ったとき）

- `int add_love`：獲得合計親愛度
- `array love_list`：親愛度一覧
    - `int serial_id`：アイドルID
    - `int before_love`：直前親愛度
    - `int after_love`：現在親愛度
- `array fan_list`：ファン数一覧
    - `int chara_id`：キャラクターID
    - `int before_fan`：直前ファン数
    - `int after_fan`：現在ファン数
- `map user_update_info`：ユーザー更新情報
    - `int before_level`
    - `int after_level`
    - `int add_exp`
    - `int exp`
    - `int add_gold`
    - `int add_fan`
    - `int fan`
    - `int add_friend_pt`
    - `int friend_pt`
    - `int before_producer_rank`：クライアント未使用
    - `int after_producer_rank`：クライアント未使用
- `array drop_list`：ドロップリスト
    - `int id`
    - `int reward_type`
    - `int num`
    - `int card_level`
    - `int card_exp`
    - `int is_new`

クライアントではプロデューサーランクの変化は `['user_update_info']['producer_rank']` を見ているのだが、送られてくるデータは `before` と `after` の形式になっている。
