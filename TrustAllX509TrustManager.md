# Insecure TLS/SSL trust manager (TrustAllX509TrustManager)

## 警告されている問題点

SSL/TLS証明書検証を適切に行っていないため、HTTPS通信などに対する中間者攻撃を受けるリスクがあります。

## 対策のポイント

- 証明書検証をしないようにするためにTrustManagerインターフェースを使用しない

## 対策の具体例

特別な証明書検証ロジックが不要であれば、自動でサーバーの証明書検証およびホスト名検証を行ってくれる方法を利用してください。
以下は、URLを指定して接続を確立する中で証明書の検証が行われます。

```java
    try {
        URL url = new URL("https://www.example.com");
        HttpsURLConnection conn = (HttpsURLConnection)url.openConnection();
        conn.connect();
        :
    } catch (...) {
        :
    }
```

デバッグでのみ使用したい接続先を指定したい場合は、Android 7.0(API 24)以降で利用可能なNetwork Security Configurationを使用してください。
詳しくは[「Network Security Configuration | Android Developers」][2]および[「Androidアプリのセキュア設計・セキュアコーディングガイド」][3]を参照してください。

HTTPSで接続するアプリをデバッグする場合には、マニフェストに記載した設定ファイルに&lt;debug-overrides&gt;タグを使用して、デバッグビルドでのみ使用するCAを指定することが出来ます。

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- デバッグビルドのときだけ反映されるので安全 -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="@raw/debug_cas" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

注意: デバッグビルドにするためにマニフェストにandroid:debuggable属性を記述しないでください。詳しくは[HardcodedDebugMode](HardcodedDebugMode.md)の項を参照してください。

## 不適切な例

javax.net.ssl.X509TrustManagerインタフェースを実装するクラスでは、本来checkServerTrustedメソッドおよびcheckClientTrustedメソッドで証明書の検証をします。
しかし、デバッグなどのために証明書を検証しなくすると、中間者攻撃を受けるリスクになります。

```java
    X509TrustManager trustManager = new X509TrustManager() {
        :
        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType)
            throws CertificateException {
            // 何もしない -> どんな証明書でも受け付ける
        }

        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType)
            throws CertificateException {
            // 何もしない -> どんな証明書でも受け付ける
        }
        :
    };

    SSLContext context = SSLContext.getInstance("TLS");
    context.init(
        null,
        new TrustManager[] { trustManager },
        new SecureRandom()
    );

    HttpsURLConnection.setDefaultSSLSocketFactory(context.getSocketFactory());
    HttpsURLConnection conn = (HttpsURLConnection)new URL(url).openConnection(proxy);
    conn.connect();
```

LintはcheckServerTrusted()、checkClientTrusted()に有効なコードが存在しないことを検知すると、それぞれ次のような警告メッセージを出力します。

-   Lint 結果(Warning)  
    "'checkServerTrusted' is empty, which could cause insecure network traffic due to trusting arbitrary TLS/SSL certificates presented by peers"
    "'checkClientTrusted' is empty, which could cause insecure network traffic due to trusting arbitrary TLS/SSL certificates presented by peers"

このチェックは簡易的なもののため、証明書検証の正当性は判断できません。例えば、ログ出力の一行だとしても「有効なコードはある」ということでメッセージは出力されなくなります。
独自TrustManagerを実装する場合は、証明書検証を適切に行えているかを慎重に確認する必要があります。

## 外部リンク

-   [X509TrustManager | Android Developers][1]
-   [Network Security Configuration | Android Developers][2]
-   [Android アプリのセキュア設計・セキュアコーディングガイド　5.4.2.4. 独自の TrustManager を作らない（必須）][3]  
-   [プレス発表　【注意喚起】HTTPSで通信するAndroidアプリの開発者はSSLサーバー証明書の検証処理の実装を][4]


[1]: https://developer.android.com/reference/javax/net/ssl/X509TrustManager.html
[2]: https://developer.android.com/training/articles/security-config.html
[3]: http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#%E7%8B%AC%E8%87%AA%E3%81%AEtrustmanager%E3%82%92%E4%BD%9C%E3%82%89%E3%81%AA%E3%81%84-%EF%BC%88%E5%BF%85%E9%A0%88%EF%BC%89
[4]: https://www.ipa.go.jp/about/press/20140919_1.html