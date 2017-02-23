# API通信の実装

サンプルとして Ruby での実装を示す。Python での実装は [marcan/deresuteme](https://github.com/marcan/deresuteme) が参考になる。

## Cryptographer（lolfuscate）

非常に単純な実装である。本家の実装は多少複雑であるが、やっていることは同じである。

```rb
def encode(s)
    '%04x' % s.length +
        s.chars.map{|c| '%02d' % rand(100) + (c.ord + 10).chr + rand(10).to_s}.join +
        (0...32).map{rand(10)}.join
end

def decode(s)
    return s if s.length < 4
    (0...s[0, 4].to_i(16)).map{|i| (s[i * 4 + 6].ord - 10).chr}.join
end
```

## Rijndael 256bit

国際規格である [AES](https://ja.wikipedia.org/wiki/Advanced_Encryption_Standard) とは異なり、デレステではブロック長が 256bit であること、またラウンド処理や鍵拡大処理などが行われていないことから、OpenSSL などは使えず直接 Rijndael を触る必要がある。

C# では `System.Security.Cryptography.RijndaelManaged` を用いることで CBC を実装せずに済むが、そのようなライブラリが標準にない場合は自前で実装せざるを得ない。

Ruby では `crypt` という gem が利用できる。

CBC は自前で実装している。詳しくは [暗号利用モード - Wikipedia](https://ja.wikipedia.org/wiki/%E6%9A%97%E5%8F%B7%E5%88%A9%E7%94%A8%E3%83%A2%E3%83%BC%E3%83%89) などを参照されたい。

```rb
require 'crypt/rijndael'

def encrypt_rj256(s, key, iv)
    r = Crypt::Rijndael.new(key, key.length * 8, iv.length * 8)
    s += "\0" * (iv.length - s.length % iv.length) if s.length % iv.length > 0
    blocks = s.each_char.each_slice(iv.length).map(&:join)
    out = [iv]
    blocks.each{|b| out << r.encrypt_block(b ^ out.last)}
    out[1..-1].join
end

def decrypt_rj256(s, key, iv)
    r = Crypt::Rijndael.new(key, key.length * 8, iv.length * 8)
    blocks = s.each_char.each_slice(iv.length).map(&:join)
    out = [iv]
    blocks.each{|b| out << (r.decrypt_block(b) ^ out.last)}
    out[1..-1].join.split("\0").first
end
```

## 全体実装

実際に API を叩くクライアントを Ruby で作ると以下のようになる。

```rb
require 'crypt/rijndael'
require 'msgpack'
require 'base64'
require 'httpclient'

class ApiClient
    def initialize(udid, user_id, viewer_id)
        @udid = udid
        @user_id = user_id
        @viewer_id = viewer_id
        @sid = viewer_id + udid
        @res_ver = '10023910'
        @app_ver = '2.7.2'
    end

    def b64e(s)
        Base64::strict_encode64(s)
    end

    def b64d(s)
        Base64::strict_decode64(s)
    end

    def encode(s)
        '%04x' % s.length +
            s.chars.map{|c| '%02d' % rand(100) + (c.ord + 10).chr + rand(10).to_s}.join +
            (0...32).map{rand(10)}.join
    end

    def decode(s)
        return s if s.length < 4
        (0...s[0, 4].to_i(16)).map{|i| (s[i * 4 + 6].ord - 10).chr}.join
    end

    def encrypt_rj256(s, key, iv)
        r = Crypt::Rijndael.new(key, key.length * 8, iv.length * 8)
        s += "\0" * (iv.length - s.length % iv.length) if s.length % iv.length > 0
        blocks = s.each_char.each_slice(iv.length).map(&:join)
        out = [iv]
        blocks.each{|b| out << r.encrypt_block(b ^ out.last)}
        out[1..-1].join
    end

    def decrypt_rj256(s, key, iv)
        r = Crypt::Rijndael.new(key, key.length * 8, iv.length * 8)
        blocks = s.each_char.each_slice(iv.length).map(&:join)
        decrypted = blocks.map{|b| r.decrypt_block(b)}.join
        ((iv + s)[0, decrypted.length] ^ decrypted).split("\0").first
    end

    def call(path, args)
        args['timezone'] = '09:00:00'
        vid_iv = (0...32).map{rand(10)}.join
        args['viewer_id'] = vid_iv + Base64::strict_encode64(encrypt_rj256(@viewer_id, "s%5VNQ(H$&Bqb6#3+78h29!Ft4wSg)ex", vid_iv))
        msgpack = MessagePack::Packer.new({:compatibility_mode => true}).write(args).flush
        plain = b64e(msgpack.to_s)
        key = b64e((0...32).map{'%x' % rand(65536)}.join)[0...32]
        msg_iv = @udid.gsub('-', '')
        body = b64e(encrypt_rj256(plain, key, msg_iv) + key)
        header = {
            'Host' => 'game.starlight-stage.jp',
            'User-Agent' => 'BNEI0242/' + @app_ver + ' CFNetwork/758.3.15 Darwin/15.4.0',
            'Content-Type' => 'application/x-www-form-urlencoded',
            'Content-Length' => body.bytesize,
            'Connection' => 'keep-alive',
            'Accept' => '*/*',
            'Accept-Encoding' => 'gzip, deflate',
            'Accept-Language' => 'en-us',
            'X-Unity-Version' => '5.1.2f1',
            'UDID' => encode(@udid),
            'USER_ID' => encode(@user_id),
            'SID' => Digest::MD5::hexdigest(@sid + 'r!I@nt8e5i='),
            'PARAM' => Digest::SHA1::hexdigest(@udid + @viewer_id + path + plain),
            'DEVICE' => '1',
            'APP_VER' => @app_ver,
            'RES_VER' => @res_ver,
            'DEVICE_ID' => 'E0F4586D-9882-4DC0-A7E2-7F749244CA1C',
            'DEVICE_NAME' => 'iPod7,1',
            'GRAPHICS_DEVICE_NAME' => 'Apple A8 GPU',
            'IP_ADDRESS' => '192.168.2.100',
            'PLATFORM_OS_VERSION' => 'iPhone OS 9.3',
            'CARRIER' => '',
            'KEYCHAIN' => '727238026'
        }

        client = HTTPClient.new
        response = client.post_content('http://game.starlight-stage.jp' + path, body, header)
        res_body = b64d(response)
        res_plain = decrypt_rj256(res_body[0...-32], res_body[-32, 32], msg_iv)
        pack = MessagePack::unpack(b64d(res_plain))
        @sid = pack['data_headers']['sid']
        pack
    end
end

client = ApiClient.new('219b8ff9-3d02-4df1-b7fc-dc5bbd560003', '123456789', '123456789')

hash = {'app_type' => 0,
    'campaign_data' => '',
    'campaign_sign' => 'da39a3ee5e6b4b0d3255bfef95601890afd80709',
    'campaign_user' => 123456
}

p client.call('/load/check', hash)
```
