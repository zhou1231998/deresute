# unity3d ファイルのバイナリフォーマット

unity3d ファイルにあるアセットは Unity で読み込んでしまえばよいのだが、実際に画像を抽出するなどの操作をする場合は Unity ではなく自ら実装したくなるのが世の常である。

unity3d ファイルのバイナリフォーマットはあまり一般的に広く知られていないため、ここで解説する。

LZ4 は解凍した後とする。

バイナリフォーマットはすべてビッグエンディアンである。また String 型は最後に NULL 文字が含まれる（このページでは省略する）。

なおこのページでは UnityRaw フォーマット以外は扱わない（他に UnityWeb フォーマットおよび UnityFS フォーマットがある）。

本ページの執筆にあたっては [HearthSim/UnityPack](https://github.com/HearthSim/UnityPack) を参考にした。特別な理由（たとえば宗教上の理由で Python が使えない場合など）がなければこのライブラリを使うのがよいだろう。

## ヘッダ

ファイルの先頭から。

- String Signature：シグニチャ（例：UnityRaw）
- Int32 FormatVersion：フォーマット番号（例：3）
- String UnityVersion：プレイヤーの Unity バージョン（例：5.x.x）
- String GeneratorVersion：生成に使われた Unity バージョン（例：5.1.2f1）
- UInt32 FileSize：ファイルサイズ（例：2113888）
- Int32 HeaderSize：ヘッダサイズ（例：60 = 0x3C）
- Int32 FileCout：ファイル数（例：1）
- Int32 BundleCount：バンドル数（例：1）
- FormatVersion が 2 以上のときのみ
    - UInt32 BundleSize：（解凍前の）バンドルのサイズ（例：2113828）
- FormatVersion が 3 以上のときのみ
    - UInt32 UncompressedBundleSize：解凍後のバンドルのサイズ（例：2113828）
- HeaderSize が 60 以上のときのみ
    - UInt32 CompressedFileSize：（例：2113888）
    - UInt32 AssetHeaderSize：（例：52 = 0x34）

ファイルの先頭から HeaderSize バイト進めて。

- Int32 AssetCount：アセット数（例：1）

## アセット

本来はバンドルなのだが、UnityPack がアセットと混同しているのでアセットと表記する。

ヘッダを読み込んだ位置から開始する（すなわち HeaderSize + 4 バイトから）。アセットが複数ある場合にはピッタリ連続する。

- String Name：アセット名（例：CAB-6d731df8b17383698b33838f59d330fb）
- UInt32 AssetHeaderSize：アセットのヘッダサイズ（例：52 = 0x34）
- UInt32 AssetSize：アセットサイズ（例：2113776）

さらにアセットを読み込み始めた場所から AssetHeaderSize - 4 バイト進める（最初のアセットであれば HeaderSize + AssetHeaderSize になる）。ここでの例ではファイルの先頭から 0x70 になる。

- UInt32 MetadetaSize：メタデータのサイズ（例：2206 = 0x089e）
- UInt32 FileSize：アセットのサイズ（例：2113776）
- UInt32 Format：フォーマット（例：15）
- UInt32 DataOffset：データのオフセット（例：4096）
- Format が 9 以上のときのみ
    - UInt32 Endianness：エンディアン（0ならリトルエンディアン）

ここからは先ほど読んだエンディアンで書かれる（Format が 8 以下のときはかならずビッグエンディアンになる）。なお次のブロックは TypeMetadata 構造体と呼ばれる。

- String GeneratorVersion：生成に使われた Unity バージョン（例：5.1.2f1）
- UInt32 TargetPlatform：ターゲットのプラットフォーム（例：13）
- Format が 13 以上のとき
    - Byte HasTypeTrees：TypeTree を持つか（例：1）
    - Int32 TypeTreeCount：TypeTree の数（例：2）
    - HasTypeTrees が true の場合、以下 TypeTreeCount の回数
        - Int32 ClassId：クラスID（例：28）
        - Byte[] Hash：ハッシュ値（ClassId が 0 未満であれば 0x20 バイト、0 以上であれば 0x10 バイト）
        - TypeTree Tree
- Format が 12 以下のとき
    - Int32 TypeTreeCount：TypeTree の数
    - 以下 TypeTreeCount の回数
        - Int32 ClassId：クラスID
        - TypeTree Tree

TypeTrees はハッシュになっており、クラスID => 構造体 の形をとる。TypeTree 構造体は以下の形をしている。

- Format が 11 ではない 10 以上のとき
    - UInt32 NumNodes：ノード数（例：24）
    - UInt32 BufferBytes：バッファバイト数（例：240）
    - 以下 NumNodes の回数（それぞれ 24 = 0x18 バイト）
        - Int16 Version：バージョン
        - Byte Depth：ノードの深さ
        - Byte IsArray：配列か
        - Int32 Type：型名
        - Int32 Name：名前
        - Int32 Size：サイズ
        - UInt32 Index：インデックス
        - Int32 Flags：フラグ
    - Byte[BufferBytes] Buffer：バッファ
- Format が 9 以下もしくは 11 のとき
    - String Type
    - String Name
    - Int32 Size
    - Int32 Index
    - Int32 IsArray：Boolean 値
    - Int32 Version
    - Int32 Flags
    - UInt32 NumNodes：子ノードの個数
    - 以下 NumNodes の回数
        - 再帰的にこれらのフィールドが続く

ノードにある `Type` および `Name` はテーブルのオフセット値で、以下の条件で生成する。

- 0 未満のとき
    - 一番上のビットを 0 にしたものを値とする
    - テーブルはあらかじめ用意しておく（[unity3d の文字列テーブル](unity3d-string.md) 参照）
    - 例：0x8000036a -> 0x036a -> Texture2D
- 0 以上のとき
    - そのままの値
    - テーブルはバッファを使用する

TypeTree はその名の通り木構造になっている。Depth の個数だけ親がいる、すなわち自分の深さが Depth になる。親は直前に存在する1つ小さい Depth のノードになる。Format が古い場合には単純に再帰的に生成すればよいだけである。

さて、グダグダと TypeTree に気を取られたが、TypeMetadata 構造体が終わったところで次のバイナリに進む。

- Format が 7 以上 13 以下のとき
    - UInt32 LongObjectIds：Boolean 値
- UInt32 NumObjects：オブジェクト数（例：2）
- 以下 NumObjects の回数（これらがオブジェクトのメタデータ部分になる）
    - Format が 14 以上のときここで 4 バイトにアライメントをとる
    - LongObjectIds が存在して true もしくは Format が 14 以上のとき
        - Int64 PathId：パスID（例：8902842285466946048）
    - Format が 6 以下もしくは LongObjectIds が存在して false のとき（上の条件以外）
        - Int32 PathId：パスID
    - UInt32 DataOffset：データオフセット（例：168）
    - UInt32 Size：データサイズ（例：2109512）
    - Int32 TypeId：型ID（例：28）
    - Int16 ClassId：クラスID（例：28）
    - Format が 10 以下のとき
        - Int16 IsDestroyed
    - Format が 11 以上のとき
        - Int16 Reserved
    - Format が 15 以上のとき
        - Byte Reserved
- Format が 11 以上のとき
    - UInt32 NumAdds：追加オブジェクトの個数（例：0）
    - 以下 NumAdds の回数（これらが追加オブジェクトのメタデータ部分になる）
        - Format が 14 以上のときここで 4 バイトにアライメントをとる
        - LongObjectIds が存在して true もしくは Format が 14 以上のとき
            - Int64 Id：ID
        - Format が 6 以下もしくは LongObjectIds が存在して false のとき（上の条件以外）
            - Int32 Id：ID
        - Int32：?
- Format が 6 以上のとき
    - UInt32 NumRefs：アセットリファレンスの個数（例：0）
    - 以下 NumRefs の回数
        - String AssetPath
        - UInt16 GUID
        - Int32 Type
        - String FilePath

これでオブジェクトのすべての情報が読み込み終わる。

オブジェクトにはそれぞれ TypeTree が割り当てられる。TypeMetadata 内の ClassId にオブジェクトの TypeId もしくは ClassId と同じものがある場合、対応する TypeTree がオブジェクトの TypeTree になる。存在しない場合はデフォルトの設定からオブジェクトの ClassId と同じ ClassId の TypeTree を割り当てる（__TypeId ではない__）。

## オブジェクト

オブジェクトの位置は、アセットの MetadetaSize の位置からさらにアセットの DataOffset およびオブジェクトの DataOffset を進めた位置にある。上の例では 0x70 + 0x1000 + 0xa8 = 0x1118 になる。

あとは対応する TypeTree の順序で読み込んで行けばよい。データのサイズは TypeTree で定義されている Size になる（-1 の場合もある）。

ただし IsArray が true の場合（基本的に Array 型）は、子ノードに size と data フィールドがあり、size の個数分 data フィールドがある形になる。

文字列は Array を中に内包する形で実現している。たとえば、

- string m_Name
    - Array Array
        - int size
        - char data

のような形である。

なお数値型や真偽値型などはわかりきっているので、[文字列テーブル](unity3d-string.md) を確認しつつ実装するとよい。

## Texture2D 型

一番抽出されがちな画像は Texture2D 型である。Texture2D 型は TypeTree が以下のような構造になっている。

- Texture2D Base
    - string m_Name
        - Array Array
            - int size
            - char data
    - int m_Width
    - int m_Height
    - int m_CompleteImageSize
    - int m_TextureFormat
    - bool m_MipMap
    - bool m_IsReadable
    - bool m_ReadAllowed
    - int m_ImageCount
    - int m_TextureDimension
    - GLTextureSettings m_TextureSettings
        - int m_FilterMode
        - int m_Aniso
        - float m_MipBias
        - int m_WrapMode
    - int m_LightmapFormat
    - int m_ColorSpace
    - TypelessData image data
        - int size
        - UInt8 data

実際のバイナリデータは image data にあって、そのフォーマットは m_TextureFormat で決まっている。フォーマットは 7（RGB565）、13（RGBA4444）、34（ETC_RGB4）あたりが使われている。フォーマットの一覧およびそれぞれのバイナリフォーマットは [Unity の unity3d の Texture2D](unity3d-texture2d.md) を参照されたい。

細かい話は [スクリプトリファレンス](https://docs.unity3d.com/560/Documentation/ScriptReference/TextureFormat.html) を参考にすればよい。
