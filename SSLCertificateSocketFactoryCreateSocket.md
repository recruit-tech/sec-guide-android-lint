# Insecure call to SSLCertificateSocketFactory.createSocket() (SSLCertificateSocketFactoryCreateSocket)

## 警告されている問題点

暗号化通信においてホスト名の検証をしていないため、中間者攻撃を受けるリスクがあります。

## 対策のポイント

-   接続先のサーバー証明書のホスト名検証で接続先の正当性を検証する

## 対策の具体例

SSL/TLSによる通信では、サーバー証明書とホスト名を合わせて検証しなければ、接続先を正しく検証することができません。
SSLCertificateSocketFactoryクラスのcreateSocketメソッドが生成するSocketはホスト名指定とIPアドレス指定の2種類がありますが、接続後に署名の検査に加えて、証明書のホスト名の検証もあわせて行われるホスト名指定を使用してください。

```java
import java.io.IOException;
import javax.net.ssl.SSLSocket;
import android.app.Activity;
import android.os.Bundle;
import android.net.SSLCertificateSocketFactory;

public class MyApp extends Activity {

    private SSLSocket sock = null;

    public boolean connectToHost (String host, int port) {
        SSLCertificateSocketFactory sf = (SSLCertificateSocketFactory) SSLCertificateSocketFactory.getDefault(0);

        try {
            // ホスト名を指定して socket を作成すると証明書のホスト名検証も実施してくれる
            sock = (SSLSocket) sf.createSocket(host, port);
        } catch (IOException e) {
            android.util.Log.e("MyApp", e.getMessage());
            return false;
        }
        android.util.Log.d("MyApp", "Success creating socket");
        return true;
    }
}
```

## 不適切な例

デバッグや開発中にホスト名検証を省略するためにIPアドレス指定メソッドすると、そのままリリースされるリスクがあります。
Android 7.0(API 24)以降では[「Network Security Configuration」][3]を利用できるので、そちらも参考にしてください。


```java
import java.io.IOException;
import javax.net.ssl.SSLSocket;
import java.net.InetAddress;
import android.app.Activity;
import android.os.Bundle;
import android.net.SSLCertificateSocketFactory;

public class MyApp extends Activity {

    private SSLSocket sock = null;

    public boolean connectToHost (InetAddress inet, int port) {
        SSLCertificateSocketFactory sf = (SSLCertificateSocketFactory) SSLCertificateSocketFactory.getDefault(0);

        try {
            // IPアドレスを指定して socket を作成すると証明書のホスト名検証は実施されない
            sock = (SSLSocket) sf.createSocket(inet, port);
        } catch (IOException e) {
            android.util.Log.e("MyApp", e.getMessage());
            return false;
        }
        android.util.Log.d("MyApp", "Success creating socket");

        return true;
    }
}
```

Lintは上のようにSSLCertificateSocketFactory.createSocket()がInetAddressを引数として呼び出しを検出すると次のようなメッセージを出力します。

-   Lint 結果(Warning)  
    "Use of‘SSLCertificateSocketFactory.createSocket()’ with an InetAddress parameter can cause insecure network traffic due to trusting arbitrary hostnames in TLS/SSL certificates presented by peers."

## 外部リンク

- [SSLCertificateSocketFactory | Android Developers][1]
- [Transport Layer Security | Wiki Pedia][2]

[1]:https://developer.android.com/reference/android/net/SSLCertificateSocketFactory.html
[2]:https://ja.wikipedia.org/wiki/Transport_Layer_Security
[3]: https://developer.android.com/training/articles/security-config.html


