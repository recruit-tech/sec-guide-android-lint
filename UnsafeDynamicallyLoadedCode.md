# load used to dynamically load code (UnsafeDynamicallyLoadedCode)

## 警告されている問題点

安全ではないライブラリ[^注釈1]を使用する可能性があるため、アプリの情報・機能が不正に利用されるリスクがあります。

## 対策のポイント

以下のすべての対策を実施してください。

- アプリが持つ共有ライブラリはアプリのライブラリディレクトリに配置する
- 共有ライブラリのロード元は標準のライブラリディレクトリに限定する

## 対策の具体例

### 共有ライブラリをアプリ標準のライブラリディレクトリに配置する

アプリ(APK)が保持する共有ライブラリ(.so)を使用する場合は、アプリのライブラリディレクトリ(lib)以外に配置することは避けてください。
詳細は[「Native code outside library directory」][3]を参照してください。

### 共有ライブラリのロード元を標準のライブラリディレクトリに限定する

ライブラリのロードには、Runtime.loadLibrary()もしくはSystem.loadLibrary()を使用することで、ロード元をアプリのlibディレクトリを含む標準のライブラリディレクトリ(/system/lib、/vendor/libなど)に限定することができます。
これらのディレクトリは、systemあるいはroot権限が設定されており、一般アプリはファイルの読み込みのみが許可されています。そのため攻撃者による改竄のリスクが低く抑えられます。

以下はloadLibrary()の使用例です。

```java
public class MainActivity extends AppCompatActivity  {
    static {
        // ライブラリディレクトリのlibhello.soを読み込み
        System.loadLibrary("hello"); 
    }

    // 読み込むライブラリにあるメソッドを定義
    public native String sampleMethod();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //ライブラリのメソッド呼び出し
        Log.i("", sampleMethod());
    }
}
```

## 不適切な例

Runtime.load()もしくはSystem.load()は任意のパスのライブラリをロードすることができますが、第三者アプリによる書き換えのリスクが高くなります。

```java
public class MainActivity extends Activity {
    private class Loader {
        public void load() {
            try {
                // アプリ内データ領域からロード
                String mydir = getFilesDir().getPath();
                Runtime.getRuntime().load(mydir + "/libhello1.so");

                // アプリ固有外部ストレージからロード
                String myexdir = getExternalFilesDir(null).getPath();
                System.load(myexdir + "libhello2.so");
            } catch (SecurityException ignore) {
                ...
            } catch (UnsatisfiedLinkError ignore) {
                ...
            } catch (NullPointerException ignore) {
                ...
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Loader ld = new Loader();
        ld.load();
    }
}
```

Lintは、load()の使用を検知すると、次のようなメッセージを出力します。

- Lint出力(Warning)  
  "Dynamically loading code using \`load\` is risky, please use \`loadLibrary\` instead when possible."

## 外部リンク

- [Runtime | Android Developers][1]
- [System | Android Develpers][2]


[1]:https://developer.android.com/reference/java/lang/Runtime.html
[2]:https://developer.android.com/reference/java/lang/System.html
[3]:UnsafeNativeCodeLocation.md


[^注釈1]: javascript:void(0); "ライブラリ：C/C++言語で記述されたネイティブコードをビルドした実行形式のバイナリファイル(標準的なファイルの拡張子は“.so”)"
