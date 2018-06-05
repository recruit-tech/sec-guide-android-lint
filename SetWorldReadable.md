# File.setReadable() used to make file world-readable (SetWorldReadable)

## 警告されている問題点

ファイルを任意のアプリから読める設定にしているため、重要な情報が漏洩するリスクがあります。

# 対策のポイント

- アプリ利用のファイルはアプリディレクトリ[^注釈1]内に非公開で作成する

## 対策の具体例

Context.openFileOutputメソッドでのファイル作成時にモードをMODE_PRIVATEに指定することで、他のアプリに公開しないファイルを作成することが出来ます。原則、作成した後はファイルのアクセス制御を変更しないようにしましょう。
外付けのストレージでは、ファイルのアクセス制御をアプリ側で行うことができないため、重要な情報の保存に使用しないようにしましょう。

```java
    try {
        // ファイルをアプリディレクトリ内に非公開で作成する
        fileout = openFileOutput(MYFILE, MODE_PRIVATE);
    } catch (IOException e) {
        android.util.Log.e("MyApp", "Failed to create file.");
        return;
    }
```

注意: 以前はMODE_WORLD_READABLE / MODE_WORLD_WRITEABLEというモードも使用できましたが、Android 4.2(API 17)で非推奨となり、Android 7.0(API 24)以降の端末では、これらのモード指定では機能しなくなりました。

Note: 特定のアプリに対して情報を共有するには、ContentProvider、BroadcastReceiver、およびServiceを利用しましょう。[Sharing Files | Android Developers][4]も参考にしてください。


## 不適切な例

以下の例では一時的に読み込みを許可するつもりで実装していますが、意図している通りに動作しません。
さらに、読み込みを許可した後、例外が発生するなどして戻せない場合のことを考慮するとこのような仕様は避けるべきです。

```java
public void extendReadPermission(File f) {
    try {
        // 第一引数（読取り許可）をtrue、第二引数（所有者のみ適用）をfalseにすることで、全ての読取りを許可する
        if (!f.setRedable(true, false)) {
            (省略)
        }

        // 第一引数（読取り許可）をtrue、第二引数（所有者のみ適用）をtrueにすることで、所有者のみ読取りを許可する
        if (!f.setRedable(true, true)) {
            // この場合、setRedable(true, true)だけだと所有者以外の読取り許可が残ってしまいます
            // setRedable(false, false)の実行後にsetRedable(true, true)とする必要があります
        }

    } catch (SecurityException e) {
        (省略)
    }
}
```

Lintは上の例のようにsetReadable(true, false)という形のメソッド呼び出しを検知すると、次のようなメッセージを出力します。

- Lint結果(Warning)  
  "Setting file permission to world-readable can be risky, review carefully."

## 外部リンク

- [File | Android Developers][1]
- [Context | Android Developers][2]
- [Androidアプリのセキュア設計・セキュアコーディングガイド　4.6.ファイルを扱う][3]    
- [Sharing Files | Android Developers][4]

[1]:https://developer.android.com/reference/java/io/File.html
[2]:https://developer.android.com/reference/android/content/Context.html
[3]:https://www.jssec.org/dl/android_securecoding.pdf#page=243
[4]:https://developer.android.com/training/secure-file-sharing/index.html


[^注釈1]: javascript:void(0); "アプリディレクトリ：アプリがインストールされた端末内のストレージ。通常、/data/data/&lt;package-name&gt;を指す。"
