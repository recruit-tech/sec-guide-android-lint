# Insecure HostnameVerifier (AllowAllHostnameVerifier)

## 警告されている問題点

SSL/TLS証明書のホスト名検証を正しく行っていないため、HTTPS通信などに対する中間者攻撃[^1]を受けるリスクがあります。

## 対策のポイント

- 証明書検証を省略するためにAllowAllHostnameVerifierクラスなどを使用しない

## 対策の具体例

SSL/TLS通信においては、サーバーの証明書検証およびホスト名検証を必ず行うようにしてください。
具体的には[TrustAllX509TrustManager](TrustAllX509TrustManager.md)の項を参照してください。

## 不適切な例

- org.apache.http.conn.ssl.SSLSocketFactory[^注釈]クラスのsetHostnameVerifierメソッド
- javax.net.ssl.HttpsURLConnectionクラスのsetHostnameVerifierメソッド

上記メソッドの引数に以下のものを指定した場合、ホスト名検証が行われないために意図せぬサーバーに接続してしまう可能性があります。

- org.apache.http.conn.ssl.AllowAllHostnameVerifier[^注釈]クラスのインスタンス
- org.apache.http.conn.ssl.SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER

```java
    try {
        URL url = new URL("https://www.example.com");
        HttpsURLConnection conn = (HttpsURLConnection)url.openConnection();
        conn.setHostnameVerifier(new AllowAllHostnameVerifier()); // ホスト名検証が行われなくなる
        conn.connect();
        :
    } catch (IOException e) {
        :
    }
```

Lintは、AllowAllHostnameVerifierの使用を検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "Using the AllowAllHostnameVerifier HostnameVerifier is unsafe because it always returns true, which could cause insecure network traffic due to trusting TLS/SSL server certificates for wrong hostnames"

Lintは、ALLOW_ALL_HOSTNAME_VERIFIERの使用を検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "Using the ALLOW_ALL_HOSTNAME_VERIFIER HostnameVerifier is unsafe because it always returns true, which could cause insecure network traffic due to trusting TLS/SSL server certificates for wrong hostnames"

## 外部リンク

-   [プレス発表　【注意喚起】HTTPSで通信するAndroidアプリの開発者はSSLサーバー証明書の検証処理の実装を][1]
-   [Network Security Configuration | Android Developers][2]
-   [Android アプリのセキュア設計・セキュアコーディングガイド][3]  
    [「5.4.2.4. 独自の TrustManager を作らない（必須）」][4]  
    [「5.4.3.7. Network Security Configuration」][5]
    

[1]: https://www.ipa.go.jp/about/press/20140919_1.html
[2]: https://developer.android.com/training/articles/security-config.html
[3]: https://www.jssec.org/dl/android_securecoding/
[4]: https://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#%E7%8B%AC%E8%87%AA%E3%81%AEtrustmanager%E3%82%92%E4%BD%9C%E3%82%89%E3%81%AA%E3%81%84-%EF%BC%88%E5%BF%85%E9%A0%88%EF%BC%89
[5]: https://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#network-security-configuration

[^注釈]: javascript:void(0); "API 22以降deprecated"
[^1]: dummy "中間者攻撃：通信している2人のユーザーの間に第三者が介在し、送信者と受信者の両方になりすまして、ユーザーが気付かないうちに通信を盗聴したり、制御したりすること"