# addJavascriptInterface Called (AddJavascriptInterface)

## 警告されている問題点

WebViewの脆弱性をついた攻撃により、アプリ機能の不正利用や、情報漏洩・改竄のリスクがあります。

## 対策のポイント

AddJavascriptInterfaceを使用する際は、以下のいずれかの対策を実施してください。

- アプリがサポートするバージョンを脆弱性のないものに設定する
- WebViewがロードするリソースを信頼できるものに限定する

## 対策の具体例

### サポートするAndroidバージョンを脆弱性のないものに設定する

[CVE-2012-6636][3]、[CVE-2013-4710][4]、および[CVE-2014-7224][7]の致命的な脆弱性がAndroid 4.2(API 17)で修正されたので、これ以前のバージョンをサポートすべき合理的な理由がない場合には、minSdkVersionを17以上に設定してください。  
Android 4.2からは、Javaコード上にJavascriptInterfaceアノテーションのないメソッドにはJavaScriptからアクセスできなくなったため、リフレクション[^注釈1]により任意のコードを実行されるリスクがなくなります。

サポートする最小バージョン(minSdkVersion)の変更は、Android Studioで以下のように行います。

```
1. Tools -> Android -> Android SDK Default Setting 画面でSDKインストール
   （指定するバージョンがインストール済なら1.の作業は不要）  
2. File -> Project Structure… 画面から、アプリのFlavorsタグのMin Sdk Versionを変更
```

### ロードするリソースを信頼できるものに限定する

不正なJavaScriptを含まない信頼できるリソースのみを読み込むようにしてください。読み込むリソースを自社の管理の及ぶ範囲や、Google社など信頼できる第三者が提供するもののみにすることで、攻撃されるリスクを抑えられます。

```java
public class ActivitySample extends Activity {
    // JavaScript実行用クラス
    class CallFromJavaScript {
        @JavascriptInterface
        public methodA() {...}

        // API 17以降では@JavascriptInterfaceのないメソッドはJavaScriptから実行できない
        public methodB() {...}
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        WebView webView = new WebView(this);
        setContentView(webView);

        // JavaScriptを有効にする
        webView.getSettings().setJavaScriptEnabled(true);

        // JavaScriptから呼び出し可能にするJavaオブジェクトを設定する
        webView.addJavascriptInterface(new CallFromJavaScript(), "callMe");

        webView.setWebViewClient(new WebViewClient(){
            @TargetApi(Build.VERSION_CODES.N)
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                // assetsディレクトリ以下以外のファイルには遷移しないようにする
                return !request.getUrl().toString().startsWith("file:///android_asset/");
            }
    
            // 本メソッドはAPI 24で非推奨となった
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                // assetsディレクトリ以下以外のファイルには遷移しないようにする
                return !url.startsWith("file:///android_asset/");
            }
        }

        // 信頼できるリソースをロードする
        webView.loadUrl("file:///android_asset/index.html");
    }
}
```

Android 4.1(API 17)以下のバージョンをサポートする場合は、ロードするリソースの検証が行えないためにLintはメッセージを出力し続けます。  

## 不適切な例

検索結果や広告に紛れている悪意のあるサイトをロードしてしまうケースは少なくありません。
JavaScriptが有効になっている状態で悪意のあるサイトをロードすると、任意のコードを実行されてしまうリスクがあります。

```java
public class ActivitySample extends Activity {
    class CallFromJavaScript {
        @JavascriptInterface
        public methodA() {...}

        public methodB() {...}
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        WebView webView = new WebView(this);
        webView.getSettings().setJavaScriptEnabled(true); // JavaScriptを有効にする

        // JavaScriptから呼び出し可能にするJavaオブジェクトを設定する
        webView.addJavascriptInterface(new CallFromJavaScript(), "callMe");

        // 信頼できないリソースにアクセスする
        webView.loadUrl(untrustUrl);

        setContentView(webView);
    }
}
```

Lintは、minSdkVersionが17未満でaddJavascriptInterface()メソッドの呼び出しを検知すると、次のようなメッセージを出力します。

- Lint出力(Warning)  
“\`WebView.addJavascriptInterface\` should not be called with minSdkVersion &lt; 17 for security reasons: JavaScript can use reflection to manipulate application”

注意: Android 4.1(API 16)以下のバージョンでは、開発者がaddJavascriptInterfaceメソッドを使用していない場合でも、フレームワークがデフォルトで呼び出していることがあるため危険です。（[CVE-2013-4710][4]、[CVE-2014-7224][7]）

## 外部リンク

- [WebView | Android Developers][1]
- [WebSettings | Android Developers][2]
- [CVE-2012-6636][3]
- [CVE-2013-4710][4]
- [スマートフォンアプリへのブラウザ機能の実装に潜む危険――WebViewクラスの問題について][5]  
    addJavascriptInterfaceメソッドの使い方によっては脆弱性を作りこんでしまうという指摘(2012年)
- [Two New Attack Vectors to Aggravate the Android addJavascriptInterface RCE Issue (CVE-2014-7224)][7]  
    addJavascriptInterfaceメソッドの呼び出しをフレームワークが無条件に行う脆弱性についての記載

[1]:https://developer.android.com/reference/android/webkit/WebView.html
[2]:https://developer.android.com/reference/android/webkit/WebSettings.html
[3]:http://www.cvedetails.com/cve/CVE-2012-6636/
[4]:http://www.cvedetails.com/cve/CVE-2013-4710/
[5]:https://codezine.jp/article/detail/6618
[7]:http://daoyuan14.github.io/news/newattackvector.html


[^注釈1]: javascript:void(0); "リフレクション：プログラム実行時に、クラス名やメソッド名を動的に指定して実行(アクセス)するJavaの機能"
