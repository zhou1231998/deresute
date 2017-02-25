# load エンドポイント

登録済みユーザーが最初に行うログイン処理を行うエンドポイント。

## load/check

`ApiType` は `VersionCheck`。`Cute.VersionCheckTask` に実装されている。

必要性は微妙だが、セッションの確立用と考えるべきか。

### リクエスト

- `int app_type`：通常は 0。
- `string campaign_data`：通常は空文字列。
- `string campaign_sign`：何かの MD5。
- `int campaign_user`：0 以上 200000 以下の乱数。root 化している場合は奇数、そうでなければ偶数。

### レスポンス

空配列。

## load/index

`ApiType` は `Load`。`Stage.LoadTask` に実装されている。

ログイン処理を行う。

### リクエスト

リクエストは以下のような内容だが、サーバー側では利用していないようで、おそらく何を送っても受理される。

基本的には `Stage.LocalData` の中で重要そうなものをピックアップしているようである。

- `int tutorial_flag`：チュートリアルの状況。`Stage.TutorialDefine.eTutorialStep` 参照。終了時は 1000。
- `int live_detail_id`：進行中の LIVE のID。通常は 0。
- `int live_setting`：LIVE のクオリティ設定。`Stage.LocalData.OptionData.eQualityType` 参照。通常は 0 以上 3 以下。
- `int live_state`：進行中の LIVE のステータス。`Stage.LocalData.LivePlayData.eStep` 参照。通常は 0。
- `long friend_view_time`：前回同僚一覧を見た UNIX 時刻。
- `long name_card_view_time`：未実装。
- `int load_state`：チュートリアルを終えていて日付が変わる時は 1。それ以外は 0。

### レスポンス

必要な情報をすべてロードするため、ものすごい量になっている。バージョンアップによって変化しやすいレスポンスでもあるので、詳細はクライアントの実装を確認するとよい。

以下の順序はクライアントでの実装順になっている。

- `map common_define`：数値定義、機能開放日など
    - `int live_continue_jewel`：コンティニューに必要なジュエル
    - `int expanding_count`：1 回の所持枠拡張で増える枠数（5）
    - `int expanding_jewel`：1 回の所持枠拡張に必要なジュエル（50）
    - `int expanding_max`：最大所持枠
    - `int stamina_recovery_jewel`：スタミナ全回復に必要なジュエル（50）
    - `int stamina_recovery_time`：スタミナ 1 増えるために必要な秒数（300）
    - `int room_lvup_shortening_time`：ルームの家具レベルアップで 1 単位あたり短縮できる秒数（720）
    - `int room_lvup_jewel`：ルームの家具レベルアップで 1 単位あたり必要なジュエル数（1）
    - `int over_limit_friend_pt`：所持できる最大友情 pt（100000）
    - `long card_storage_start_time`
    - `int rank_sss_rank_condition`：SSS ランキング参加に必要なプロデューサーランク（8）
    - `int rank_sss_prp_condition`：SSS ランキング参加に必要な PRP（1000）
    - `int rank_sss_start_date`
    - `int live_sort_add_start_time`
    - `int master_plus_start_date`
    - `int live_reload_start_time`
    - `int live_reload_limit_count`
    - `int live_reload_cool_time`
    - `int room_expanding_count`：1 回の倉庫枠拡張で増える枠数（25）
    - `int room_expanding_jewel`：1 回の倉庫枠拡張に必要なジュエル（25）
    - `int room_expanding_max`：最大倉庫枠
    - `int exchange_money_start_date`
    - `int exchange_item_start_date`
    - `int name_card_v250_start_time`
    - `int add_unit_num_start_time`
    - `int dress_shop_start_time`
    - `int pinya_request_start_date`
    - `int pinya_request_start_level`：ぴにゃセレクトに必要なプロデューサーLv（30）
    - `int room_multi_anime_enabled`
    - 以下クライアント未使用
    - `int mission_enable`
    - `int mission2_enable`
    - `int story2_enable`
    - `int idol_skill_rate`
    - `int gacha_effect`
    - `int anniversary_page_enable`
    - `int produce_add_enable`
    - `int produce_anime_enable_date`
    - `int mission_sort_enable_date`
    - `int anv_login`
    - `int mission_instant_reward_start_time`
    - `int produce_add_voice_start_time`
    - `int sound_management_start_time`
    - `int guest_setting_start_time`
    - `int idol_protect_start_time`
