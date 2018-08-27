# Hardcoded value of android:debuggable in the manifest (HardcodedDebugMode)

## 警告されている問題点

デバッグ情報やデバッグ専用機能を含んだ状態でアプリをビルドし、設計上の内部情報や秘匿情報を公開してしまうリスクがあります。

## 対策のポイント

- 人為的ミスの原因となるため、設定ファイルでデバッグモード切替を行わないようにする

## 対策の具体例

以前のAndroid Studioでは、debuggable属性を切り替えてデバッグモードを変更していましたが、現在は明示されていなければ、ビルドタイプに応じて自動で適用されます。  
そのため、Android Studio利用時にはマニフェストのapplicationタグにdebuggable属性を設定しない方が利便性が高くなります。

```
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
              package="com.example.sample">

    <!-- ツールが変更できるようにdebuggable属性を記載しない -->
        <application
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:theme="@style/AppTheme">
        </application>

    </manifest>
```

[Build Variants][2]を利用して、デバッグと本番リリースのAPKを生成し分けるのが正しい手法とされます。これを利用すると「広告つきの無料版」 と 「広告なしの有料版」といったようにアプリの派生バージョンの管理も可能になります。

## 不適切な例

AndroidManifest.xmlにdebuggable属性が記述されていると、ビルドの種類にかかわらず常にその値を使用します。リリース時、この設定の変更を忘れると、デバッグ情報やデバッグ専用機能を含んだアプリを公開してしまう危険性があります。

```
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
              package="com.example.sample">

        <application
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:theme="@style/AppTheme"
            android:debuggable="true">　<!-- デバッグ/リリースなど用途に応じて逐一変更が必要 -->
        </application>

    </manifest>
```

Lintは、上の例のようにdebuggable属性の設定を検知すると、次のようなメッセージを出力します。

-   Lint出力(Error)  
    "Avoid hardcoding the debug mode; leaving it out allows debug and release builds to automatically assign one"

## 外部リンク

-   [AndroidManifest.xmlの属性値][1]

[1]: https://developer.android.com/guide/topics/manifest/application-element.html
[2]: https://developer.android.com/studio/build/build-variants.html?hl=ja


