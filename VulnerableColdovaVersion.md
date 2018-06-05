# Vulnerable Cordova Version(VulnerableCordovaVersion)

## 警告されている問題点

問題のあるバージョンのCordovaが利用されているため、アプリを改変される可能性があります。

## 対策のポイント

- 問題が修正されたバージョン(4.1.1以上)のCordovaを利用する

## 対策の具体例

2016年7月11日以降、Google Playではバージョン4.1.1より前のApache Cordovaを使用するすべての新規アプリおよびアップデートの公開が禁止されています。
速やかにApache Cordova v.4.1.1 以降にアップグレードし、Cordova CLIでプラットフォーム追加の際は4.1.1以降を指定してください。
アップグレードの詳細は[Apache Cordova のウェブサイト][4]を参照してください。

```
    # cd <project root path>
    # cordova platform add android@latest
```

latestは最新バージョンが適用されます。

## 不適切な例

Cordova CLIでプラットフォームを追加する際に4.1.1より前のバージョンを指定しないでください。

```
    # cd <project root path>
    # cordova platform add android@3.6.4
```

Lintは使用しているCordovaのバージョンが4.1.1より前であることを検出すると、次のようなメッセージを出力をします。

- Lint結果  
    "You are using a vulnerable version of Cordova: 3.6.4"

## 外部リンク

  - [CVE-2015-5256][1]  
    URLホワイトリストの回避による不正なコンテンツの読み込みが可能になっていたという脆弱性
  - [CVE-2015-8320][2]  
    Java機能呼び出しに必要な擬似乱数が予測可能になっていたという脆弱性
  - [「Apache Cordova」を使ったハイブリッドアプリケーションの脆弱性に関する調査報告書][3]  
    CVE-2015-5256の詳細な解説

[1]: http://jvndb.jvn.jp/ja/contents/2015/JVNDB-2015-000187.html
[2]: http://jvndb.jvn.jp/ja/contents/2015/JVNDB-2015-006002.html
[3]: https://github.com/JPCERTCC/cordova
[4]: https://cordova.apache.org/docs/en/7.x/guide/platforms/android/upgrade.html

