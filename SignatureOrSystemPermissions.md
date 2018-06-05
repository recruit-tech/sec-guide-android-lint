# signatureOrSystem permissions declared (SignatureOrSystemPermissions)

## 警告されている問題点

通常のアプリには必要としないレベルのパーミッションが要求されているために、過剰な要求として警告が出力されています。

## 対策のポイント

- パーミッションに適切なレベルを設定してください。

## 対策の具体例

### 保護レベルを適切なレベルに設定する

以下は、ユーザーの個人情報を参照するなどの重要な情報を取り扱う例です。READ_CONTACTSのprotection levelは、dangerousで、パーミッショングループは、PERSONAL_INFOです。
アプリがDangerousパーミッションを要求すると、Android OSはユーザーに対して確認画面を表示し、そのPermissionの利用を許可するかどうかの判断を求めます。
つまり、ユーザーの許可なく個人情報にアクセス出来ないように保護をしています。

```
<manifest ... >
    <application ... >
        <provider
            android:exported="true"
            android:name="com.example.provider.providerClass"
            android:authorities="com.example.provider.providerData"
            android:permission="android.permission.READ_CONTACTS">
        </provider>
        :
    </application>
    :
</manifest>
```

次は、他社のアプリから利用されては困る機能や情報を取り扱う例です。この場合、自社独自のsignatureレベルのパーミッションを定義して、同じ秘密鍵で署名したアプリのみがアクセスできるようにします。

```
<manifest ...>
    <permission android:name="com.example.permission.SIGNATURE"
                android:label="@string/foo"
                android:description="@string/foo"
                android:protectionLevel="signature"/>
    <application
            android:icon="@drawable/ic_launcher"
            android:label="@string/app_name" >
              :
              :
    </application>
</manifest>
```

## 不適切な例

### パーミッションの保護レベルで"signatureOrSystem"を指定する

下の例ではパーミッションの保護レベルとして"signatureOrSystem"を指定しています。過剰な保護レベルを要求していると判断して、Lintが警告を出力します。

```
<?xml version="1.0" encoding="utf-8"?>

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example"
          android:versionCode="1"
          android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />
    <permission android:name="com.example.permission.SIGNATURE_OR_SYSTEM"
                android:label="@string/foo"
                android:description="@string/foo"
                android:protectionLevel="signatureOrSystem"/>
    <application
            android:icon="@drawable/ic_launcher"
            android:label="@string/app_name" >
    </application>
</manifest>
```

Lintは`android:protectionLevel="signatureOrSystem"`となっている設定を検知すると、次のようなメッセージを出力します

- Lint出力結果(Warning)  
  "protection level should probably not be set to 'signatureOrSystem'."

## 外部リンク

- [Defining Permissions | Android Developers][1]
- [&lt;permission&gt; | Android Developers][2]
- [Android アプリのセキュア設計・セキュアコーディングガイド 5.2. Permission と Protection Level][3]  

[1]:https://developer.android.com/guide/topics/permissions/defining.html
[2]:https://developer.android.com/guide/topics/manifest/permission-element.html
[3]:http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#permission%E3%81%A8protection-level


