# Code contains easter egg (EasterEgg)

## 警告されている問題点

ソースコード内にコメントに見せかけたプログラムが埋め込まれており、悪意のあるコードが実行されてしまう可能性があります。

```
このチェックはデフォルトで無効ですが、次の手順で有効にすることができます。

1. 「Settings」画面の表示
・Android Studio全体での設定の場合：  
　起動画面（「Welcome to Android Studio」画面）の右下「Configure」から「Settings」画面を開く  
・プロジェクト単位での設定の場合：  
　プロジェクト画面の「File」メニューから「Settings...」画面を開く
2. EasterEggチェックの有効化  
    1. 左ペインでEditor -> Inspectionsを選択する  
    2. 右ペインでAndroid -> Lint -> Securityの一覧から"Code contains easter egg"のチェックをオンにする
```

## 対策のポイント

-   不適切なコメント文(実際は実行可能なコード)を削除する

## 対策の具体例

ソースコードの検出された箇所を確認して、不適切なコメント文(実行可能コード)であればすべて削除してください。  
ペア・コーディングやソースコード・レビューでは、これを見逃さないようにしてください。

## 不適切な例

隠しコードを意図的に混入したり、放置したままリリースした場合、アプリの利用者が被害を受ける可能性があります。

以下の例では、"\u002a\u002f"（ASCIIコードでは"\*/")によりコメントを閉じ、実行可能なコードを続けて挿入し、行末に"\u002f\u002a"(ASCIIコードでは"/\*")を置くことで新しいコメントを開き、ブロック全体がコメントアウトされているように見せることで、実行可能なコードを埋め込んでいます。

```java
    // ...
    public class MainActivity extends AppCompatActivity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            /*\u002a\u002f\u0042\u0075\u0074\u0074\u006f\u006e\u0020\u0074\u0061\u0070\u0048\u0065\u0072\u0065\u0020\u003d\u0020\u0028\u0042\u0075\u0074\u0074\u006f\u006e\u0029\u0066\u0069\u006e\u0064\u0056\u0069\u0065\u0077\u0042\u0079\u0049\u0064\u0028\u0052\u002e\u0069\u0064\u002e\u0074\u0061\u0070\u0048\u0065\u0072\u0065\u0029\u003b\u002f\u002a
            /*\u002a\u002f\u0074\u0061\u0070\u0048\u0065\u0072\u0065\u002e\u0073\u0065\u0074\u004f\u006e\u0043\u006c\u0069\u0063\u006b\u004c\u0069\u0073\u0074\u0065\u006e\u0065\u0072\u0028\u006e\u0065\u0077\u0020\u0056\u0069\u0065\u0077\u002e\u004f\u006e\u0043\u006c\u0069\u0063\u006b\u004c\u0069\u0073\u0074\u0065\u006e\u0065\u0072\u0028\u0029\u0020\u007b\u002f\u002a
            /*\u002a\u002f\u0040\u004f\u0076\u0065\u0072\u0072\u0069\u0064\u0065\u002f\u002a
            /*\u002a\u002f\u0070\u0075\u0062\u006c\u0069\u0063\u0020\u0076\u006f\u0069\u0064\u0020\u006f\u006e\u0043\u006c\u0069\u0063\u006b\u0028\u0056\u0069\u0065\u0077\u0020\u0076\u0029\u0020\u007b\u002f\u002a
            /*\u002a\u002f\u0054\u0065\u0078\u0074\u0056\u0069\u0065\u0077\u0020\u0074\u0065\u0078\u0074\u0056\u0069\u0065\u0077\u0020\u003d\u0020\u0028\u0054\u0065\u0078\u0074\u0056\u0069\u0065\u0077\u0029\u0066\u0069\u006e\u0064\u0056\u0069\u0065\u0077\u0042\u0079\u0049\u0064\u0028\u0052\u002e\u0069\u0064\u002e\u0074\u0065\u0078\u0074\u0056\u0069\u0065\u0077\u0029\u003b\u002f\u002a
            /*\u002a\u002f\u0074\u0065\u0078\u0074\u0056\u0069\u0065\u0077\u002e\u0073\u0065\u0074\u0054\u0065\u0078\u0074\u0028\u0022\u0057\u0065\u006c\u0063\u006f\u006d\u0065\u0020\u0074\u006f\u0020\u0045\u0061\u0073\u0074\u0065\u0072\u0020\u0045\u0067\u0067\u0021\u0022\u0029\u003b\u002f\u002a
            /*\u002a\u002f\u007d\u002f\u002a
            /*\u002a\u002f\u007d\u0029\u003b\u002f\u002a
            /**/
        }
    }
```

上のコメントの実態は以下のコードで、ボタンのタップでメッセージが表示されてしまいます。

```java
            /**/Button tapHere = (Button)findViewById(R.id.tapHere);/*
            /**/tapHere.setOnClickListener(new View.OnClickListener() {/*
            /**/@Override/*
            /**/public void onClick(View v) {/*
            /**/TextView textView = (TextView)findViewById(R.id.textView);/*
            /**/textView.setText("Welcome to Easter Egg!");/*
            /**/}/*
            /**/});/*
            /**/
```

Lintは、ソースのコメント中に"\u002a\u002f"を検知すると、次のようなメッセージを出力します。  

-   Lint出力(Warning)  
    "Code might be hidden here; found unicode escape sequence which is interpreted as comment end, compiled code follows"


[1]: http://www.i-programmer.info/history/computer-languages/2340-coded-easter-eggs.html