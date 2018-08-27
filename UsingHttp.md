# Using HTTP instead of HTTPS (UsingHttp)

## 警告されている問題点

Gradleの配布元URLにhttpを使用しているため、中間者攻撃[^1]によりアプリを制御されてしまうリスクがあります。

## 対策のポイント

- Gradleラッパーの設定ファイルgradle-wrapper.propertiesのdistributionUrlはhttpsから始まるURLを指定する

## 対策の具体例

プロジェクトのビルドに使用するGradleのバイナリを指定する場合、中間者攻撃[^1]を受けないようにするためgradle/wrapper/gradle-wrapper.propertiesのdistributionUrlはhttpsプロトコルで指定してください。

    distributionUrl = https\://services.gradle.org/distributions/gradle-3.3-all.zip

過去のAndroid Studioではhttpを使用していましたが、現在はhttpsがデフォルトになっています。http設定の既存のプロジェクトはhttps設定に修正してください。

## 不適切な例

以下は、gradleのURLをhttpで指定しているため、中間者攻撃[^1]を受けるリスクがあります。

    distributionUrl = http\://services.gradle.org/distributions/gradle-3.3-all.zip

Lintは、gradle/wrapper/gradle-wrapper.propertiesのdistributionUrlにhttpのURL指定を検出すると、次のようなメッセージを出力をします。

-   Lint出力(Warning)  
    "Replace HTTP with HTTPS for better security; use https\\://services.gradle.org/distributions/gradle-3.3-all.zip"

## 外部リンク

-   [Android Plugin for Gradle Release Notes][1]
-   [Gradle Docs 4.3.1 「Chapter 6. The Gradle Wrapper」][2]


[1]: https://developer.android.com/studio/releases/gradle-plugin.html?hl=ja\#updating-gradle
[2]: https://docs.gradle.org/4.3.1/userguide/gradle_wrapper.html

[^1]: dummy "中間者攻撃：通信している2人のユーザーの間に第三者が介在し、送信者と受信者の両方になりすまして、ユーザーが気付かないうちに通信を盗聴したり、制御したりすること"
