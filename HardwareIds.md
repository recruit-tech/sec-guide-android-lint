# Hardware Id Usage (HardwareIds)

## 警告されている問題点

デバイス(端末)に基づく識別子を個人情報に関連付けている場合、不正アクセスや、ユーザーのプライバシー侵害につながるリスクがあります。

## 対策のポイント

- デバイスに基づく識別子を個人情報に紐づけて用いることを避ける

## 対策の具体例

アプリの目的・用途を考慮し、以下の識別子の利用を検討してください。

|識別子|特徴|
|:----:|:----:|
|広告ID|ユーザーがリセットできる匿名かつ固有の識別子で、広告のユースケースに適している|
|インスタンスID|アプリがインストールされている間だけ維持されるため、比較的簡単にリセット可能|
|UUID|グローバルに一意であるため、特定のアプリインスタンスの識別に使用可能|

### 広告IDを利用する

広告とユーザーの分析を目的とする場合は、以下の方法で取得できる広告IDを使用してください。
ただし、取得前に[インタレストベース広告をオプトアウト]の設定のステータスを確認してください。

```java
import com.google.android.gms.ads.identifier.AdvertisingIdClient;
    :
    String AdId = null;

    try {
        AdvertisingIdClient.Info info = AdvertisingIdClient.getAdvertisingIdInfo(context);
        //「インタレストベース広告をオプトアウト」の設定を確認した上で広告IDを取得する
        if (info.isLimitAdTrackingEnabled()) {
            // チェックされていてもgetIdメソッドは広告IDを取得しますが、設定を尊重してください
            ...
        } else {
            AdId = info.getId();
        }
    } catch (IOException e) { // GooglePlayServices接続失敗
        ...
    } catch (GooglePlaySericesNotAvailableException e) { // GooglePlay未インストール
        ...
    } catch (GooglePlayServicesRepairableException e) { // GooglePlayServices使用不可
        ...
    }
```

広告IDを利用する場合、Googleの定めた[広告IDの利用規約][4]に従ってください。

### インスタンスIDを利用する

Google提供のInstance ID Serviceを利用し、アプリのインスタンスごとにユニークな識別子を取得することができます。
この識別子はアプリのインスタンスを識別し、トラッキングに用いることが可能ですが、Google Playによって配布されたアプリに限られます。

```java
// 発行済みのInstance IDを取得する
String iid = InstanceID.getInstance(context).getId()

// 実際の識別にはトークンを取得して確認する
String authorizedEntity = PROJECT_ID; // Google Developer Consoleが生成するProject idが必要
String scope = "GCM";
String token = InstanceID.getInstance(context).getToken(authorizedEntity, scope);
    :
```

インスタンスIDおよびその利用方法についての詳細は[InstanceID Androidでの実装][7]や[Google API InstaceID解説][8]を参照してください。

### UUID を利用する

インスタンスIDが利用できないケースではUUIDを利用して、アプリのインスタンスごとにユニークなIDを割り振る方法もあります。UUIDを作成するにはjava.util.UUIDクラスを利用します。


```java
String uniqueID = UUID.randomUUID().toString();
```

詳細は[Working with Instance IDs and GUIDs][9]を参照してください。

## 不適切な例

ハードウェアに紐づいた識別子は、

- ユーザーが値を変えることが出来ない、または困難
- 複数のアプリが同一の値を利用する

といった特徴を持つため、以下のようなリスクが考えられます。

- ユーザーが長期間のトラッキングの対象になる  
- 個々のアプリが扱う情報を「名寄せ」することが可能  
- 前のデバイス所有者の情報にアクセス出来てしまう

以上のことから、下表に示すようなデバイスに紐づく識別子を認証のベースとして用いたり、ユーザー情報と一緒に扱うことは推奨されていません。

|識別子|特徴|
|:----:|:----:|
|BluetoothおよびWi-FiのMACアドレス|スキャンする時はパーミッションが必要|
|ANDROID ID|端末の最初の設定時に生成され、ファクトリーリセットすることで値が再生成|
|Telephony Managerに関する情報|READ_PHONE_STATEパーミッションが必要|
|シリアル番号|Android2.3以上の非携帯電話端末と一部の携帯電話端末で取得可能|

### MACアドレスのアドレスを使用する

BluetoothやWi-FiのMACアドレスはデバイスに紐づいており、識別子として用いることは前述のとおり不適切です。

