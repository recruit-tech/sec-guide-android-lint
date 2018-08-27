# Invalid Permission Attribute (InvalidPermission)

## 警告されている問題点

マニフェストでのパーミッション指定が無効なため、期待する保護が得られず、機能の不正利用や情報の漏洩・改竄のリスクがあります。

## 対策のポイント

- 保護すべき機能・情報を確認し、適切なコンポーネントにパーミッションを指定してください。

## 対策の具体例

パーミッションは、Androidが提供するコンポーネント(Activity、ContentProvider、Service、BroadcastReceiver)に対して設定します。

マニフェストでパーミッション(permission属性) の設定が有効なタグは以下の7個で、それ以外の場所で設定した場合は無視されます。  
&lt;activity&gt;, &lt;application&gt;, &lt;provider&gt;, &lt;service&gt;, &lt;receiver&gt;, &lt;activity-alias&gt;, &lt;path-permission&gt;  

下の例は&lt;activity&gt;に対して設定しています。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="example.com"
         ... >

    <application
            android:icon="@drawable/ic_launcher"
            android:label="@string/app_name">
        <!-- activityへのpermission属性指定は有効 -->
        <activity
                android:label="@string/app_name"
                android:name="com.example.service.serviceClass"
                android:permission="android.permission.CAMERA">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

パーミッションの使い方の詳細は、[Defining Permissions | Android Developers][1]やAndroid アプリのセキュア設計・セキュアコーディングガイドの[5.2. Permission と Protection Level][2]などを参照してください。

## 不適切な例

下の例では、permission属性が無効なactionタグに対して属性を指定しているため、READ_CONTACTSパーミッションを持たないアプリからもアクセスが可能です。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="example.com"
          ... >

    <application
            android:icon="@drawable/ic_launcher"
            android:label="@string/app_name">
        <activity
                android:label="@string/app_name"
                android:name="com.example.service.serviceClass">
            <intent-filter>
                <!-- actionへのpermission属性指定は無効 -->
                <action android:name="android.intent.action.MAIN"
                        android:permission="android.permission.READ_CONTACTS"/>
            </intent-filter>
        </activity>
     :
```

Lintは、permission属性が無効なタグに対する設定を検知すると、次のようなメッセージを出力します。

- Lint出力(warning)  
  "Protecting an unsupported element with a permission is a non-op and potentially dangerous."

注意: permissionタグに対する設定は無効ですが、Lintはこれを検知しません。逆に、path-permissionタグは設定が有効ですが、Lintはメッセージを出力します。

## 外部リンク

- [Defining Permissions | Android Developers][1]
- [Android アプリのセキュア設計・セキュアコーディングガイド　5.2. Permission と Protection Level][2]  
- [App Manifest | Android Developers][3]
- [&lt;permission&gt; | Android Developers][4]
- [&lt;activity&gt; | Android Develpers][5]
- [&lt;activity-alias&gt; | Android Developers][6]
- [&lt;application&gt; | Android Developers][7]
- [&lt;provider&gt; | Android Developers][8]
- [&lt;path-permission> | Android Developers][9]
- [&lt;receiver&gt; | Android Developers][10]
- [&lt;service&gt; | Developers][11]


[1]:https://developer.android.com/guide/topics/permissions/defining.html
[2]:http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#permission%E3%81%A8protection-level
[3]:https://developer.android.com/guide/topics/manifest/manifest-intro.html
[4]:https://developer.android.com/guide/topics/manifest/permission-element.html
[5]:https://developer.android.com/guide/topics/manifest/activity-element.html
[6]:https://developer.android.com/guide/topics/manifest/activity-alias-element.html
[7]:https://developer.android.com/guide/topics/manifest/application-element.html
[8]:https://developer.android.com/guide/topics/manifest/provider-element.html
[9]:https://developer.android.com/guide/topics/manifest/path-permission-element.html
[10]:https://developer.android.com/guide/topics/manifest/receiver-element.html
[11]:https://developer.android.com/guide/topics/manifest/service-element.html
