# Code might contain an auth leak (AuthLeak)

## 警告されている問題点

パスワードがハードコードされているため、なりすましや不正アクセスにつながるリスクがあります。

## 対策のポイント

- Basic認証のパスワードをハードコードしないでください

## 対策の具体例

### パスワードをハードコードしない

Basic認証を行う場合、初回はユーザーにパスワードを入力させるなど、パスワードをハードコードしないようにしてください。

```java
    URL url = new URL("http://www.example.com");
    URLConnection conn = url.openConnection();

    String password = "";
    for(String cookie : getCookies("www.example.com").split(";")) {
        // cookieに保存済みのパスワードがあれば取得する
        if(isPassword(cookie)) password = parsePassword(cookie);
    }

    String encode = Base64.encodeToString(("username:" + password).getBytes(), Base64.NO_WRAP);
    conn.setRequestProperty("Authorization", "Basic " + encode);
    InputStream is = conn.getInputStream();
```

このような対策を実施したとしても、重要な処理の認証にBasic認証のみを利用することは避けるのが賢明です。

## 不適切な例

### パスワードをハードコードする

Lintは、Javaソースコード中のBasic認証用URLのパスワードを検出してメッセージを出力しますが、リソースやファイルなどソースコード以外の場所への記載、URL形式のパスワード以外の秘密情報は検出できません。
このことを利用してメッセージの出力を回避してパスワードをハードコードしても、APKファイルを解析されればハードコードされた情報は簡単に漏洩します。

```java
    String badurl = "http://user1:password1@www.example.com";
```

Lintは上の例のように「プロトコル://ユーザ名:パスワード@接続先」形式の文字列を検知すると、パスワードがハードコードされているとして次のようなメッセージを出力します。

-   Lint結果(Warning)  
    "Possible credential leak"

## 外部リンク

