# Potential Multiple Certificate Exploit (PackageManagerGetSignatures)

## 警告されている問題点

署名の一致チェックの方法を間違えた場合、誤って悪意あるアプリを正規の相手だと認識してしまい、重要な情報や機能に不正にアクセスされてしまうリスクがあります。

> 以下のようなアプリだけが、この警告の対象になります。その他のアプリは無視して構いません。
> - 他のアプリの開発元確認をアプリケーションパッケージ(APK)の署名に使った証明書をベースに行っている
>
> また、Lintは上記開発元確認を行う上で必須のメソッド呼び出し(のみ)を見て警告を表示しているため、(メソッドの呼び出しを止めない限り)下記の対策を行っても警告が消えることはないので注意してください。

## 対策のポイント

以下のうちで、条件に合う対策を実施してください。
- APKへの署名が一つ、すなわち署名する証明書(もしくは証明書チェーン)が一つの場合は、最初の証明書をチェックする
- Android 4.4(API 19)以前の端末も動作対象になる場合(minSdkVersionが19以下)は、アプリケーションの開発元を検証する際に、対象アプリのAPKが持つすべての証明書をチェックする 
- 上記以外の場合は、対象アプリのAPKが持つ(複数の)証明書に想定した証明書が最低一つ含まれることをチェックすれば十分です

> APKの署名に使用する証明書は単独ではなく証明書チェーンを構成したものも可能になっています。また、APKへの署名は、複数の証明書(チェーン)によって多重に行うことが出来ます。そのためAPKには複数の証明書が含まれ、どのようにチェックするかが安全な実装のポイントになります。

## 対策の具体例

Androidアプリにおけるアプリの開発元確認は、そのアプリのAPKを署名するのに使用した証明書が、予め想定してたものと一致するかどうかを確かめることで行うことが出来ます。
通常は、複数含まれる証明書の中で一つでも想定していたものが含まれれば、開発元の検証は成功したことになります。
しかし、Android 4.4(API 18)以前の端末では、インストール時の証明書チェーンのチェック方法が脆弱(FakeID)なため、不正な証明書チェーンを持つアプリをインストールすることが可能であり、一つの証明書が正しくても正規のAPKである保証がありません。そのため、状況によって適宜下記のいずれかの対策を実施することが推奨されます。

> Lintによる警告がAndroidのバージョンの区別なく表示されるのは、証明書の扱いが上記のような脆弱性を生みやすいことに対する注意喚起の意図があると考えられます。

### アプリ(APK)に含まれる最初の証明書を確認する

アプリを署名する証明書チェーンはインストール時の検証で正規化されるため、1セットの証明書チェーンでの署名を前提とすれば、署名に使用した証明書がPackageInfoのsignature配列の最初の要素として格納されていることが期待できます。そのため、検査対象のアプリが1セットの証明書チェーンでの署名を前提としている場合は、対象アプリのAPKに含まれる最初の証明書の一致を確認すれば十分です。

> 複数の証明書が含まれている場合のPackageInfoのsignature配列に格納されている証明書の順番は明確に仕様化されていませんが、初期の頃から動作は変わってないようです。Android 4.4(API 19)以前の端末での動作を想定したアプリを開発する人で、不安を感じる場合は、次に紹介するすべての証明書をチェックする方法を採用してください。Android 5.0(API 21)以降のみ対象のアプリは、PackageInfoのsignature配列には証明書チェーンの内の署名に使用した証明書のみ格納されるため、この方法で特に問題は起きないはずです。

