# Unsafe Protected BroadcastReceiver(UnsafeProtectedBroadcastReceiver)

## 警告されている問題点

受信したIntentの安全性を確認していないため、意図しない処理の実行や重要情報漏洩のリスクがあります。

## 対策のポイント

- 受信Intentの安全性を確認する

## 対策の具体例

公開BroadcastReceiverは、悪意のある偽装Intentを受け取る可能性があります。
非公開BroadcastReceiverであっても、同一アプリ内で他のアプリから受け取った情報を転送することがあるため、受信したIntentの安全性を確認してください。
BroadcastReceiverの公開非公開に拘らず、受信したIntentのAction文字列を最低限確認し、データの安全性も十分に確認してください。

```
    <receiver
        android:name="com.example.BatteryLowReceiver"
        android:exported="true">
        <intent-filter>
            <action android:name="com.example.broadcast.MY_BROADCAST_PUBLIC"/>
        </intent-filter>
    </receiver>
```

```java
public class BatteryLowReceiver extends BroadcastReceiver {
    private static final String MY_BROADCAST_PUBLIC = 
            "com.example.broadcast.MY_BROADCAST_PUBLIC";

    @Override
    public void onReceive(Context context, Intent intent) {
        // 受信したIntentのAction文字列が期待したものであることを確認する
        if (!MY_BROADCAST_PUBLIC.equals(intent.getAction())) return;

        Bundle extras = intent.getExtras();
        //データの安全性も確認する
        ...
    }
}
```

注意: アプリがAndroid 8.0を対象にしている場合は、暗黙的Broadcastを受信する静的Receiverを登録できないため、動的Receiverを使用する必要があります。ただし、静的Receiver登録が可能な[例外][broadcast_exception]があります。

非公開BroadcastReceiverに対しては、intent-filterにシステムだけが送信するBroadcast[^注釈1]のみ設定するようにしてください。
同一アプリ内に指定のIntentを投げたつもりでも、他アプリの公開Receiverが受け取る可能性があるからです。

```
    <receiver
        android:name=".Receiver"
        android:exported="false">
        <!-- 非公開BroadcastReceiverに対しては、システムだけが送信するBroadcastのみ設定する -->
        <intent-filter>
            <action android:name="android.intent.action.BATTERY_LOW" />
        </intent-filter>
    </receiver>
```

## 不適切な例

受け取ったIntentには攻撃者が細工した​データが含まれている可能性があるため、Intentの安全性を確認せずに使用することにはリスクがあります。

（動的BroadcastReceiverの例）

```java
public class MainActivity extends AppCompatActivity  {
    private BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            //Action文字列もデータも確認なし
            ...
        }
    };

    @Override
    protected void onResume() {
        super.onResume();
        registerReceiver(mReceiver, new IntentFilter(Intent.ACTION_BATTERY_LOW));
    }

    @Override
    protected void onPause() {
        super.onPause();
        unregisterReceiver(mReceiver);
    }
}
```

（静的BroadcastReceiverの例）

```
    <receiver android:name=".Receiver">
        <intent-filter>
            <action android:name="android.intent.action.BATTERY_LOW" />
        </intent-filter>
    </receiver>
```

```java
public class BatteryLowReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        //Action文字列の確認なし
        ...

        Bundle extras = intent.getExtras();
        ...
    }
}
```

Lintは、後者の静的Receiverのように&lt;intent-filter&gt;でシステム送信のBroadcastを受信する設定とし、かつ、受信したIntentのAction文字列の未確認を検知すると、次のようなメッセージを出力します。

  - Lint出力(Warning)  
    "This broadcast receiver declares an intent-filter for a protected broadcast action string, which can only be sent by the system, not third-party applications. However, the receiver's onReceive method does not appear to call getAction to ensure that the received Intent's action string matches the expected value, potentially making it possible for another actor to send a spoofed intent with no action string or a different action string and cause undesired behavior."

## 外部リンク

  - [Androidアプリのセキュア設計・セキュアコーディングガイド 4.2 Broadcast を受信する・送信する][1]  
  - [Android 8.0での暗黙的Broadcastの動作変更][2]
  - [暗黙的なブロードキャストの例外(Implicit Broadcast Exceptions)][broadcast_exception]

[1]: http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#broadcast%E3%82%92%E5%8F%97%E4%BF%A1%E3%81%99%E3%82%8B%E3%83%BB%E9%80%81%E4%BF%A1%E3%81%99%E3%82%8B
[2]: https://developer.android.com/about/versions/oreo/android-8.0-changes.html
[broadcast_exception]: https://developer.android.com/guide/components/broadcast-exceptions.html


[^注釈1]: javascript:void(0); "システムだけが送信するBroadcast：Androidシステムが送信するBroadcast。BATTERY_LOW、BOOT_COMPLETED、SCREEN_ON等が該当する。"

