# AllowBackup/FullBackupContent Problems (AllowBackup)

## 警告されている問題点

アプリデータのバックアップ/リストア許可設定を適切に行っていない可能性があるため、アプリの重要情報の漏洩や改竄のリスクがあります。

## 対策のポイント

バックアップ/リストアに関して、以下のいずれかの対策を実施してください。

- 原則、アプリのデータのバックアップ/リストアを不許可とする
- バックアップ対象を公開されても問題ないデータに限定する

## 対策の具体例

### アプリのデータのバックアップ/リストアを許可しない

アプリがバックアップを必要とするデータを持たないなら、情報が公開されるリスクを減らすためバックアップ/リストアを不許可に設定してください。

```
    <application
        android:allowBackup="false"                  ≪バックアップ不許可≫
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" />

```

### バックアップ対象を公開されても問題ないデータに限定する

バックアップされたファイルは、ユーザーによる自由な読み書きが可能です。
そのため、ユーザーに公開したくない（改竄されたくない）情報はバックアップ対象にしないでください。

また、別の端末でリストアされた場合を考慮し、デバイスを特定するような識別データは除外する必要があります。
例えば、Google Cloud Messaging(GCM)の登録IDを別の端末にリストアしてしまうとGCMメッセージを正しく受信しなくなります。

Android5.1（API22）以前の端末をサポートする場合は、BackupAgentを実装する必要があります。
この実装の詳細に関しては、[Android Devlopers][3]を参照してください。
実装したBackupAgentを使用するにはマニフェストに以下のように記述します。

```
    <application
        android:allowBackup="true"                   ≪バックアップ許可≫
        android:backupAgent="MyBackupAgent"          ≪実装したBackupAgentクラス≫
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" />
```

Android 6.0 (API 23)以降の端末では、[Auto Backup][1]の機能を利用することで、設定ファイル(XML)によりファイルやフォルダ単位の指定を行うことができます。

```
    <application
        android:allowBackup="true"                   ≪バックアップ許可≫
        android:fullBackupContent="@xml/my_backup"   ≪バックアップ対象を記述したファイル≫
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" />

```

上の場合、res/xml/my_backup.xmlファイルにバックアップ対象を記述します。
my_backup.xmlに下のようにincludeで対象、excludeで対象外を指定します。
設定ファイルの書き方の詳細は、[Android Devlopers][1-3]を参照してください。  

```
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <include domain="sharedpref" path="."/>
    <exclude domain="database" path="device_info.db" />
    <include domain="file" path="need_to_backup.txt" />
    <exclude domain="file" path="instant-run" />
</full-backup-content>
```

バックアップの対象・非対象を判断する際は、[XML Config Syntax | Android Developers][1-2]も参考にしてください。

Android 5.1以前と6.0以降の端末どちらもサポートする場合は、下のようにfullBackupOnly属性を使用します。
これをtrueに設定することで6.0以降ではBackupAgentを使用せず、fullBackupContent属性に指定した設定が使用されます。

```
    <application
        android:allowBackup="true"                   ≪バックアップ許可≫
        android:fullBackupContent="@xml/my_backup"   ≪バックアップ対象を記述したファイル[6.0以降]≫
        android:backupAgent="MyBackupAgent"          ≪実装したBackupAgentクラス[5.1以前]≫
        android:fullBackupOnly="true"                ≪6.0以降はBackupAgentを不使用≫
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" />
```

## 不適切な例

### バックアップの意思を明確にしていない

allowBackup属性を指定しない場合、デフォルトでバックアップが許可され、すべてのアプリファイルがバックアップの対象になります。

```
    <!-- allowBackup属性の設定なしはバックアップ許可 -->
    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" />
```

Lintは、allowBackup属性の指定がないことを検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "Should explicitly set \`android:allowBackup\` to \`true\` or \`false\` (it's \`true\` by default, and that can have some security implications for the application's data)"

### バックアップ対象を明確にしていない

BackupAgentや設定ファイル(XML)でバックアップ対象になるファイルやデータを明示的に指定しなければ、すべてのアプリファイルがバックアップの対象になります。

```
    <!-- allowBackup属性でバックアップを許可しているが、fullBackupContent属性の指定なし -->
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" />
```

Lintは、fullBackupContent属性の指定がないことを検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "On SDK version 23 and up, your app data will be automatically backed up, and restored on app install. 
    Consider adding the attribute \`android:fullBackupContent\` to specify an \`@xml\` resource which configures which files to backup. More info: https://developer.android.com/training/backup/autosyncapi.html"

注意: backupAgent属性の有無に関しては検知しません。

加えて、マニフェストに[GCM Receiver][4]が定義されていることを検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "On SDK version 23 and up, your app data will be automatically backedup, and restored on app install. Your GCM regid will not work across restores, so you must ensure that it is excluded from the back-up set.Use the attribute \`android:fullBackupContent\` to specify an \`@xml\` resource which configures which files to backup. More info: https://developer.android.com/training/backup/autosyncapi.html"

fullBackupContent属性の指定があってもファイルが存在しない場合は、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "Missing \`&lt;full-backup-content&gt;\` resource"

## 外部リンク

-   [Back Up User Data with Auto Backup | Android Devlopers][1]
-   [XML Config Syntax | Android Developers][1-2]
-   [Including and excluding files | Android Devlopers][1-3]
-   [BackupAgent | Android Developers][2]
-   [Back Up Key-Value Pairs with Android Backup Service | Android Developers][3]
-   [Set up a GCM Client App on Android | Google Developers][4]


[1]: https://developer.android.com/guide/topics/data/autobackup.html
[1-2]: https://developer.android.com/guide/topics/data/autobackup.html#XMLSyntax
[1-3]: https://developer.android.com/guide/topics/data/autobackup.html#IncludingFiles
[2]: https://developer.android.com/reference/android/app/backup/BackupAgent.html
[3]: https://developer.android.com/guide/topics/data/keyvaluebackup.html
[4]: https://developers.google.com/cloud-messaging/android/client
