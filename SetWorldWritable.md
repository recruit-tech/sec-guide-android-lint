# File.setWritable() used to make file world-writable (SetWorldWritable)

## 警告されている問題点

ファイルを任意のアプリから書き込める設定にしているため、重要な情報が改竄されたり、破壊されるリスクがあります。

## 対策のポイント

- アプリ利用のファイルはアプリディレクトリ[^注釈1]内に非公開で作成する

## 対策の具体例

非公開でファイルを作成する方法は[File.setReadable() used to make file world-readable][5]に同じなのでそちらを参照してください。

## 不適切な例

以下の例ではファイルに対して他のアプリからの書き込みを許可しているため、情報の安全性を保持することができません。

```java
public void setWritePermission(File f) {
    try {
        // 第一引数（書込み許可）をtrue、第二引数（所有者のみ適用）をfalseにすることで、全ての書き込みを許可する
        f.setWritable(true, false);

        (省略)

    } catch (SecurityException e) {
        (省略)
    }
}
```

Lintは上の例のようにsetWritable(true, false)という形のメソッド呼び出しを検知すると、次のようなメッセージを出力します。

-   Lint結果(Warning)  
    "Setting file permission to world-writable can be risky, review carefully."

## 外部リンク

- [File | Android Developers][1]
- [Context | Android Developers][2]
- [Androidアプリのセキュア設計・セキュアコーディングガイド　4.6.ファイルを扱う][3]  
- [Sharing Files | Android Developers][4]

[1]:https://developer.android.com/reference/java/io/File.html
[2]:https://developer.android.com/reference/android/content/Context.html
[3]:http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%92%E6%89%B1%E3%81%86
[4]:https://developer.android.com/training/secure-file-sharing/index.html
[5]:SetWorldReadable.md


[^注釈1]: javascript:void(0); "アプリディレクトリ：アプリがインストールされた端末内のストレージ。通常、/data/data/&lt;package-name&gt;を指す。"
