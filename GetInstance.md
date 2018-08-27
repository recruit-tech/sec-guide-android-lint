# Cipher.getInstance with ECB (GetInstance)

## 警告されている問題点

安全とされる暗号方式を指定していないため、情報漏洩や改竄のリスクがあります。

## 対策のポイント

-   ブロック暗号のモード、およびパディングは、推奨されるものを明示的に指定する

## 対策の具体例

javax.crypto.CipherクラスのgetInstanceメソッドの引数のモードとしてECBを指定することは避け、以下の例に示すような[安全とされているアルゴリズム/モード/パディングの組み合わせ][2]を使用してください。

```java
    Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
```

```java
    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
```

## 不適切な例

### モード/パディング指定を省略する

getInstanceメソッドのパラメータは、アルゴリズム/モード/パディング、もしくはアルゴリズムのみの指定が可能です。
アルゴリズムのみを指定した場合、Androidは[欠点のあるECB][3]をモードとして選択することがあり、安全ではありません。

```java
    Cipher cipher = Cipher.getInstance("AES");  // アルゴリズムのみ指定
```

Lintは、上の例のようにgetInstanceメソッドにアルゴリズムのみの指定を検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
"\`Cipher.getInstance` should not be called without setting the encryption mode and padding"
ただし、API26以降に追加されたAES_128やAES_256を指定した場合には検知しません。

### ECBモードを指定する

[欠点のあるECB][3]をモードとして指定することは安全ではありません。

```java
    Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");  // モードとしてECBを指定
```

Lintは、上の例のようにgetInstanceメソッドにECBのモード指定を検知すると、次のようなメッセージを出力します。

-   Lin結果(Warning)  
    "ECB encryption mode should not be used"

## 外部リンク

-   [Cipher | Android Developers][1]
-   CRYPTREC[『電子政府における調達のために参照すべき暗号のリスト（CRYPTREC暗号リスト）』][2]
-   IPA[『ブロック暗号を使った秘匿、メッセージ認証、および認証暗号を目的とした利用モードの技術報告書』][3]



[1]: https://developer.android.com/reference/javax/crypto/Cipher.html
[2]: http://www.cryptrec.go.jp/list/cryptrec-ls-0001-2016.pdf
[3]: https://www.ipa.go.jp/security/enc/CRYPTREC/fy15/documents/mode_wg040607.pdf#page=34