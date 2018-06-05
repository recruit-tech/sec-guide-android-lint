# Weak RNG (TrulyRandom)

## 警告されている問題点

脆弱性のある標準API[^注釈2]を使用して疑似乱数の生成や暗号化を行っているため、乱数の生成が予測される可能性があります。

## 対策のポイント

- アプリがサポートするAndroidの最小バージョン(minSdkVersion)を上げる

## 対策の具体例

Android 4.4以降の端末では、脆弱性が修正されているので、Android 4.3以下のバージョンをサポートすべき合理的な理由がない場合には、minSdkVersionを19 (Android 4.4)以上にすることを積極的に行ってください。

minSdkVersionの変更は、Android Studioで以下のように行います。

```
1. 指定するバージョンがインストール済ならこの作業は不要  
   Tools -> Android -> Android SDK Default Setting 画面でSDKインストール
2. File -> Project Structure… 画面から、アプリのFlavorsタグのMin Sdk Versionを変更
```

アプリの仕様や設計上の理由でどうしてもminSdkVersionを上げられない場合は、アプリで対応コードを実装してください。
この場合、SHA1PRNGを差し替え、予測困難なビット列でシードを再設定する必要がありますが、具体的な方法については [Android Developers Blog - Some SecureRandom Thoughts][5]を参照してください。

## 不適切な例

Android 4.3以下のバージョンで、SecureRandomの他、内部的に擬似乱数を生成する暗号化、鍵生成、鍵ペア生成、鍵共有 (鍵合意)、SSLEngineクラスなどの暗号関連クラスを使用すると暗号学的に安全となりません。
具体的には下記のメソッド呼び出しあるいはクラスのインスタンス化がLintの検出対象となります。

- SecureRandom
- Cipher.init(mode, key) (ただし、modeがENCRYPT\_MODE, WRAP\_MODEのいずれかの場合のみ)
- Signature.initSign(...)
- KeyGenerator.getInstance(algorithm)
- KeyPairGenerator.getInstance(algorithm)
- KeyAgreement.getInstance(algorithm)
- SSLEngine.wrap, SSLEngine.unwrap

注意: 本来、Signatureクラスも検出対象ですが、Lintが検出するべきクラスのパッケージを誤っているため検出されません。  
(誤) javax.crypto.Signature  
(正) java.security.Signature

以下は、Android 4.3 (API 18)以下をサポートし、Cipherインスタンスで暗号化しています。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest ...>
  <uses-sdk android:minSdkVersion="18" android:targetSdkVersion="25" />
  <application ...>
    ...
  </application>
</manifest>
```

```java
    KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
    keyStore.load(null);

    Cipher aes = Cipher.getInstance("AES/CBC/PKCS5Padding");
    final Key key = keyStore.getKey("secret", null);

    aes.init(Cipher.ENCRYPT_MODE, key);
```

Lintは上の例のようにアプリがAndroid 4.3(API 18)以下をサポートした状態で暗号機能の使用を検知すると、次のようなメッセージを出力します。

- Lint結果(Warning)   
   "Potentially insecure random numbers on Android 4.3 and older. Read https://android-developers.blogspot.com/2013/08/some-securerandom-thoughts.html for more info."

## 外部リンク

-  [CVE-2013-7372][1]
-  [CVE-2013-7373][2]
-  [Kai Michaelis, Christopher Meyer, and Jörg Schwenk. 2013. Randomly Failed! The State of Randomness in Current Java Implementations. Topics in Cryptology – CT-RSA 2013 Lecture Notes in Computer Science (February 2013), 129–144. DOI][3]
-  [Martin Boßlet. 2013. OpenSSL PRNG Is Not (Really) Fork-safe. (August 2013). Retrieved November 20, 2017][4]  
    脆弱性の詳細な情報はKai Michaelisらの論文およびMartin BoßletのWebページを参照してください。
-  [Android Developers Blog - Some SecureRandom Thoughts][5]  
    Android Developers Blogで公開されているワークアラウンド


[1]:http://www.cvedetails.com/cve/CVE-2013-7372/
[2]:http://www.cvedetails.com/cve/CVE-2013-7373/
[3]:http://www.nds.rub.de/media/nds/veroeffentlichungen/2013/03/25/paper_2.pdf
[4]:http://emboss.github.io/blog/2013/08/21/openssl-prng-is-not-really-fork-safe/
[5]:https://android-developers.googleblog.com/2013/08/some-securerandom-thoughts.html


[^注釈2]: javascript:void(0); "SHA1PRNG：CVE-2013-7372およびCVE-2013-7373に関連する問題"
[^注釈1]: javascript:void(0); "脆弱性のある標準API：Androidがデフォルトで採用する疑似乱数生成の実装"
