# Content provider does not require permission (ExportedContentProvider)

## 警告されている問題点

ContentProviderのアクセス制御が不十分なため、第三者アプリからの操作による情報の漏洩や改竄のリスクがあります。

## 対策のポイント

ContentProviderに関して、以下のいずれかの対策を実施してください。

- 第三者アプリからの利用を不許可にする
- パーミッションを指定して利用可能なアプリを限定する
- 利用を制限しないなら重要な情報を扱わない

## 対策の具体例

ContentProviderのexported属性は、バージョンにより無効であったり、デフォルト値が異なるので、以下の例のように明示することを推奨します。
具体的には、Android 2.2(API 8)以前の端末ではContentProviderを非公開にできないため、Android 2.2(API 8)以前の端末を非サポートにする(minSdkVersionを9以上にする)ことが必要です。デフォルト値については、targetSdkVersionが16以下で公開(true)、17以上で非公開(false)です。
また、すべての場合において受信データの安全性を確認することが前提となります。

### 第三者アプリからの利用を不許可にする

他アプリからの利用を許可しない場合はContentProviderを非公開としてください。
下に示す例のように、&lt;provider&gt;でexported属性をfalseに設定することで、他アプリからのアクセスを不許可にできます。

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="9" />
    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <!-- 非公開であることを明示 -->
        <provider
            android:exported="false"
            android:name="com.example.provider.providerClass"
            android:authorities="com.example.provider.providerData">
        </provider>
    </application>
</manifest>
```

### パーミッションを指定して利用可能なアプリを限定する

以下にパーミッション指定によるいくつかの対策例を示します。必要に応じて、ContentProviderのデータのパスを限定したり、[権限の再移譲問題][6]が起きないようにパーミッションを設定してください。

最初に、扱うデータが連絡先(Contacts)情報の場合で、アクセスをパーミッションで許可する例を示します。
下の例では、第三者アプリはREAD_CONTACTSパーミッションを持たなければContentProviderを利用できません。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
        :
    <application ... >     
        <!-- android:permission属性で他アプリにパーミッションの保持を要求するように設定する -->
        <provider
            android:exported="true"
            android:name="com.example.provider.MyProvider"
            android:authorities="com.example.provider.providerData"
            android:permission="android.permission.READ_CONTACTS">
        </provider>
        :
    </application>
    :
</manifest>
```

ContentProviderは他のコンポーネント(Activity, Service, BroadcastReceiver)と異なり、読み書き２つのパーミッションの設定が可能です。

次の例では、読み書きそれぞれにパーミッションを設定しています。第三者アプリは、READ\_CONTACTSパーミッションを持たなければContentProviderからの情報を読み取るqueryメソッドを利用することができません。同様に、WRITE\_CONTACTSパーミッションを持たなければinsert、update、およびdeleteメソッドによる書き込みが制限されます。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
        :
    <application ... > 
        <!-- 読み書きそれぞれ別のパーミッションを設定 -->
        <provider
            android:exported="true"
            android:name="com.sample.provider.MyProvider"
            android:authorities="com.example.provider.providerData"
            android:readPermission="android.permission.READ_CONTACTS"
            android:writePermission="android.permission.WRITE_CONTACTS">
        </provider>
        :
    </application>
    :
</manifest>
```

次の例は、アクセスを許可する情報の範囲を特定のパターンにマッチするパスに限定しています。
path-permissionタグでは、さらに細かくパスやパーミッションを設定することも可能です。詳細は[Android Developers][3]を参照してください。
```
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <permission android:name="com.example.provider.MySignaturePermission" android:protectionLevel="signature"/>
        :
    <application ... >
        <!-- Signatureパーミッションで全体的に保護 -->
        <!-- <path-permission>でアクセス可能なパスを限定 -->
        <provider
            android:exported="true"
            android:name="com.example.provider.MyProvider"
            android:authorities="com.examble.provider.providerData"
            android:permission="com.example.provider.MySignaturePermission">
            <path-permission
                android:pathPrefix="/you_can_access_here"
                android:permission="android.permission.READ_CONTACTS">
            </path-permission>
        </provider>
        :
    </application>
    :
</manifest>
```

### 利用を制限しないなら重要な情報を扱わない

ContentProviderを公開し、パーミッションによる制限もせずに第三者アプリからの利用を許可する場合、不正なデータを受信することを前提にアプリを設計・実装してください。特に、結果を返す時に重要な情報を含めてはいけません。
ContentProviderをパーミッションで制限せずに公開する状態は、不適切な例と重なるためLintはメッセージを出力します。

## 不適切な例

以下の例では、targetSdkVersionが16以下の場合、ContentProviderが制限なく公開されるため、マルウェアなどに不正なアクセスを許し、情報の漏洩や改竄のリスクがあります。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest ...>
    :
    <application
       ... >
        <!-- exported属性を設定していないので、targetSdkVersionが16以下では公開扱い -->
        <provider
            android:name="com.sample.provider.MyProvider"
            android:authorities="com.sample.provider.providerData">
        </provider>
    </application>
</manifest>
```

Lintは、ContentProviderがパーミッションもパスも制限なく公開されていることを検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    “Exported content providers can provide access to potentially sensitive data.”

## 外部リンク

- [Content Provider | Android Developers][1]
- [Content Providerのガイド][2]
- [&lt;path-permission&gt; | Android Developers][3]
- [Androidアプリのセキュア設計・セキュアコーディングガイド][4]  
  [「4.3 Content Providerを作る・利用する」][4.3]にContent Providerを安全に使用するための指針や実装例が解説されています
  [「5.2 PermissionとProtection Level」][5.2]にパーミッションの適切な使用方法の解説があります

[1]:https://developer.android.com/reference/android/content/ContentProvider.html
[2]:https://developer.android.com/guide/topics/providers/content-providers.html
[3]:https://developer.android.com/guide/topics/manifest/path-permission-element.html
[4]:http://www.jssec.org/dl/android_securecoding/
[4.3]:http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#content-provider%E3%82%92%E4%BD%9C%E3%82%8B%E3%83%BB%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B
[5.2]:http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#permission%E3%81%A8protection-level
[6]:http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#permission%E3%81%AE%E5%86%8D%E5%A7%94%E8%AD%B2%E5%95%8F%E9%A1%8C



