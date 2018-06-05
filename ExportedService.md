# Exported service does not require permission (ExportedService)

## 警告されている問題点

Serviceのアクセス制御が不十分なため、不正利用による情報の改竄や漏洩のリスクがあります。

## 対策のポイント

以下のいずれかの対策を実施してください。

- 第三者アプリからの利用を不許可にする
- パーミッションを指定して利用可能なアプリを限定する
- 利用を制限しないなら重要な情報を扱わない

## 対策の具体例

Serviceのexported属性が無指定である場合にそのService゙公開されるか非公開となるかは、intent-filterの定義の有無で変化します。
ですが、Serviceの公開/非公開はセキュリティ的に大変重要ですので、以下の例のように明示することを推奨します。
また、すべての場合において受信Intentの安全性を確認することが前提となります。

### 第三者アプリからの利用を不許可にする

他アプリからの利用が必要ない場合は、Serviceを非公開としてください。
exported属性を設定せず、intent-filterを持たない場合は非公開となるのがデフォルトの挙動ですが、明示的に非公開に設定します。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.service"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <!-- 非公開であることを明示 -->
        <service
            android:exported="false"
            android:label="@string/app_name"
            android:name="com.example.service.serviceClass"
            android:process=":remote" >
        </service>
    </application>
</manifest>
```

### パーミッションを指定して利用可能なアプリを限定する

Serviceに対してパーミッションを設定することで、利用側に同じパーミッションが必要となります。
下の例では、自社製アプリとの連携を前提に保護レベルを"signature"に設定した独自パーミッションを定義し、同じ秘密鍵で署名したアプリ(APK)からのみ呼び出しが可能な設定にしています。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.service"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />

    <!-- serviceで使用するパーミッションを宣言 -->
    <permission
        android:name="com.example.permission.GOGO" 
        android:protectionLevel="signature" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <!-- 明示的に公開し、パーミッションを設定 -->
        <service
            android:exported="true"
            android:label="@string/app_name"
            android:name="com.example.service.serviceClass"
            android:permission="com.example.permission.GOGO"
            android:process=":remote" >
            <intent-filter >
                <action android:name="com.example.service.serviceClass" >
                </action>
            </intent-filter>
        </service>
    </application>
</manifest>
```

下に示す例のように親の&lt;application&gt;タグにパーミッションの設定を行った場合も、下位の&lt;service&gt;に対するパーミッションの設定が有効となります。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.service"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />

    <!-- serviceで使用するパーミッションを宣言 -->
    <permission
        android:name="com.example.permission.GOGO" 
        android:protectionLevel="signature" />

    <!-- applicationレベルでパーミッションを設定 -->
    <application
        android:permission="com.example.permission.GOGO"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <!-- 明示的に公開 -->
        <service
            android:exported="true"
            android:label="@string/app_name"
            android:name="com.example.service.serviceClass"
            android:process=":remote" >
            <intent-filter >
                <action android:name="com.example.service.serviceClass" >
                </action>
            </intent-filter>
        </service>
    </application>
</manifest>
```

詳細は  [Android アプリのセキュア設計・セキュアコーディングガイド　5.2. Permission と Protection Level][3-2]を参照してください。

### 利用を制限しないなら重要な情報を扱わない

Serviceを公開し、パーミッションによる制限もせずに第三者アプリからの利用を許可する場合、不正なデータを受信することを前提にアプリを設計・実装してください。特に、結果を返す時に重要な情報を含めてはいけません。
Serviceをパーミッションで制限せずに公開する状態は、不適切な例と重なるためLintはメッセージを出力します。

## 不適切な例

下の例では、android:exported属性の設定がありませんが&lt;intent-filter&gt;があるため暗黙的に公開となります。しかも、パーミッションによる保護もないため、重要情報などを返すようになっていると情報漏洩などのリスクがあります。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />
    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <!-- intent-filterがあるためserviceが暗黙的に公開される状態 -->
        <service
            android:label="@string/app_name"
            android:name="com.example.service.serviceClass"
            android:process=":remote" >
            <intent-filter >
                <action android:name="com.example.service.serviceClass" >
                </action>
            </intent-filter>
        </service>
    </application>
</manifest>
```

Lintは、公開されているServiceに対してパーミッションの設定がないことを検知すると、次のようなメッセージを出力します。

- Lint出力(Warning)  
  "Exported service does not require permission."

## 外部リンク

- [Service | Android Developers][1]
- [&lt;service&gt; | Android Developers][2]
- [Android アプリのセキュア設計・セキュアコーディングガイド][3]  
  [「4.4. Service を作る・利用する」][3-1]に Service を安全に使用するための指針や実装例が解説されています

[1]:https://developer.android.com/guide/components/services.html
[2]:https://developer.android.com/guide/topics/manifest/service-element.html
[3]:http://www.jssec.org/dl/android_securecoding/
[3-1]:http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#service%E3%82%92%E4%BD%9C%E3%82%8B%E3%83%BB%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B
[3-2]:http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#permission%E3%81%A8protection-level

