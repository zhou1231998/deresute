# unity3d の Texture2D

unity3d ファイルに含まれている Texture を展開する手法に関して述べる。

TypeTree などの型に関する情報は [バイナリフォーマット](unity3d.md) を参照のこと。

## 展開について

面倒であれば、Unity の Texture2D.EncodeToPNG を使ってしまうのが早い。

ただどうしてもコマンドライン上から動かしにくいので、自前で実装せざるをえないのが現状である。

ある程度手軽に利用できるものとして、[starlight_sync/misc_utils/libahff](https://github.com/summertriangle-dev/starlight_sync/tree/master/misc_utils/libahff) を使う方法もある。

## ビット数の少ないフォーマットの展開について

たとえば RGB565 の場合、R が 5 bit で表現されている。通常は 8 bit で表現したいので、上位 5 bit はそのままビットシフトで得て、下位 3 bit は 5 bit の上位 3 bit を利用して補間する方法をとる。

プログラムで書けば

```c
r = (bin & 0xF800) >> 8
r |= r >> 5
```

のようになる。

RGBA4444 なども同様である。

## 34. ETC_RGB4 の展開について

ETC（Ericsson Texture Compression）はなかなか上手な画像圧縮アルゴリズムになっており、自分で展開するのは少々面倒くさい。

そこで Android SDK を使う方法がある。以下のようにバイナリの前に 16 バイトのヘッダを付けてやればよい。

width および height は actual_width および actual_height を 4 の倍数に切り上げた値になる。なおエンディアンは CPU 依存である。

```
0x0000 | 50 4B 4D 20 31 30 | 'PKM 10'
0x0006 | 00 00             | type = 0
0x0008 | 04 00             | width = 1024
0x000a | 04 00             | height = 1024
0x000c | 04 00             | actual_width = 1024
0x000e | 04 00             | actual_height = 1024
0x0010 | 10 10 10 02 ...   | binary
```

なお [rg-etc1](https://code.google.com/archive/p/rg-etc1/) というライブラリもあり、上の starlight_sync ではこれを利用している。

## TextureFormat 一覧

| Number | TextureFormat                |
|-------:|------------------------------|
|      1 | Alpha8                       |
|      2 | ARGB4444                     |
|      3 | RGB24                        |
|      4 | RGBA32                       |
|      5 | ARGB32                       |
|      6 |                              |
|      7 | RGB565                       |
|      8 |                              |
|      9 |                              |
|     10 | DXT1                         |
|     11 |                              |
|     12 | DXT5                         |
|     13 | RGBA4444                     |
|     14 |                              |
|     15 |                              |
|     16 |                              |
|     17 |                              |
|     18 |                              |
|     19 |                              |
|     20 | WiiI4                        |
|     21 | WiiI8                        |
|     22 | WiiIA4                       |
|     23 | WiiIA8                       |
|     24 | WiiRGB565                    |
|     25 | WiiRGB5A3                    |
|     26 | WiiRGBA8                     |
|     27 | WiiCMPR                      |
|     28 |                              |
|     29 |                              |
|     30 | PVRTC_RGB2                   |
|     31 | PVRTC_RGBA2                  |
|     32 | PVRTC_RGB4                   |
|     33 | PVRTC_RGBA4                  |
|     34 | ETC_RGB4                     |
|     35 | ATC_RGB4                     |
|     36 | ATC_RGBA8                    |
|     37 | BGRA32                       |
|     38 | ATF_RGB_DXT1                 |
|     39 | ATF_RGBA_JPG                 |
|     40 | ATF_RGB_JPG                  |
|     41 | EAC_R                        |
|     42 | EAC_R_SIGNED                 |
|     43 | EAC_RG                       |
|     44 | EAC_RG_SIGNED                |
|     45 | ETC2_RGB4                    |
|     46 | ETC2_RGB4_PUNCHTHROUGH_ALPHA |
|     47 | ETC2_RGBA8                   |
|     48 | ASTC_RGB_4x4                 |
|     49 | ASTC_RGB_5x5                 |
|     50 | ASTC_RGB_6x6                 |
|     51 | ASTC_RGB_8x8                 |
|     52 | ASTC_RGB_10x10               |
|     53 | ASTC_RGB_12x12               |
|     54 | ASTC_RGBA_4x4                |
|     55 | ASTC_RGBA_5x5                |
|     56 | ASTC_RGBA_6x6                |
|     57 | ASTC_RGBA_8x8                |
|     58 | ASTC_RGBA_10x10              |
|     59 | ASTC_RGBA_12x12              |
