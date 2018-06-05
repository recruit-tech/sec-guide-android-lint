# Content provider shares everything (GrantAllUris)

## 警告されている問題点

ContentProviderの全リソース利用が他のアプリに許可されているため、重要な情報が漏洩・改竄されるリスクがあります。

## 対策のポイント

- アクセスを許可するリソースを必要最小限にする

より適切なアクセス制御を行うには、[Content provider does not require permission][4]の項も参照してください。

## 対策の具体例

下の例は、リソースの一時的なアクセスを"/for_sharing/this_directory_only/"以下に限定するための設定です。

```
<!-- exported属性をfalseにすることで、基本的に外部からの利用不可 -->
<!-- grantUriPermissions属性をfalseにすることで、一時許可するパスをマニフェストでの指定に限定 -->
<provider
    android:exported="false"
    android:grantUriPermissions="false"
    android:name=...
    android:authorities=...>
    <!-- 一時許可するパスの指定 -->
    <grant-uri-permission android:pathPrefix="/for_sharing/this_directory_only/" />
</provider>
```

pathPrefix属性の他にpath属性、pathPattern属性も使用できます。併用も可能です。
詳しくは[&lt;grant-uri-permission&gt; | Android Developers][0]を参照してください。

## 不適切な例

以下は、パス指定の3種の方法path属性、pathPrefix属性、およびpathPttern属性によってアクセス可能なリソースのパスを指定していますが、どれもすべてのリソースに対するアクセスを許可する設定になっていて、情報漏洩・改竄のリスクがあります。

```
<provider
  ... >
  <!-- すべてのリソースへのアクセスを許可していてLintが検知する例 -->
  <grant-uri-permission android:path="/" />
  <grant-uri-permission android:pathPrefix="/" />
  <grant-uri-permission android:pathPattern="/" />
  <!-- すべてのリソースへのアクセスを許可していてLintが検知しない例 -->
  <grant-uri-permission android:pathPattern=".*" />
</provider>
```
Lintは、grant-uri-permission属性のパス指定が “/” となっているものを検知すると、次のようなメッセージを出力します。

- Lint結果(Warning)  
  "Content provider shares everything; this is potentially dangerous."

このメッセージはexported属性をfalse（非公開）に設定していても出力されます。

## 外部リンク

-   [Content Provider | Android Developers][1]
-   [Content Provider のガイド][2]
-   [Android アプリのセキュア設計・セキュアコーディングガイド 4.3. Content Provider を作る・利用する][3]  

[0]: https://developer.android.com/guide/topics/manifest/grant-uri-permission-element
[1]: https://developer.android.com/reference/android/content/ContentProvider.html
[2]: https://developer.android.com/guide/topics/providers/content-providers.html
[3]: http://www.jssec.org/dl/android_securecoding/4_using_technology_in_a_safe_way.html#content-provider%E3%82%92%E4%BD%9C%E3%82%8B%E3%83%BB%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B
[4]: ExportedContentProvider.md


