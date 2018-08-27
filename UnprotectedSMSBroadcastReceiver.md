# Unprotected SMS BroadcastReceiver(UnprotectedSMSBroadcastReceiver)

## 警告されている問題点

BroadcastReceiverの保護が不十分なため、意図しないSMS Broadcast[^注釈1]を受信し、フィッシングサイトへの誘導や重要情報の漏洩のリスクがあります。

## 対策のポイント

- Broadcastの送信元にBROADCAST\_SMSパーミッションが付与されていることを確認する

note: Android 6.0(API 23)以上の端末では、第三者アプリがSMS Broadcast[^注釈1]を送信できないため、minSdkVersionが23以上の場合は、[Unsafe Protected BroadcastReceiver][UnsafeProtectedBroadcastReceiver]の項の受信Intentの安全性を確認する対策でも問題が発生しなくなります。

## 対策の具体的例

Signatureパーミッション[^注釈2]であるandroid.permission.BROADCAST\_SMSをBroadcastReceiverに指定することで、第三者アプリが送信したIntentを受信しないようにできます。

(静的BroadcastReceiverの例)

```
<receiver
    android:name=".SMSReceiver"
    android:permission="android.permission.BROADCAST_SMS">
    <intent-filter>
        <action android:name="android.provider.Telephony.SMS_RECEIVED" />
    </intent-filter>
</receiver>
```

(動的BroadcastReceiverの例)

```java
public class MainActivity extends AppCompatActivity  {
    private BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onCreate(Context context, Intent intent) {
            ...
        }
    };

    @Override
    public void onResume() {
        super.onResume();
　　　　// IntentFilterでSMS受信を設定し、パーミッションを指定する
        registerReceiver(mReceiver,
            new IntentFilter(Telephony.Sms.Intents.SMS_RECEIVED_ACTION),
            Manifest.permission.BROADCAST_SMS,
            new Handler(getMainLooper()));
    }

    @Override
    public void onPause() {
        super.onPause();
        unregisterReceiver(mReceiver);
    }
}
```

## 不適切な例

第三者アプリが送信するSMS Broadcastを無条件に受信し処理してしまうと、攻撃者が細工した​データが含まれている可能性があり危険です。

（静的BroadcastReceiverの例）

```
<!-- BROADCAST_SMSパーミッションの指定なし -->
<receiver android:name=".MySMSBroadcastReceiverNoPermission"
    android:exported="true">
    <intent-filter>
        <action android:name="android.provider.Telephony.SMS_RECEIVED" />
    </intent-filter>
</receiver>
```

（動的BroadcastReceiverの例）

```java
   @Override
    protected void onResume() {
        super.onResume();
        // IntentFilterのみ設定し、パーミッションを指定していない
        registerReceiver(mReceiverNoPermission,
                new IntentFilter(Telephony.Sms.Intents.SMS_RECEIVED_ACTION));
    }
```

Lintは、前者の静的ReceiverのようにSMS Broadcast[^注釈1]を受信する設定とし、かつ、BROADCAST\_SMS パーミッションの指定がないことを検知すると、次のようなメッセージを出力します。

- Lint出力(Warning)  
    "BroadcastReceivers that declare an intent-filter for SMS_DELIVER or SMS_RECEIVED must ensure that the caller has the BROADCAST_SMS permission, otherwise it is possible for malicious actors to spoof intents."

Lintは、後者の動的Receiverの例を検知できません。

## 外部リンク

  - [BroadcastReceiver | Android Developers][1]  
    
[1]: https://developer.android.com/reference/android/content/BroadcastReceiver.html

[UnsafeProtectedBroadcastReceiver]: UnsafeProtectedBroadcastReceiver.md


[^注釈1]: javascript:void(0); "SMS Broadcast：SMS_DELIVERまたはSMS_RECEIVEをActionに持つBroadcast Intentのこと。"
[^注釈2]: javascript:void(0); "Signatureパーミッション：同じ証明書で署名されているアプリにのみ利用を許可するパーミッションのこと。"

