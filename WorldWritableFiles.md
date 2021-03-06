# openFileOutput() or similar call passing MODE\_WORLD\_WRITEABLE (WorldWritableFiles)

## 警告されている問題点

ファイルを任意のアプリから書き込める設定にしているため、重要な情報が改竄されたり、破壊されるリスクがあります。

## 対策のポイント

- アプリ利用のファイルはアプリディレクトリ[^注釈1]内に非公開で作成する

## 対策の具体例

非公開でファイルを作成する方法は[File.setReadable() used to make file world-readable][4]に同じなのでそちらを参照してください。

## 不適切な例

以下の例では他のアプリから書き込み可能なモードでファイルを作成/オープンしているため、情報の確実性を保証することができません。

```java
    try {
        // ローカルファイルを他のアプリから書き込み可能モードで作成/オープン
        FileOutputStream myfile = openFileOutput(FILE_NAME, MODE_WORLD_WRITEABLE);

        // ローカル設定ファイルを他のアプリから書き込み可能モードで作成/オープン
        SharedPreferences mypref = getSharedPreferences(PREFS_NAME, MODE_WORLD_WRITEABLE);

        // ローカルディレクトリを他のアプリから書き込み可能モードで作成/オープン
        File mydir = getDir(DIR_NAME, MODE_WORLD_WRITEABLE);

        // 同様のメソッドは他にopenOrCreateDatabaseがあります
    } catch (IOException e) {
        （省略）
    }
```


Lintは、上の例のようにopenFileOutput()、getSharedPreferences()およびgetDir()がWORLD_WRITEABLEを第2引数として呼び出されていることを検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "Using 'MODE_WORLD_WRITEABLE’ when creating files can be risky, review carefully."

注意: 以前はMODE_WORLD_READABLE / MODE_WORLD_WRITEABLEが使用できましたが、Android 4.2(API 17)で非推奨となり、Android 7.0(API 24)以降の端末では、これらのモード指定では機能しなくなりました。

## 外部リンク

- [Context | Android Developers][1]
- [Androidアプリのセキュア設計・セキュアコーディングガイド　4.6.ファイルを扱う][2]  
- [Sharing Files | Android Developers][3]

[1]:https://developer.android.com/reference/android/content/Context.html
[2]:http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%92%E6%89%B1%E3%81%86
[3]:https://developer.android.com/training/secure-file-sharing/index.html
[4]:SetWorldReadable.md



[^注釈1]: javascript:void(0); "アプリディレクトリ：アプリがインストールされた端末内のストレージ。通常、/data/data/&lt;package-name&gt;を指す。"
