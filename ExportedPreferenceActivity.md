# PreferenceActivity should not be exported (ExportedPreferenceActivity)

## 警告されている問題点

[脆弱性のあるPreferenceActivity][1]のサブクラスを公開しているため、アプリ内のFragmentが任意のパラメータで起動されてしまうリスクがあります。

## 対策のポイント

PreferenceActivityサブクラス利用に関して、以下のいずれかの対策を行ってください。

- PreferenceActivityサブクラスを非公開にする（minSdkVersion18以下なら原則必須）
- 有効なFragmentであることを確認する

通常のActivityに関しては、別の対策が必要です。[Androidアプリのセキュア設計・セキュアコーディングガイド][0]を参考にしてください。

## 対策の具体例

### PreferenceActivityサブクラスを非公開にする

Android 4.3(API 18)以前の端末では、脆弱性を利用した攻撃が有効になるため、第三者アプリから利用できないようにすべてのPreferenceActivityサブクラスを非公開にしてください。
API19以降でも、第三者アプリからのActivity起動が必要ない場合は同様に非公開にしてください。

```
    <application
        ...>
        <activity android:name=".MyPreferenceActivity"
              ...
              android:exported="false"/>  <!-- 他アプリからのIntentによる呼び出しを禁止 -->
    </application>
```

### Fragmentを確認する

minSdkVersionが19以上の場合、PreferenceActivityサブクラスは、呼び出しFragmentの名前で使用の可不可を判定する必要があります。

```java
public class MyPreferenceActivity extends PreferenceActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        ...
    }

    @Override
    protected boolean isValidFragment(String fragmentName) {
        // fragmentNameを確認して、想定しているものだけを通す
        return XXXFragment.class.getName().equals(fragmentName) ||
                YYYFragment.class.getName().equals(fragmentName);
    }
}
```

Android 4.3(API 18)以前の端末では、上記実装があっても有効に働かないので原則非公開にしてください。

## 不適切な例

targetSdkVersionが19以上では、マニフェストでPreferenceActivityサブクラスを公開に設定し、isValidFragmentメソッドをオーバーライドしていない場合、常にRuntimeExceptionが発生します。
これを回避するため常にisValidFragmentメソッドがtrueを返すようにオーバーライドすると、このActivityが悪用されるリスクがあります。

```
        <uses-sdk android:targetSdkVersion="19"/>

        <activity android:name=".MyPreferenceActivity"
                  android:exported="true"/>  <!-- 他アプリからのIntentによる呼び出しを許可 -->
```

```
import android.preference.PreferenceActivity;
import android.os.Bundle;

public class MyPreferenceActivity extends PreferenceActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        // ...
    }

    @Override
    protected boolean isValidFragment(String fragmentName) {
        return true;  // すべてのFragmentによる起動が許可される
    }
}
```

Lintは、公開されているPreferenceActivityサブクラスのisValidFragmentメソッドがオーバーライドされていない（コードの正誤は検知しません）ことを検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "\`PreferenceActivity\` subclass \`com.example.exportedpreferenceactivity.MyPreferenceActivity\` should not be exported"

targetSdkVersionが18以下の場合、isValidFragmentメソッドはオーバーライドの有無にかかわらず常にtrueを返します。

## 外部リンク

-   [A New Vulnerability in the Android Framework: Fragment Injection][1]
-   [Fragment Injection脆弱性を修正する方法][2]
-   [Androidアプリのセキュア設計・セキュアコーディングガイド　4.1.3.6.PreferenceActivityのFragment Injection対策について][3]  


[0]: http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#activity%E3%82%92%E4%BD%9C%E3%82%8B%E3%83%BB%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B
[1]: http://securityintelligence.com/new-vulnerability-android-framework-fragment-injection
[2]: https://support.google.com/faqs/answer/7188427?hl=ja
[3]: http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#preferenceactivity%E3%81%AEfragment-injection%E5%AF%BE%E7%AD%96%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6