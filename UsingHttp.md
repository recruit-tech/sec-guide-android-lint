# Using HTTP instead of HTTPS (UsingHttp)

## 警告されている問題点

Gradleの配布元URLにhttpを使用しているため、中間者攻撃によりアプリを制御されてしまうリスクがあります。

## 対策のポイント

- Gradleラッパーの設定ファイルgradle-wrapper.propertiesのdistributionUrlはhttpsから始まるURLを指定する

## 対策の具体例

プロジェクトのビルドに使用するGradleのバイナリを指定する場合、gradle/wrapper/gradle-wrapper.propertiesのdistributionUrlはhttpsプロトコルで指定してください。

    distributionUrl = https\://services.gradle.org/distributions/gradle-3.3-all.zip

過去のAndroid Studioではhttpを使用していましたが、現在はhttpsがデフォルトになっています。http設定の既存のプロジェクトはhttps設定に修正してください。

## 不適切な例

以下は、gradleのURLをhttpで指定しているため、中間者攻撃を受けるリスクがあります。

    distributionUrl = http\://services.gradle.org/distributions/gradle-3.3-all.zip

Lintはgradle/wrapper/gradle-wrapper.propertiesのdistributionUrlにhttpのURL指定を検出すると、次のようなメッセージを出力をします。

-   Lint結果(Warning)  
    "Replace HTTP with HTTPS for better security; use https\\://services.gradle.org/distributions/gradle-3.3-all.zip"

## 外部リンク

-   [Android Plugin for Gradle Release Notes][1]
-   [Gradle Docs 4.3.1 「Chapter 6. The Gradle Wrapper」][2]


[1]: https://developer.android.com/studio/releases/gradle-plugin.html?hl=ja\#updating-gradle
[2]: https://docs.gradle.org/4.3.1/userguide/gradle\_wrapper.html