# addJavascriptInterface Called (AddJavascriptInterface)

## 警告されている問題点

WebViewの脆弱性に起因して、アプリの情報が漏洩・改竄されたり、機能が不正に利用されるリスクがあります。

## 対策のポイント

AddJavascriptInterfaceを使用する際は、以下のいずれかの対策を実施してください。

- アプリがサポートするバージョンを脆弱性のないものに設定する
- WebViewがロードするリソースを信頼できるものに限定する

## 対策の具体例

### サポートするAndroidバージョンを脆弱性のないものに設定する

Android 4.2(API 17)からは、[CVE-2012-6636][3]、[CVE-2013-4710][4]、および[CVE-2014-7224][7]の致命的な脆弱性が修正されているので、これ以前のバージョンをサポートすべき合理的な理由がない場合には、minSdkVersionを**17以上**に設定してください。  
Android 4.2からは、Javaコード上にJavascriptInterfaceアノテーションのないメソッドにはJavaScriptからアクセスできなくなったため、リフレクション[^注釈1]により任意のコードを実行されるリスクがなくなります。

サポートする最小バージョンの変更は、Android Studioで以下のように行います。

```
1. 指定するバージョンがインストール済ならこの作業は不要  
   Tools -> Android -> Android SDK Default Setting 画面でSDKインストール
2. File -> Project Structure… 画面から、アプリのFlavorsタグのMin Sdk Versionを変更
```

### ロードするリソースを信頼できるものに限定する

不正なJavaScriptを含むおそれのない信頼できるリソースのみを読み込むようにしてください。読み込むリソースを自社の管理の及ぶ範囲や、Google社など信頼できる第三者の提供するもののみにすることで、攻撃されるリスクを抑えられます。

```java
    // JavaScript実行用クラス
    class CallFromJavaScript {
        @JavascriptInterface
        public methodA() {...}

        // API 17以降では@JavascriptInterfaceのないメソッドはJavaScriptから実行できない
        public methodB() {...}
    }

    WebView webView = new WebView(context);
    webView.getSettings().setJavaScriptEnabled(true); // WebViewでのJavaScript実行を有効化

    // JavaScriptから呼び出し可能にするJavaオブジェクトを設定する
    webView.addJavascriptInterface(new CallFromJavaScript(), "callMe");

    // 不用意なリソースをロードしない
    webView.load("trustworthy.example.com");
```

Android 4.1(API 17)以下のバージョンをサポートする場合は、ロードするリソースの検証が行えないためにLintは警告を発し続けます。  

## 不適切な例

### 信頼できないリソースのJavascriptを有効にする

検索結果や広告に紛れている悪意のあるサイトをロードしてしまうケースは少なくありません。
JavaScriptが有効になっている状態で悪意のあるサイトをロードすると、任意のコードを実行されてしまうリスクがあります。

```java
    class CallFromJavaScript {
        @JavascriptInterface
        public methodA() {...}

        public methodB() {...}
    }

    WebView webView = new WebView(context);
    webView.getSettings().setJavaScriptEnabled(true); // JavaScriptを有効化する

    // JavaScriptから呼び出し可能にするJavaオブジェクトを設定する
    webView.addJavascriptInterface(new CallFromJavaScript();, "callMe");

    // 信頼できないリソースにアクセスする
    webView.loadUrl(untrustUrl);
```

LintはminSdkVersionが17未満でaddJavascriptInterface()メソッドの呼び出しを検知すると、次のようなメッセージを出力します。

- Lint結果(Warning)  
“\`WebView.addJavascriptInterface\` should not be called with minSdkVersion &lt; 17 for security reasons: JavaScript can use reflection to manipulate application”

注意: Android 4.1(API 16)以下のバージョンでは、開発者がaddJavascriptInterfaceメソッドを使用していない場合でも、フレームワークがデフォルトで呼び出していることがあるため危険です。（[CVE-2013-4710][4]、[CVE-2014-7224][7]）

## 外部リンク

- [WebView | Android Developers][1]
- [WebSettings | Android Developers][2]
- [CVE-2012-6636][3]
- [CVE-2013-4710][4]
- [スマートフォンアプリへのブラウザ機能の実装に潜む危険――WebViewクラスの問題について][5]  
    addJavascriptInterfaceメソッドの使い方によっては脆弱性を作りこんでしまうという指摘(2012年)
- [Androidセキュリティ勉強会～WebViewの脆弱性編～][6]  
    リフレクション[^注釈1]を使用されるためaddJavascriptInterfaceメソッドは使うべきでないという指摘
- [Two New Attack Vectors to Aggravate the Android addJavascriptInterface RCE Issue (CVE-2014-7224)][7]  
    addJavascriptInterfaceメソッドの呼び出しをフレームワークが無条件に行う脆弱性についての記載

[1]:https://developer.android.com/reference/android/webkit/WebView.html
[2]:https://developer.android.com/reference/android/webkit/WebSettings.html
[3]:http://www.cvedetails.com/cve/CVE-2012-6636/
[4]:http://www.cvedetails.com/cve/CVE-2013-4710/
[5]:https://codezine.jp/article/detail/6618
[6]:http://ierae.co.jp/uploads/webview.pdf
[7]:http://daoyuan14.github.io/news/newattackvector.html


[^注釈1]: javascript:void(0); "リフレクション：プログラム実行時に、クラス名やメソッド名を動的に指定して実行(アクセス)するJavaの機能"