- `map user_info`：ユーザー情報
    - `int tutorial_flag`：サーバー側のチュートリアルステップ（`Stage.TutorialDefine.eTutorialServerStep`、100が完了）
    - `int viewer_id`：ViewerID
    - `string name`：名前
    - `string comment`：コメント
    - `int max_card_num`：アイドル所持枠
    - `int max_room_storage_num`：倉庫枠
    - `int friend_pt`：所持友情pt
    - `int jewel`：所持有償ジュエル
    - `int free_jewel`：所持無償ジュエル
    - `int gold`：所持マニー
    - `int stamina`：現在のスタミナ
    - `int level`：プロデューサーレベル
    - `int exp`：累計経験値
    - `int fan`：累計ファン数
    - `int producer_rank`：プロデューサーランク
    - `int birth`：?
    - `int sum_of_money`：?
    - `int last_payment_date`：?
    - `long stamina_heal_time`：スタミナが全回復する UNIX 時刻
    - `int emblem_id`：設定している称号
    - `int leader_serial_id`：リーダーアイドルID
    - `int support_serial_id_1`：キュートサポートアイドルID
    - `int support_serial_id_2`：クールサポートアイドルID
    - `int support_serial_id_3`：パッションサポートアイドルID
    - `int support_serial_id_4`：全属性サポートアイドルID
    - `int unit_slot`：選択ユニットID
    - 以下クライアント未使用
    - `int device_type`
    - `int transition_flag`
    - `int max_friend`
    - `string last_login_time`
    - `int rare_gacha_count`
    - `int max_stamina`
    - `int fan_type_1`
    - `int fan_type_2`
    - `int fan_type_3`
    - `int prp`
    - `int leader_chara_id`
    - `int support_chara_id_1`
    - `int support_chara_id_2`
    - `int support_chara_id_3`
    - `int support_chara_id_4`
    - `int favorite_serial_id`
    - `string create_time`
    - `string update_time`
    - `string delete_time`
- `array user_card_list`：所持アイドルリスト
    - `int viewer_id`：所持主の Viewer ID（クライアント未使用）
    - `int serial_id`：アイドルID（所持アイドルに対するシリアルID）
    - `int card_id`：カードID（カードのマスターデータに対するID）
    - `int level`：レベル（クライアント未使用）
    - `int exp`：累計経験値
    - `int step`：スターランク?
    - `int love`：親愛度
    - `int skill_level`：特技 Lv
    - `int protect`
        - 1 の位：奇数なら鍵付き
        - 10 以上の位：女子寮ID?（0 なら所持枠）
- `array user_chara_list`：ファン数一覧
    - `int chara_id`：キャラクターID
    - `int fan`：ファン数
- `array user_chara_potential`：ポテンシャル一覧
    - `int chara_id`：キャラクターID
    - `int param_1`：ボーカルのポテンシャル
    - `int param_2`：ダンスのポテンシャル
    - `int param_3`：ビジュアルのポテンシャル
    - `int param_4`：ライフのポテンシャル
- `int user_guerrilla_group_id`：トレチケタイムに用いられるグループ番号
- `map schedule_list`：スケジュールのリスト
    - `array guerrilla_daily_schedule_list`：トレチケタイムのリスト
        - `int guerrilla_type`：1
        - `int start_time`：開始 UNIX 時刻
        - `int end_time`：終了 UNIX 時刻
- `array user_unit_list`：ユニットリスト
    - `int viewer_id`：所持主の Viewer ID（クライアント未使用）
    - `int unit_id`：ユニットID（1 から）
    - `string name`：ユニット名
    - `int serial_id_0`
    - `int serial_id_1`
    - `int serial_id_2`
    - `int serial_id_3`
    - `int serial_id_4`
    - `int dress_type_0`
    - `int dress_type_1`
    - `int dress_type_2`
    - `int dress_type_3`
    - `int dress_type_4`
- `array room_unit_info`：お気に入りユニットリスト
    - `int room_id`：ユニットID（1から）
    - `string name`：ルーム名
    - `int serial_id_0`
    - `int serial_id_1`
    - `int serial_id_2`
    - `int serial_id_3`
    - `int serial_id_4`
    - `int change_flag_0`：1 なら特訓前（センター）
    - `int change_flag_1`
    - `int change_flag_2`
    - `int change_flag_3`
    - `int change_flag_4`
- `int max_room_num`：ルーム数
- `int home_room_id`：メインのルームID
- `array item_list`：所持アイテムリスト
    - `int item_id`：アイテムID
    - `int number`：所持数
    - `long use_limit`：使用期限（UNIX 時刻、なければ 0）
- `array user_live_list`：LIVE の達成状況リスト
    - `int viewer_id`：所持主の Viewer ID（クライアント未使用）
    - `int live_detail_id`：Live Detail ID
    - `int max_score`：ハイスコア
    - `int max_combo`：最大コンボ数
    - `int clear_number`：クリア回数
    - `int play_number`：プレイ回数
    - `int score_rank`：ハイスコアランク
    - `int combo_rank`：コンボ数ランク
    - `int clear_rank`：クリア回数ランク
    - `string create_time`：最初にプレイした時刻
    - `string update_time`：最後にプレイした時刻
- `map music_list`：LIVE のリスト（未開放含む・それぞれ Live ID）
    - `int[] normal`：通常楽曲リスト
    - `int[] sp`：限定楽曲リスト
    - `int master_plus_term_id`：Master+のアルバムID
    - `int[] master_plus_live_list`：Master+の楽曲リスト
- `int season_id`：シーズンID
- `array prp_list`：PRP リスト
    - `int live_detail_id`：Live Detail ID
    - `int score`：スコア
    - `double prp`：PRP
