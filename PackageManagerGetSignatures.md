# Potential Multiple Certificate Exploit (PackageManagerGetSignatures)

## 警告されている問題点

アプリパッケージを署名した証明書（以下、「アプリ証明書」と称す）の検証方法を間違えた場合、不正なアプリに重要な情報や機能を利用されてしまうリスクがあります。

## 対策のポイント

- 対象のAPKが持つすべてのアプリ証明書を確認する 

## 対策の具体例

APKに含まれるアプリ証明書を取得し、利用する際は次のことに気をつける必要があります。
Android 4.4(API 19)以前の端末では、インストール時のアプリ証明書の検証方法に脆弱性がある([FakeID][7])ため、アプリのなりすましが可能です。
同一端末の他アプリからの呼び出しがなりすましされていないかを確認する場合、下の例のようにAPKに含まれる「すべての」アプリ証明書を確認し、疑わしいものが含まれないことを検査する必要があります。

```java
package com.example.checksignature;

import android.app.Activity;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.content.pm.Signature;
import android.os.Bundle;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Arrays;

public class myActivity extends Activity {
    private Signature[] sigs = null;
    // ホワイトリストに登録した証明書のメッセージダイジェストのみを持つアプリを許可する
    // 証明書は開発元から入手する
    private static final String[] sigWhiteList = {
            "3A544EC58814727C3C424DFD2ECF2F8C7FBE783E11C020649AAE969594485019",
            "9A291DF4BC470525B824240ED9EF81C6D6FDEEA3052EFD9FC5A5D7014D98E869"};

    private boolean getSignatures(String pname) {
        PackageInfo pinfo = null;
        try {
            // 引数で使用しているGET_SIGNATURESはAPI28で非推奨となり、
            // 代わりにGET_SIGNING_CERTIFICATESが追加されています
            pinfo = getPackageManager().getPackageInfo(pname, PackageManager.GET_SIGNATURES);
        } catch (PackageManager.NameNotFoundException e) {
            android.util.Log.e("myActivity", "Could not get package info.");
            return false;
        }
        sigs = pinfo.signatures;
        return true;
    }

    private String byte2hex(byte[] dat) {
        if (dat == null) return null;
        final StringBuilder hexdec = new StringBuilder();
        for (final byte b : dat) {
            hexdec.append(String.format("%02X", b));
        }
        return hexdec.toString();
    }

    // パッケージを署名したすべての証明書を検査する
    private boolean checkSignatures(String packagename) {
        if (!getSignatures(packagename)) return false;
        MessageDigest digest = null;
        try {
            digest = MessageDigest.getInstance("SHA-256");
        } catch (NoSuchAlgorithmException e) {
            return false;
        }

        for (Signature sig : sigs) {
            // アプリ証明書をメッセージダイジェスト化して比較
            byte[] sha256 = digest.digest(sig.toByteArray());
            if(!Arrays.asList(sigWhiteList).contains(byte2hex(sha256))) return false;
        }
        return true;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        String caller = getCallingActivity().getPackageName();
        // 呼び出し元のアプリ証明書を検査する
        if (!checkSignatures(caller)) {
            android.util.Log.e("getSig", "Caller is not my partner: " + caller);
            finish();
            return;
        }
        // 以下利用元の確認が出来た時の処理を行う
        ...
    }
}
```

ただし、Lintはアプリ証明書の取得自体に対して警告を発しているため、上記対応を実施しても警告は継続します。

note: 自社開発のアプリの連携であれば、signatureレベルのパーミッションを定義・利用することでアプリの身元確認を代用できます。ただし、第三者アプリに対して広く連携する場合も含めて、受信Intentの安全性は常に確認すべきです。

## 不適切な例

以下の実装は、「対策の具体例」のcheckSignaturesメソッドをホワイトリストの証明書がひとつでも含まれていれば問題なしと判断するように処理しています。
Android 5.0(API 21)以降の端末のみを対象とするのであれば十分ですが、Android 4.4以前の端末では不十分となります。

```java
    private boolean checkSignatures(String packagename) {
        if (!getSignatures(packagename)) return false;
        MessageDigest digest = null;
        try {
            digest = MessageDigest.getInstance("SHA-256");
        } catch (NoSuchAlgorithmException e) {
            return false;
        }

        for (Signature sig : sigs) {
            // アプリ証明書をメッセージダイジェスト化して比較
            byte[] sha256 = digest.digest(sig.toByteArray());
            if(Arrays.asList(sigWhiteList).contains(byte2hex(sha256))) return true;
        }
        return false;
    }
```


Lintは、android.content.pm.PackageManagerのgetPackageInfoメソッドの第2引数にPackageManager.GET_SIGNATURESが含まれていることを検知すると、その戻り値の用途には依らず次のようなメッセージを出力します。

- Lint出力(Information)  
  "Reading app signatures from getPackageInfo: The app signatures could be exploited if not validated properly; see issue explanation for details."

## 外部リンク

- [PackageManager | Android Developers][1]
- [PackageInfo | Android Developers][2]
- [Signature | Android Developers][3]
- [MessageDigest | Android Developers][4]
- [FakeID に関する情報1][5]
- [FakeID に関する情報2][6]
- [Black Hat 2014 における FakeID についての発表スライド][7]
- [Android アプリのセキュア設計・セキュアコーディングガイド 4.1.1.3. パートナー限定Activity を作る・利用する][8]

[1]:https://developer.android.com/reference/android/content/pm/PackageManager.html
[2]:https://developer.android.com/reference/android/content/pm/PackageInfo.html
[3]:https://developer.android.com/reference/android/content/pm/Signature.html
[4]:https://developer.android.com/reference/java/security/MessageDigest.html
[5]:http://blog.trendmicro.co.jp/archives/9642
[6]:https://www.androidcentral.com/fake-id-and-android-security-updated
[7]:https://www.blackhat.com/docs/us-14/materials/us-14-Forristal-Android-FakeID-Vulnerability-Walkthrough.pdf
[8]:http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#%E3%83%91%E3%83%BC%E3%83%88%E3%83%8A%E3%83%BC%E9%99%90%E5%AE%9Aactivity%E3%82%92%E4%BD%9C%E3%82%8B%E3%83%BB%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B

