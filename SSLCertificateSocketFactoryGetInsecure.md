# Call to SSLCertificateSocketFactory.getInsecure() (SSLCertificateSocketFactoryGetInsecure)

## 警告されている問題点

暗号化通信においてホスト名の検証をできないSocketを使用しているため、中間者攻撃[^1]を受けるリスクがあります。

## 対策のポイント

- 接続先のサーバー証明書のホスト名検証で、接続先の正当性を検証する

## 対策の具体例

接続先の正当性検証は相当の専門知識を必要としますので、自作せずに既存の仕組みを正しく利用するようにしてください。

### HttpsURLConnectionクラスを使用する

HttpsURLConnectionクラスはデフォルトでセキュリティチェックが有効なSocketFactoryおよびSocketを使用しているので、安全な通信が可能になります。

```java
import javax.net.ssl.HttpsURLConnection;

public class SSLCertificateSocketFactoryExample {
    public void connect() {
        try {
            URL url = new URL("https://example.com/");
            HttpsURLConnection conn = (HttpsURLConnection)url.openConnection();
        } catch (...) {
            :
        }
    }
}
```

### セキュリティチェックが有効なSSLCertificateSocketFactoryを使用する

SSLSocketを直接生成する場合は、getDefaultメソッドによって取得されるSSLCertificateSocketFactoryを使用してください。
getDefaultメソッドで得られるセキュリティチェックが有効なインスタンスは、安全なSocketを生成できます。

```java
import android.net.SSLCertificateSocketFactory;

public class SSLCertificateSocketFactoryExample {
    public void connect() {
        SSLCertificateSocketFactory sf = (SSLCertificateSocketFactory)SSLCertificateSocketFactory.getDefault(0);

        try {
            sf.createSocket("example.com", 443);
            :
        } catch (...) {
            :
        }
    }
}
```

ただし、createSocketの引数にホスト名ではなく接続先InetAddress(IPアドレス)を指定すると、ホスト名の検証を行わない安全ではないSocketが生成されるので注意してください。
これについての詳細は [Insecure call to SSLCertificateSocketFactory.createSocket()][4]の項を参照してください。

## 不適切な例

下の例のようにgetInsecureメソッドで取得したセキュリティチェックが無効なSocketFactoryを使用すると、証明書の検証を行わなくなるため正しく接続先を確認できません。

```java
import android.net.SSLCertificateSocketFactory;
import javax.net.ssl.HttpsURLConnection;

public class SSLCertificateSocketFactoryExample {
    public void connect() {
        HttpsURLConnection.setDefaultSSLSocketFactory(SSLCertificateSocketFactory.getInsecure(0,null));
        try {
             URL url = new URL("https://example.com/");
             HttpsURLConnection conn = (HttpsURLConnection)url.openConnection();
            :
        } catch (...) {
            :
        }
    }
}
```

デバッグ時など、事情により正式なサーバーを用いたHTTPS接続ができずにセキュリティチェックを無効とすることがありますが、リリース後にその実装が残る危険があります。
Android 7.0(API 24)以降では[Network Security Configuration][2]を利用できるので、そちらも参考にしてください。

Lintは、SSLCertificateSocketFactoryクラスのgetInsecureメソッドの呼び出しを検知すると、次のようなメッセージを出力します。

- Lint出力(Warning)  
  "Use of SSLCertificateSocketFactory.getInsecure() can cause insecure network traffic due to trusting arbitrary TLS/SSL certificates presented by peers."

## 外部リンク

- [SSLCertificateSocketFactory | Android Developers][1]
- [ネットワークセキュリティ構成 | Android Developers][2]
- [Security with HTTPS and SSL | Android Developers][3]

[1]:https://developer.android.com/reference/android/net/SSLCertificateSocketFactory.html
[2]:https://developer.android.com/training/articles/security-config.html
[3]:https://developer.android.com/training/articles/security-ssl.html
[4]:SSLCertificateSocketFactoryCreateSocket.md


[^1]: dummy "中間者攻撃：通信している2人のユーザーの間に第三者が介在し、送信者と受信者の両方になりすまして、ユーザーが気付かないうちに通信を盗聴したり、制御したりすること"