- `map live_daily_bonus_list`
    - `array param`：クライアント未使用?
        - `int id`
        - `long start_date`
        - `long end_date`
    - `array drop`：LIVE ドロップリスト
        - `int id`
        - `int start_date`
        - `int end_date`
- `map login_bonus_list`：ログインボーナスリスト（未確認）
    - `array total`
        - `int campaign_id`
        - `int days`
        - `int reward_type`
        - `int reward_id`
        - `int reward_value`
    - `array first`
        - `int campaign_id`
        - `int total_count`
        - `int rap`
        - `int count_num`
        - `string type`
        - `int reward_type`
        - `int reward_id`
        - `int reward_value`
    - `array normal`
        - `int campaign_id`
        - `int total_count`
        - `int rap`
        - `int count_num`
        - `string type`
        - `int reward_type`
        - `int reward_id`
        - `int reward_value`
        - `int bg_rap`
    - `array campaign`
        - `int campaign_id`
        - `int total_count`
        - `int rap`
        - `int count_num`
        - `string type`
        - `int reward_type`
        - `int reward_id`
        - `int reward_value`
    - `array lottery`
        - `int campaign_id`
        - `int total_count`
        - `int rap`
        - `int count_num`
        - `string type`
        - `int reward_type`
        - `int reward_id`
        - `int reward_value`
- `map login_bonus_sp`：クライアント未使用
    - `int count_down_value`
    - `int anv_flag`
- `int present_count`：プレゼントボックスの中身の個数
- `int liked_count`：未受理の Like 数
- `int new_friend_count`：新規同僚数
- `int approval_pending_count`：同僚申請数
- `int comment_count`：新規コメント数
- `array panel_mission_list`：パネルミッションリスト
    - `int viewer_id`：所持主の Viewer ID（クライアント未使用）
    - `int campaign_id`
    - `int rap`
    - `int state`
    - `array detail_list`
        - `int mission_id`
        - `int score`
        - `int state`
- `int mission_clear_num`：ミッションクリア数（もう使用されていない）
- `map lottery_campaign`：宝くじ情報
    - `int sp_campaign_id`
    - `int lottery_period_type`
    - `sp_campaign_type`
- `array open_event_period_list`：開催中のイベント情報
    - `int event_type`：イベントの種類（1：atapon、2：caravan、3：medley、4：party、5：tour）
    - `int event_id`：イベントID
    - `int period`：`eventPeriodType`
    - `int beta_flg`
- `map event_atapon`
    - `map user_detail`
        - `int event_id`
        - `int user_event_point`
        - `int user_event_score`
        - `int point_rank`
        - `int score_rank`
        - `int event_item_id`
        - `int event_item_num`
    - `array stamina_multiple`
        - `int event_id`
        - `int start_time`
        - `int end_time`
        - `int multiple`
- `map event_caravan`
    - `int day_number`
- `map event_medley`
    - `map user_detail`
        - `int event_id`
        - `int user_event_point`
        - `int user_event_score`
        - `int point_rank`
        - `int score_rank`
- `map event_party`
    - `map user_detail`
        - `int event_id`
        - `int user_event_point`
        - `int user_event_score`
    - `array event_unit_list`
        - `int[]`
    - `array event_dress_list`
        - `int[]`
- `array event_unit_list`：なぜかルートの位置にもこのフィールドがある（クライアント未使用）
    - `int[]`：推定
- `map event_tour`
    - `map user_detail`
        - `int event_id`
        - `int user_event_point`
        - `int user_event_score`
- `array album_list`：アルバムリスト
    - `int card_id`：カードID
    - `int love`：経験親愛度
    - `int max_love_flag`：親愛度 Max を取得しているか（1 で true）
- `array story_list`：全ユーザーに解放するスペシャルストーリーのリスト（通常はない?）
    - `int story_id`
    - `int state`
    - `int create_time`
- `array user_story_list`：ユーザーが解放しているストーリーのリスト
    - `int story_id`
    - `int state`
    - `string create_time`
- `array open_jewel_shop_list`：ジュエルショップのリスト（?）
    - `int id`
    - `int user_buy_count`
    - `long buy_end_time`
    - `long sc_use_limit_time`
- `map draw_ssr_gacha`
    - `int id`
    - `bool is_draw`
- `map user_manager_data`
    - `int transition_flag`：1 なら成功
- `array emblem_list`：所持している称号のリスト
    - `int emblem_id`：称号のID
    - `int ex_value`：記入される順位（`null` のことがある）
- `array dress_list`：購入した衣装のリスト
    - `int dress_id`
- `int[] onetime_story_id_list`
- `array add_story_list`
    - `int story_id`
- `map friend_pt_info`：`room/get_liked_history_list` 参照（クライアント未使用）
    - `int liked_count`
    - `int add_pt_count`
    - `int before_friend_pt`
    - `int after_friend_pt`
    - `int discarded_friend_pt`
- `int room_change_bgm_enabled`：クライアント未使用

`schedule_list` は現状トレチケタイム以外実装されていない。

`user_unit_list` および `room_unit_info` における番号は センター→左1→右1→左2→右2 の順。

`string` で示されている日時類はハイフン区切り（`2016-08-23 14:03:17` など）。
