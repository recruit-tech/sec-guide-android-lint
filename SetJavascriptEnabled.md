# Using setJavaScriptEnabled (SetJavascriptEnabled)

## 警告されている問題点

WebViewのJavaScriptを有効にしているため、クロスサイトスクリプティングによる攻撃を受けるリスクがあります。

## 対策のポイント

以下のいずれかの対策を実施してください。

- ロードするリソースを信頼できるものに限定する
- JavaScriptを無効にする

## 対策の具体例

JavaScriptを有効にしていると[addJavascriptInterface Called (AddJavascriptInterface)][7]に記載している脆弱性の影響を受ける可能性がありますので、そちらも参照してください。

### リソースを限定する

JavaScriptを有効にしたWebViewを使う場合は、接続先を信頼できるサイトに限定することが重要です。
不用意なリソースをロードしないようにアプリを設計すると共に、ロードするリソースに不正なリンクなどを含まないようにしてください。

```java
    WebView webView = new WebView(context);
    webView.getSettings().setJavaScriptEnabled(true);

    webView.loadUrl("https://trustworthy.example.com");
```

注意: Lintは、JavaScriptを有効（`setJavaScriptEnabled(true);`）にしていることだけを検知して警告メッセージを出力しますので、ロードするリソースを如何に制限しようとメッセージは出力され続けます。クロスサイトスクリプティング（XSS）を懸念してのことですが、XSSの対策については[「OWASP クロスサイトスクリプティング対策チートシート」][6]を参照してください。

接続先を完全に信頼できるサイトに限定することが出来ない場合は、Android 5.0(API 21)以降で使えるWebViewのSafe Browsing[^注釈2]設定を有効にすることで、Googleが管理するブラックリストに含まれるサイトへのアクセスを抑制できます。
詳細は[「What is Safe Browsing?」][4]、アプリでURLの個別検査が必要な場合は[「SafetyNet Safe Browsing API」][5]を参照してください。

```
<manifest>
    <application>
        // Googleが提供するSafe Browsingを有効化
        <meta-data android:name="android.webkit.WebView.EnableSafeBrowsing"
                   android:value="true" />
        ...
    </application>
</manifest>
```

Android 7.0(API 24)以降ではNetwork Security Configurationの機能を使う方法もあります。この機能は、宣言型設定ファイルを使用してネットワークセキュリティの設定をカスタマイズします。

まず、マニフェストに設定ファイルの場所を記載します。

```
<?xml version="1.0" encoding="utf-8"?>
    ...
    <app ...>
        // ネットワークセキュリティ設定を res/xml/network_security_config.xml に記述する
        <meta-data android:name="android.security.net.config"
               android:resource="@xml/network_security_config" />
        ...
    </app>
```

これでnetwork_security_config.xmlに信用する認証局を指定できるようになります。
他にも、デバッグ用の接続先、平文通信の抑制、特定の証明書の強化を設定することができます。
詳しくは[Android DevelopersのAPIガイド][8]を参照してください。

以下に、平文通信を抑制するときの例を示します。
&lt;domain-config&gt;タグのcleartextTrafficPermitted属性をfalseにすることで実現できます。
ただし、この設定がWebViewに適用されるのは、Android 8.0(API 26)以降となります。

```
<?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <domain-config cleartextTrafficPermitted="false">
            <domain includeSubdomains="true">example.com</domain>
        </domain-config>
    </network-security-config>
    :
```

### JavaScriptを無効にする

信頼できないリソースにアクセスする可能性のある場合には、JavaScriptを無効にすることを強く推奨します。
現在のWebViewのデフォルトは「無効」ですが、`setJavaScriptEnabled(false);`で明示的にJavaScriptを無効にしておくことをお薦めします。

```java
import android.app.Activity;
import android.os.Bundle;
import android.webkit.WebView;

public class MyActivity extends Activity {

    private WebView webView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        WebView webView = (WebView)findViewById(R.id.webView);

        // JavaScriptが不要な場合は明示的に無効にする
        webView.getSettings().setJavaScriptEnabled(false);

        webView.loadUrl("https://unknown.example.com/");
    }
}
```


## 不適切な例

設計上、JavaScriptを使用しないにもかかわらず、これを有効にすることは余計なリスクを抱えることになります。

```java
    import android.app.Activity;
    import android.os.Bundle;
    import android.webkit.WebView;

    public class SetJavaScriptEnabled extends Activity {

        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main);
            WebView webView = (WebView)findViewById(R.id.webView);

            // webViewでJavaScriptを有効にする
            webView.getSettings().setJavaScriptEnabled(true);

            webView.loadUrl("https://example.com/");
        }
    }
```

Lintは `setJavaScriptEnabled(true)` という形のメソッド呼び出しを検知すると、次のようなメッセージを出力します：

-  Lint結果(Warning)  
    “Using setJavaScriptEnabled can introduce XSS vulnerabilities into your application, review carefully.”

## 外部リンク

-   [WebView | Android Developers][1]
-   [WebView を使用する上でのセキュリティ対策情報][2]
-   [Androidアプリのセキュア設計・セキュアコーディングガイド][3]
-   [What is Safe Browsing?][4]
-   [SafetyNet Safe Browsing API][5]
-   [OWASP クロスサイトスクリプティング (XSS) 対策チートシート][6]


[1]:https://developer.android.com/reference/android/webkit/WebView.html
[2]:https://developer.android.com/training/articles/security-tips.html\#WebView
[3]:http://www.jssec.org/dl/android\_securecoding.pdf
[4]:https://developers.google.com/safe-browsing/
[5]:https://developer.android.com/training/safetynet/safebrowsing.html
[6]:https://jpcertcc.github.io/OWASPdocuments/CheatSheets/XSSPrevention.html
[7]:AddJavascriptInterface.md
[8]:https://developer.android.com/training/articles/security-config.html


[^注釈1]: javascript:void(0); "接続先サーバーの証明書や公開鍵をアプリ内にあらかじめ保持しておき、接続先のサーバーをアプリ内で保持しているものに限定する機能。"

[^注釈2]: javascript:void(0); "Safe Browsing：Googleが安全でないと判断したサイトにユーザーがアクセスしようとしたときに警告を出す、Googoleが提供するサービス。"