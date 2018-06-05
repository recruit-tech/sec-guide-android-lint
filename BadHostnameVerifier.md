# Insecure HostnameVerifier (BadHostnameVerifier)

## 警告されている問題点

SSL/TLS証明書のホスト名検証を正しく行っていないため、HTTPS通信などに対する中間者攻撃を受けるリスクがあります。

## 対策のポイント

- 独自のHostnameVerifierを作らない

## 対策の具体例

独自のHostnameVerifierを安全に実装することはもちろん可能ですが、暗号処理や暗号通信に十分な知識をもった技術者でなければミスを作り込む危険性があるため、既存の安全な仕組みでサーバーの証明書検証およびホスト名検証を行うようにしてください。
具体的には[TrustAllX509TrustManager](TrustAllX509TrustManager.md)の項を参照してください。

## 不適切な例

証明書検証を省略するためにjavax.net.ssl.HostnameVerifierインタフェースのサブクラスでverify()メソッドで常にtrueを返すと、中間者攻撃を受けるリスクがあります。

```java
    HostnameVerifier allowAllHV = new HostnameVerifier() {
        @Override
        public boolean verify(String hostname, SSLSession session) {
            return true;  // どんなホスト名でも信頼する
        }
    };
```

LintはHostnameVerifierのverifyメソッドが常にtrueを返す実装であることを検知すると、次のようなメッセージを出力します。

-   Lint結果(Warning)  
    "\`verify\` always returns \`true\`, which could cause insecure network traffic due to trusting SSL/TLS server certificates for wrong hostnames"

このチェックは簡易的なもののため、証明書検証の正当性は判断できません。
独自HostnameVerifierを実装する場合は、証明書検証を適切に行えているかを慎重に確認する必要があります。

## 外部リンク

-   [プレス発表　【注意喚起】HTTPSで通信するAndroidアプリの開発者はSSLサーバー証明書の検証処理の実装を][1]
-   [HTTPS および SSL によるセキュリティ | Android Developers「ホスト名の確認に関する一般的な問題」][2]
-   [Network Security Configuration | Android Developers][3]
-   [『Android アプリのセキュア設計・セキュアコーディングガイド』][4]  
    「5.4.2.4. 独自の TrustManager を作らない（必須）」  
    「5.4.3.7. Network Security Configuration」


[1]: https://www.ipa.go.jp/about/press/20140919\_1.html
[2]: https://developer.android.com/training/articles/security-ssl.html\#CommonHostnameProbs
[3]: https://developer.android.com/training/articles/security-config.html
[4]: https://www.jssec.org/dl/android\_securecoding.pdf