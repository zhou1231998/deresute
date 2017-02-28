# デレステ解析ノート

## 本ノートに関して

このノートはデレステにまつわる解析にもとづく知見をまとめたものである。

逆コンパイルして得られた情報のほか、デレステを解析して作られた様々なウェブサイトにあるソースコードも対象として分析を行い、そこから得られた情報をなるべく整理してまとめたものになる。

デレステは広くユーザー数がおり、解析により作られたウェブサイトも多数存在する一方、第一言語である日本語でまとめられた情報はほとんどなく、手当たり次第に探ることが多いのも現状である。

本ノートはユーザーが広くデレステ内部のデータを分析できるよう、改めてその知見を集合させたものである。

## 二次利用に関して

本ノートにより得られた情報をもとに、ツールやウェブサイトを構築することは、その良心の限り許される。

なお本ノートを焼き直ししたテキストおよび同人誌の存在は、その有用性の観点からいって不適切であろう。

ただし本ノートの他言語への翻訳はその限りではないだろう。

## コンテンツ

- [リソース](resource/index.md)
- [ユーザーデータの通信（API）](user/index.md)

## サポート

[Twitter](https://twitter.com/Ishotihadus) まで。

## 参考文献

理解の上で大いに役立つであろうインターネットという広大な宇宙にいる輝ける星屑たちである。

### デレステ関連

- [アイマス デレステ攻略まとめwiki【アイドルマスター シンデレラガールズ スターライトステージ】 - Gamerch](https://imascg-slstage-wiki.gamerch.com/)
- [Starlight DB Main](https://starlight.kirara.ca/)
    - [summertriangle-dev (The Holy Constituency of the Summer Triangle)](https://github.com/summertriangle-dev)
        - [summertriangle-dev/sparklebox: https://starlight.kirara.ca/](https://github.com/summertriangle-dev/sparklebox)
        - [summertriangle-dev/starlight_sync: automation intensifies](https://github.com/summertriangle-dev/starlight_sync)
        - [summertriangle-dev/more-ss-tools: needful things](https://github.com/summertriangle-dev/more-ss-tools)
- [プロデューサー検索 - deresute.me](https://deresute.me/)
    - [marcan/deresuteme: The code behind https://deresute.me](https://github.com/marcan/deresuteme)
- [hozuki/libcgss: libcgss is a helper library for Idolmaster Cinderella Girls Starlight Stage (CGSS/DereSute). It currently includes HCA audio decoding, and... well, ACB can wait.](https://github.com/hozuki/libcgss)
- [OpenCGSS/DereTore: Music and beatmap authoring toolkit for Idolmaster Cinderella Girls Starlight Stage (CGSS/DereSute). / 偶像大师灰姑娘女孩星光舞台音乐&谱面制作工具箱](https://github.com/OpenCGSS/DereTore)

### そのほかの解析

- Unity 関連
    - [ata4/disunity: An experimental toolset for Unity asset and asset bundle files.](https://github.com/ata4/disunity)
    - [HearthSim/UnityPack: Python deserialization library for Unity3D Asset format](https://github.com/HearthSim/UnityPack)
    - [aki017/assetbundle: unity3d assetbundle](https://github.com/aki017/assetbundle)
- C# 関連
    - [ILSpy](http://ilspy.net/)
    - [aerror2/ILSpy-For-MacOSX: ILSpy for Mac OS X ,Linux and any mono supported platform](https://github.com/aerror2/ILSpy-For-MacOSX)

### オープンにされているフォーマットの仕様など

- [RealTime Data Compression: LZ4 Frame format : Final specifications](http://fastcompression.blogspot.jp/2013/04/lz4-streaming-format-final.html)
- [msgpack/spec.md at master · msgpack/msgpack](https://github.com/msgpack/msgpack/blob/master/spec.md)
