# Incorrect constant (WrongConstant)

## 警告されている問題点

メソッドの戻り値や、パラメータなどに想定外の値が入力され、アプリを正しく実行できない可能性があります。

## 対策のポイント

-   コードとアノテーションが競合する潜在的な問題に対処する

## 対策の具体例

Support Annotationsライブラリを使用すると、見落とされやすいnullポインター例外やリソースタイプの不一致などの問題を検出できるようになります。利用方法などの詳細はAndroid Studioの[アノテーションによるコード検査の改善のページ][0]を参照してください。  
ここでは、Typedefアノテーションの用例を紹介します。Typedefアノテーションは、パラメータ、戻り値、フィールド参照が特定の定数セット、または範囲内の値をとるように宣言するための仕組みです。  

```java
@Retention(RetentionPolicy.SOURCE)
@IntDef({DOC_WORD, DOC_EXCEL, DOC_PDF})
public @interface DocType {}

public static final int DOC_WORD  = 0;
public static final int DOC_EXCEL = 1;
public static final int DOC_PDF   = 2;

public void openFile(String filename, @DocType int type) { ... }
```

上のようにするとopenFileメソッドの引数typeには、アノテーションで宣言した定数DOC_WORD, DOC_EXCEL, DOC_PDFのみが使用できます。

フラグ（|、&、^ など）属性をつけて、使用できる定数を統合することもできます。

```java
@Retention(RetentionPolicy.SOURCE)
@IntDef(flag=true, value={FONT_BOLD, FONT_ITALIC, FONT_UNDERLINE})
public @interface TextStyle {}

public static final int FONT_BOLD      = 0x01;
public static final int FONT_ITALIC    = 0x02;
public static final int FONT_UNDERLINE = 0x04;

public void setTextStyle(@TextStyle int style) { ... }

setTextStyle(FONT_BOLD|FONT_ITALIC);
```

[@IntRange][2] と組み合わせて、整数が指定された定数セットまたは範囲内の値をとるようにすることもできます。

## 不適切な例

「対策の具体例」で示したopenFileメソッドのtype引数に、アノテーションで宣言していない定数を使用することはコードの信頼性を損ないます。
アノテーションが競合するので警告を発生しますが、この警告が出てもアプリのコンパイルは可能です。

```java
openFile("sample.doc", 0); // 宣言した定数の値であっても不適切

int type = getDocType(); // typeにもgetDocTypeメソッドの戻り値にもアノテーションがあれば不適切とはならない
openFile("sample.doc", type);
```

Lintは上のようにアノテーションで宣言した定数以外の使用を検知すると、次のようなメッセージを出力します。

-   Lint結果(Error)  
    "Must be one of: （アノテーションで宣言した定数の列挙）"

また、DocTypeの例はflag属性を使用したアノテーション定義ではないため、フラグ（|、&、^ など）で引数のパターンを指定することができません。

```java
openFile("sample.doc", DOC_WORD|DOC_PDF);
```

Lintは上のようにflag属性を使用していないアノテーションでフラグの使用を検知すると、次のようなメッセージを出力します。

-   Lint結果(Error)  
    "Flag not allowed here"

## 外部リンク

-   [Improve Code Inspection with Annotations | Android Studio #Typedef annotations][1]


[0]: https://developer.android.com/studio/write/annotations.html
[1]: https://developer.android.com/studio/write/annotations.html#enum-annotations
[2]: https://developer.android.com/reference/android/support/annotation/IntRange.html