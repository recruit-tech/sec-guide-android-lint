# Code might contain an auth leak (AuthLeak)

## 警告されている問題点

パスワードがハードコードされているため、なりすましや不正アクセスのリスクがあります。

## 対策のポイント

- Basic認証のパスワードをハードコードしない

## 対策の具体例

Basic認証を行う場合、初回はユーザーにパスワードを入力させてCookieに保存するなどして、パスワードをハードコードしないようにしてください。

```java
    URL url = new URL("http://www.example.com");
    URLConnection conn = url.openConnection();

    String password = "";
    for(String cookie : getCookies("www.example.com").split(";")) {
        // cookieに保存済みのパスワードがあれば取得する
        if(isPassword(cookie)) {
            password = parsePassword(cookie);
            break;
        }
    }

    // パスワードをcookieから取得できなかった場合の処理
    （省略）

    conn.setRequestProperty("Authorization", "Basic " + 
            Base64.encodeToString((username + ":" + password).getBytes(), Base64.NO_WRAP));
    InputStream is = conn.getInputStream();
```

上のような対策を実施したとしても、重要な処理の認証にBasic認証のみを利用することは避けるのが賢明です。

## 不適切な例

ソースコード中にパスワード付きBasic認証用URLを記載している場合、APKファイルを解析されるとパスワードは簡単に漏洩します。
ソースコード以外の場所へ記載しても、リスクはさほど小さくなりません。

```java
    String badurl = "http://user1:password1@www.example.com";
```

Lintは、上の例のように「プロトコル://ユーザ名:パスワード@接続先」形式の文字列を検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "Possible credential leak"

ただし、リソースファイルなどソースコード以外の場所への記載は検出できません。