```java
    BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
    // BluetoothのMACアドレスを取得
    String bluetoothAddress = adapter.getAddress();

    WifiManager wm = (WifiManager) getSystemService(WIFI_SERVICE);
    WifiInfo info = wm.getConnectionInfo();
    // Wi-FiのMACアドレスを取得
    String wifiAddress =  info.getMacAddress();
```

Lintは上のようにBluetoothAdapter.getAddressメソッドおよびWifiInfo.getMacAddressメソッドの呼び出しを検知すると次のようなメッセージを出力します。

- Lint出力(Warning)  
  "using 'getAddress' to get identifiers is not recommended."
  "using 'getMacAddress' to get identifiers is not recommended."

注意: Android 6.0(API 23)で常にMACアドレスは、02:00:00:00:00:00を返すようになり、取得は不可能になっています。

### ANDROID IDを使用する

下記のようにして取得できる文字列(Android ID)はデバイスに紐づいており、識別子として用いることは前述のとおり不適切です。

```
    // Android IDを取得
    String androidId = Settings.Secure.getString(context.getContentResolver(), Settings.Secure.ANDROID_ID);
```

Lintは上のようにAndroid IDを取得しているのを検出すると次のようなメッセージを出力します。

- Lint出力(Warning)  
  "using 'getString' to get identifiers is not recommended."

### Telephony Managerから取得可能なデバイスに紐づいた識別子を使用する

Telephony Managerから以下のようなデバイスに紐づいた値を取得可能ですが、これらを識別子として使用することは前述のとおり不適切です。

```
    TelephonyManager telMgr = (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);
    // デバイスIDを取得
    St ring deviceid = telMgr.getDeviceId();
    // 電話番号を取得
    String phoneNumber = telMgr.getLine1Number();
    // SIMシリアルナンバーを取得
    String simSerialNumber = telMgr.getSimSerialNumber();
    // サブスクライバーIDを取得
    String subscriberId = telMgr.getSubscriberId();
```

Lintは上のようなメソッドの呼び出しを検知すると、それぞれ次のようなメッセージを出力します

- Lint出力(Warning)  
  "using 'getDeviceId' to get identifiers is not recommended."  
  "using 'getLine1Number' to get identifiers is not recommended."  
  "using 'getSerial' to get identifiers is not recommended."  
  "using 'getSubscriberId' to get identifiers is not recommended."

### デバイスのシリアル番号を使用する

デバイスのシリアル番号を識別子として使用することは前述のとおり不適切です。

```
import java.lang.reflect.Method;
    :
    String serialNo;

    // SERIALプロパティ利用
    serialNo = android.os.Build.SERIAL;

    // 内部API利用
    try {
        Class<?> c = Class.forName("android.os.SystemProperties");
        Method get = c.getMethod("get", String.class);
        serialNo = (String) get.invoke(null, "ro.serialno");
    } catch (Exception e) {
    }

    // getSerialメソッド利用
    serialNo = android.os.Build.getSerial();
```

Lintは上のようなシリアル番号の取得を検知すると、それぞれ次のような警告メッセージを出力します

- Lint出力(Warning)  
  "using ‘SERIAL’ to get identifiers is not recommended."    
  "using ‘ro.serial’ to get identifiers is not recommended."

注意: getSerialメソッドは現在のlintは検知しません。

## 外部リンク

- [一意識別子のベストプラクティス][1]
- [AdvertisingIdClient | Android Developers][2]
- [AdvertisingIdClient.Info | Android Developers][3]
- [広告IDの利用規約][4]
- [InstanceID | Android Developers][5]
- [インスタンスIDとは何か][6]
- [InstanceID Androidでの実装][7]
- [Google API InstaceID解説][8]
- [Working with Instance IDs and GUIDs][9]

[1]:https://developer.android.com/training/articles/user-data-ids.html
[2]:https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient
[3]:https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient.Info
[4]:https://play.google.com/intl/ja_ALL/about/monetization-ads/ads/ad-id/
[5]:https://developers.google.com/android/reference/com/google/android/gms/iid/InstanceID
[6]:https://developers.google.com/instance-id/
[7]:https://developers.google.com/instance-id/guides/android-implementation
[8]:https://developers.google.com/android/reference/com/google/android/gms/iid/InstanceID
[9]:https://developer.android.com/training/articles/user-data-ids.html#working_with_instance_ids_&_guids