```java
package com.example.checksignature;

import android.app.Activity;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.content.pm.Signature;
import android.os.Bundle;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;


public class myActivity extends Activity {
    private Signature[] sigs = null;
    // このホワイトリストに登録されている証明書のメッセージダイジェストを持つアプリのみを許可する
    private static final String[] sigWhiteList = {
            "3A544EC58814727C3C424DFD2ECF2F8C7FBE783E11C020649AAE969594485019",
            "9A291DF4BC470525B824240ED9EF81C6D6FDEEA3052EFD9FC5A5D7014D98E869"};

    private boolean getSignatures(String packagename) {
        PackageInfo pinfo = null;
        try {
            pinfo = getPackageManager().getPackageInfo(packagename, PackageManager.GET_SIGNATURES);
        } catch (PackageManager.NameNotFoundException e) {
            android.util.Log.e("myActivity", "Could not get package info.");
            return false;
        }
        sigs = pinfo.signatures;
        return true;
    }
    private String byte2hexStr(byte[] dat) {
        if (dat == null) return null;
        final StringBuilder hexdec = new StringBuilder();
        for (final byte b:dat) {
            hexdec.append(String.format("%02X", b));
        }
        return hexdec.toString();
    }
    private boolean checkSignatures(String packagename) {
        if (!getSignatures(packagename)) return false;

        // 最初の証明書のみを検査する
        Signature sig = sigs[0];
        // 証明書のメッセージダイジェストを計算し16進文字列に変換
        byte[] sha256;
        try {
            sha256 = MessageDigest.getInstance("SHA-256").digest(sig.toByteArray());
        } catch (NoSuchAlgorithmException e) {
            return false;
        }
        String sigDigest = byte2hexStr(sha256);
        // ホワイトリストに登録されているかを確認する
        for (String wsig : sigWhiteList) {
            // ホワイトリストに登録されていれば，OK
            if (wsig.equals(sigDigest)) return true;
        }
        // ホワイトリストに登録されていない
        return false;
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        String caller = getCallingActivity().getPackageName();
        // 自分を起動したアプリの証明書を検査する
        if (!checkSignatures(caller)) {
            // ホワイトリストに登録されていない証明書を持つアプリだった
            android.util.Log.e("getSig", "Caller is not my partner: " + caller);
            finish();
            return;
        }
        // 以下利用元の確認が出来た時の処理を行う
        // ....
    }
}
```

### アプリ(APK)に含まれるすべての証明書を確認する

Android 4.4(API 19)以前の端末での動作を想定したアプリの場合は、以下の例を参考に実装を行ってください。

```
package com.example.checksignature;

import android.app.Activity;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.content.pm.Signature;
import android.os.Bundle;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;


public class myActivity extends Activity {
    private Signature[] sigs = null;
    // このホワイトリストに登録されている証明書のメッセージダイジェストを持つアプリのみを許可する
    private static final String[] sigWhiteList = {
            "3A544EC58814727C3C424DFD2ECF2F8C7FBE783E11C020649AAE969594485019",
            "9A291DF4BC470525B824240ED9EF81C6D6FDEEA3052EFD9FC5A5D7014D98E869"};

    private boolean getSignatures(String packagename) {
        PackageInfo pinfo = null;
        try {
            pinfo = getPackageManager().getPackageInfo(packagename, PackageManager.GET_SIGNATURES);
        } catch (PackageManager.NameNotFoundException e) {
            android.util.Log.e("myActivity", "Could not get package info.");
            return false;
        }
        sigs = pinfo.signatures;
        return true;
    }
    private String byte2hexStr(byte[] dat) {
        if (dat == null) return null;
        final StringBuilder hexdec = new StringBuilder();
        for (final byte b:dat) {
            hexdec.append(String.format("%02X", b));
        }
        return hexdec.toString();
    }
    private boolean checkSignatures(String packagename) {
        if (!getSignatures(packagename)) return false;

        // パッケージを署名した全ての証明書について検査する
        for (Signature sig : sigs) {
            // 証明書のメッセージダイジェストを計算し16進文字列に変換
            byte[] sha256;
            try {
                sha256 = MessageDigest.getInstance("SHA-256").digest(sig.toByteArray());
            } catch (NoSuchAlgorithmException e) {
                return false;
            }
            String sigDigest = byte2hexStr(sha256);
            // ホワイトリストに登録されているかを確認する
            for (String wsig : sigWhiteList) {
                // ホワイトリストに登録されていれば，次の証明書を検査する
                if (wsig.equals(sigDigest)) continue;
            }
            // ホワイトリストに登録されていない
            return false;
        }
        // 全ての証明書がホワイトリストに登録されている
        return true;
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        String caller = getCallingActivity().getPackageName();
        // 自分を起動したアプリの証明書を検査する
        if (!checkSignatures(caller)) {
            // ホワイトリストに登録されていない証明書を持つアプリだった
            android.util.Log.e("getSig", "Caller is not my partner: " + caller);
            finish();
            return;
        }
        // 以下利用元の確認が出来た時の処理を行う
        // ....
    }
}
```

