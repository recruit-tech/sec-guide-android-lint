# Missing @JavascriptInterface on methods (JavascriptInterface)

## 警告されている問題点

JavaScriptからアクセスするためのルールが守られていないため、スクリプト処理が実行できません。

## 対策のポイント

- JavaScriptから呼び出すメソッドには、@JavascriptInterfaceアノテーションを追加する

## 対策の具体例

JavaのクラスのメソッドをWebView内のJavaScriptから使用したい場合、当該メソッドに@JavascriptInterfaceアノテーションを付記し、WebViewクラスのaddJavascriptInterfaceメソッドで当該クラスのインスタンスを加える必要があります。
セキュリティが強化されたAndroid 4.2(API 17)以降では、@JavascriptInterfaceアノテーションのないメソッドのJavaScriptからの使用はできません。

```java
    // JavaScript実行用クラス
    public class JavaScriptAPI {
        @JavascriptInterface
        public void methodA(){...}

        // API 17以降では@JavascriptInterfaceのないメソッドはJavaScriptから実行できない
        public void methodB(){...}
    }
```

```java
    // Activity内で、WebViewインスタンスを生成
    WebView webView = new WebView(context);
    webView.getSettings().setJavaScriptEnabled(true);  // WebViewでのJavaScript実行を有効化

    webView.addJavascriptInterface(new JavaScriptAPI(),"XXX");  // "XXX"はJavaScriptから参照するオブジェクト名

    wv.loadUrl("file:///android_asset/test.html");
```

```
    // test.htmlからの使用例
    <button onClick="XXX.methodA()">MethodAの実行</button>
```

## 不適切な例

WebView#addJavascriptInterfaceJavaScript()メソッドに登録するオブジェクト(クラス)に1つも@JavascriptInterfaceアノテーションを付けたメソッドがない場合、メソッドを使用するとエラーとなり実行できません。

```java
    // @JavascriptInterfaceアノテーションを持たないJavaScript実行用クラス
    public class JavaScriptBadAPI {
        public void methodA(){...}

        public void methodB(){...}
    }
```

```java
    // Activity内で、WebViewインスタンスを生成
    WebView webView = new WebView(context);
    webView.getSettings().setJavaScriptEnabled(true);

    // @JavascriptInterfaceアノテーションを持たないインスタンスを設定
    webView.addJavascriptInterface(new JavaScriptBadAPI(),"XXX");

    wv.loadUrl("file:///android_asset/test.html");
```

LintはminSdkVersionが17以上でaddJavascriptInterface()メソッドの対象が@JavascriptInterfaceアノテーションを一つも持っていないことを検知すると、次のようなメッセージを出力します。

-   Lint 結果(Error)  

    "None of the methods in the added interface(＜クラス名＞) have been annotated with '@android.webkit.JavascriptInterface'; they will not be visible in API 17"

注意: targetSdkVersionを17未満に設定するとアノテーションがないメソッドも使用可能になりますが、よりリスクの高い状況になります。詳しくは[「addJavascriptInterfaceCalled (AddJavascriptInterface)」][AddJavascriptInterface]の項目を参照してください。

## 外部リンク

-   [Android 4.2 APIs | Android Developers][1]
-   [Building Web Apps in WebView | Android Developers][2]
-   [WebView | Android Developers][3]
-   [Androidアプリのセキュア設計・セキュアコーディングガイド　4.9.3.1. Android4.2未満の端末におけるaddJavascriptInterface()に起因する脆弱性について][4]  
    

[AddJavascriptInterface]: AddJavascriptInterface.md

[1]: https://developer.android.com/about/versions/android-4.2.html
[2]: https://developer.android.com/guide/webapps/webview.html\#BindingJavaScript
[3]: https://developer.android.com/reference/android/webkit/WebView.html\#addJavascriptInterface
[4]: http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#android-4_x2e2%E6%9C%AA%E6%BA%80%E3%81%AE%E7%AB%AF%E6%9C%AB%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8Baddjavascriptinterface_x28_x29%E3%81%AB%E8%B5%B7%E5%9B%A0%E3%81%99%E3%82%8B%E8%84%86%E5%BC%B1%E6%80%A7%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6