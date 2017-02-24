# エンドポイント別通信概要一覧

通信で用いられる主要なエンドポイントについて解説する。

## 表記について

通信内容の説明は箇条書きで行う。たとえば以下のような MessagePack であれば、

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

以下のように箇条書きにする。

- `int app_type`
- `string campaign_data`
- `string campaign_sign`
- `int campaign_user`

`timezone` と `viewer_id` は常に必要なため毎回説明することはしない。これに関しては [概要](../general.md) を参照のこと。

また内部に Map が含まれれば `map`、Map の配列が含まれれば `array`、その他の配列になっていれば `int[]` などと表記している。`array` はすなわち `map[]` ということである。

## 解析について

通信内容は [概要](../general.md) でも言及したとおり、`Cute.NetworkTask` もしくは `Stage.BaseTask` を継承した各クラスで行われている。

クラスの `SetParameter` メソッドで送信内容の生成を、`Parse` メソッドで受信内容の解析を行っている（`SetParameter` はオーバーライドされたメソッドではなく個々で実装している）。

パケットのキャプチャと各クラスのソースコードをにらめっこすることで、より深い理解が得られるだろう。

## ミッションクリアのフィールド

様々な手法でミッションがクリアできるため、この情報はルートにある場合がある。

JSON のデータを `Cute.NetworkUtil` の `SetMissionClearedData` に渡しているので、実装は `NetworkUtil` 側である。

- `array cleared_missions`
    - `array reward_list`：得られる報酬のリスト
        - `int reward_type`：種類（1：マニー、2：友情pt、3・4：ジュエル、5：アイテム、6：アイドル、7：ルームアイテム、11：称号）
        - `int reward_id`：`reward_type` が 5・6・7 の場合にそのIDを示す
        - `int reward_num`：獲得数
    - `int present_count`：プレゼントの個数?
    - `int mission_type`：ミッションの種類（1：デイリー、2：ウィークリー、3：ノーマル、4：限定）
    - `int mission_id`：ミッションのID
    - `? is_new`
    - `int group_id`：ノーマルミッションのグループID
    - `int step`：ノーマルミッションのステップ数
    - `array live_data_list`：解放したライブのリスト?
        - `int live_detail_id`：Live Detail ID
    - `array story_data_list`：解放したストーリーのリスト?
        - `int next_value`：ストーリーのID
    - `int lottery_count`：宝くじチケットの枚数

なおこの情報がある可能性のあるエンドポイントは `Stage.BaseTask` を参照のこと。

## ユーザーごとのポテンシャル解放情報

他ユーザーのポテンシャル解放状況も見られるため、この情報も複数のエンドポイントに存在する。こちらはルートではない場合がある。

- `array user_chara_potential`
    - `map chara_?`：?にはキャラクターIDが入る
        - `int param_1`：ボーカルのポテンシャル
        - `int param_2`：ダンスのポテンシャル
        - `int param_3`：ビジュアルのポテンシャル
        - `int param_4`：ライフのポテンシャル

## コンテンツ

- [load](load.md)
    - [load/check](load.md#loadcheck)
    - [load/index](load.md#loadindex)
- [migration](migration.md)
    - [migration/index](migration.md#migrationindex)