## 不適切な例

### 想定した証明書が最低一つ含まれることを確認する(Lint検出：対象)

直接Lintで警告される部分ではありませんが、Android 4.4以前の端末で以下の例ような検査をしている高い実行権限を持つアプリが存在していたため、正規プラグインに成りすまして高い権限で動作するマルウェアを作成することが可能でした。  

> Android 5.0(API 21)以降の端末のみが対象のアプリであれば、以下の実装でも十分に安全に動作します。

```java
    private boolean checkSignatures(String packagename) {
        if (!getSignatures(packagename)) return false;

        // パッケージを署名した全ての証明書について検査する
        for (Signature sig : sigs) {
            // 証明書のメッセージダイジェストを計算し16進文字列に変換
            byte[] sha256;
            try {
                sha256 = MessageDigest.getInstance("SHA-256").digest(sig.toByteArray());
            } catch (NoSuchAlgorithmException e) {
                return false;
            }
            String sigDigest = byte2hexStr(sha256);
            // ホワイトリストに登録されているかを確認する
            for (String wsig : sigWhiteList) {
                // 一つでもホワイトリストに登録されていれば，オーケーとする
                if (wsig.equals(sigDigest)) return true;
            }
        }
        // 全ての証明書がホワイトリストに登録されていない
        return false;
    }
```


Lintの検出は、検査ロジックではなく、Signature(証明書)付きのパッケージ情報を取得するメソッドに対して行われます。

> そのため、検査ロジックが正当性に関係なくLintの警告が出力されるので注意が必要です。
下のようにandroid.content.pm.PackageManagerのgetPackageInfo()メソッドの第2引数にPackageManager.GET_SIGNATURESが含まれているのを発見するとメッセージを出力します。

```java
private Signature[] sigs;
private ActivitiInfo[] acts;
private void getSignaturesAndActivities(String packagename) {
  PackageInfo pinfo = null;
  try {
        pinfo = getPackageManager().getPackageInfo(packagename, PackageManager.GET_SIGNATURES | PackageManager.GET_ACTIVITIES);
  } catch (PackageManager.NameNotFoundException e) {
        android.util.Log.e("myActivity", "Could not get package info.");
        return null;
  }
  sigs = pinfo.signatures;
  acts = pinfo.activities;
}
```

* Lint出力(Information)  
  "Reading app signatures from getPackageInfo: The app signatures could be exploited if not validated properly; see issue explanation for details."

## 外部リンク

- [PackageManager | Android Developers][1]
- [PackageInfo | Android Developers][2]
- [Signature | Android Developers][3]
- [MessageDigest | Android Developers][4]
- [FakeID に関する情報1][5]
- [FakeID に関する情報2][6]
- [Black Hat 2014 における FakeID についての発表スライド][7]
- [Android アプリのセキュア設計・セキュアコーディングガイド][8]  
  「4.1.1.3. パートナー限定Activity を作る・利用する」

[1]:https://developer.android.com/reference/android/content/pm/PackageManager.html
[2]:https://developer.android.com/reference/android/content/pm/PackageInfo.html
[3]:https://developer.android.com/reference/android/content/pm/Signature.html
[4]:https://developer.android.com/reference/java/security/MessageDigest.html
[5]:http://blog.trendmicro.co.jp/archives/9642
[6]:https://www.androidcentral.com/fake-id-and-android-security-updated
[7]:https://www.blackhat.com/docs/us-14/materials/us-14-Forristal-Android-FakeID-Vulnerability-Walkthrough.pdf
[8]:http://www.jssec.org/dl/android_securecoding.pdf


## 脚注
