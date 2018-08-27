# Receiver does not require permission (ExportedReceiver)

## 警告されている問題点

Receiverのアクセス制御が不十分なため、不正なブロードキャストによる情報の改竄や漏洩のリスクがあります。

## 対策のポイント

以下のいずれかの対策を実施してください。

- 第三者アプリからブロードキャストを受信しない
- パーミッションを指定して受信するブロードキャストを限定する
- 受信するブロードキャストを制限しないなら重要な情報を扱わない

## 対策の具体例

Receiverのexported属性が無指定である場合にそのReceiverが公開されるか非公開となるかは、intent-filterの定義の有無で変化します。
最初にコードを書くときにはその挙動を理解できていると思いますが、後の修正や他の開発者のためにも、以下の例のようにexported属性を明示することを推奨します。
また、すべての場合において[受信Intentの安全性を確認する][UnsafeProtectedBroadcastReceiver]ことが前提となります。

### 第三者アプリからブロードキャストを受信しない

他アプリからBroadcastを受信する必要がない場合は、Receiverを非公開としてください。
exported属性を設定せず、intent-filterを持たない場合は非公開となるのがデフォルトの挙動ですが、明示的に非公開に設定します。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.exportedreceiver"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />
    <application
        android:icon="drawable/ic_launcher"
        android:label=“string/app_name" >
        <!-- 非公開であることを明示 -->
        <receiver
            android:exported="false"
            android:label="string/app_name”
            android:name=“com.example.exportedreceiver.Receiver”>
        </receiver>
    </application>
</manifest>
```

### パーミッションを指定して受信するブロードキャストを限定する

Receiverに対してパーミッションを設定することで、送信側と受信側に同じパーミッションが必要となります。
下の例では、自社製アプリとの連携を前提に保護レベルを"signature"に設定した独自パーミッションを定義し、同じ秘密鍵で署名したアプリ(APK)からのみ呼び出しが可能な設定にしています。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.exportedreceiver"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />

    <!-- receiverで使用するパーミッションを宣言 -->
    <permission
        android:name="com.example.permission.RECEIVE_MAGIC"
        android:protectionLevel="signature" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >

        <!-- 明示的に公開し、パーミッションを設定 -->
        <receiver
            android:exported="true"
            android:label="@string/app_name"
            android:name="com.example.exportedreceiver.Receiver1"
            android:permission="example.com.permission.RECEIVE_MAGIC"
            android:process=":remote" >
            <intent-filter>
                <action android:name="com.example.exportedreceiver.Receiver" >
                </action>
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

下の例のように親の&lt;application&gt;でパーミッションの設定を行った場合も、下位の&lt;receiver&gt;に対するパーミッションの設定が有効となります。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.exportedreceiver"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />

    <!-- receiverで使用するパーミッションを宣言 -->
    <permission
        android:name="com.example.permission.RECEIVE_MAGIC"
        android:protectionLevel="signature" />

    <!-- applicationのレベルでパーミッションを設定 -->
    <application
        android:icon="@drawable/ic_launcher"
        android:permission="com.example.permission.RECEIVE_MAGIC"
        android:label="@string/app_name" >
        <!-- 明示的に公開 -->
        <receiver
            android:exported="true"
            android:label="@string/app_name"
            android:name="com.example.exportedreceiver.Receiver"
            android:process=":remote" >
            <intent-filter>
                <action android:name="com.example.exportedreceiver.Receiver" >
                </action>
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

詳細は  [Android アプリのセキュア設計・セキュアコーディングガイド　5.2. Permission と Protection Level][6-2]を参照してください。

### 受信するブロードキャストを制限しないなら重要な情報を扱わない

Receiverを公開し、パーミッションによる制限もせずに第三者アプリからのブロードキャストを受信する場合、不正なデータを受信することを前提にアプリを設計・実装してください。特に、結果を返す時に重要な情報を含めてはいけません。
ブロードキャストをパーミッションで制限せずに公開する状態は不適切な例と重なるため、Lintはメッセージを出力します。

## 不適切な例

下の例では、exported属性の設定がありませんが&lt;intent-filter&gt;があるため暗黙的に公開となります。パーミッションによる保護もないため、重要情報などを返すようになっていると情報漏洩などのリスクがあります。

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.exportedreceiver"
    android:versionCode="1"
    android:versionName="1.0" >
    <uses-sdk android:minSdkVersion="14" />
    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <!-- intent-filterがあるためreceiverが暗黙的に公開される -->
        <receiver
            android:label="@string/app_name"
            android:name="com.example.exportedreceiver.Receiver2" >
            <intent-filter>
                <action android:name="com.example.exportedreceiver.Receiver2" >
                </action>
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

Lintは、公開されているReceiverに対しパーミッションの設定がないことを検知すると、次のようなメッセージを出力します。

- Lint 出力(Warning)  
  "Exported receiver does not require permission."

## 外部リンク

- [&lt;receiver&gt; | Developer][1]
- [BroadcastReceiver | Android Developers][2]
- [Broadcast一般に関する解説記事][3]
- [&lt;application&gt; | Developer][4]
- [InstallReferrerReceiver][5]
- [Android アプリのセキュア設計・セキュアコーディングガイド][6]  
  [「4.2. Broadcast を受信する・送信する」][6-1]に Receiver を安全に使用するための指針や実装例が解説されています  

    
[1]:https://developer.android.com/guide/topics/manifest/receiver-element.html
[2]:https://developer.android.com/reference/android/content/BroadcastReceiver.html
[3]:https://developer.android.com/guide/components/broadcasts.html
[4]:https://developer.android.com/guide/topics/manifest/application-element.html
[5]:https://developers.google.com/android/reference/com/google/android/gms/tagmanager/InstallReferrerReceiver
[6]:http://www.jssec.org/dl/android_securecoding/
[6-1]:http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#broadcast%E3%82%92%E5%8F%97%E4%BF%A1%E3%81%99%E3%82%8B%E3%83%BB%E9%80%81%E4%BF%A1%E3%81%99%E3%82%8B
[6-2]:http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#permission%E3%81%A8protection-level

[UnsafeProtectedBroadcastReceiver]: UnsafeProtectedBroadcastReceiver.md

