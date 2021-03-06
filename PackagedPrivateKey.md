# Packaged private key (PackagedPrivateKey)

## 警告されている問題点

アプリケーションパッケージ(APK)に秘密鍵を格納すると、それを悪意あるユーザーやマルウェアによって取り出され、秘密鍵で署名した情報が改竄されたり、なりすましのリスクがあります。

## 対策のポイント

- アプリのAPK(ソースコードやリソースなど)に秘密鍵を格納しない

## 対策の具体例

秘密鍵をAPKで保持しなければならない設計には問題があると考えられます。アプリやサービスの仕様・設計を見直してください。
万一、秘密鍵のAPKへの埋め込みが必要な場合は、セキュア設計の専門家に相談してください。

## 不適切な例

秘密鍵をアセット(assets)やリソース(res)ディレクトリに配置したり、あるいはソースコードに直接埋め込むことができます。
Lintは、このうちassetsとresディレクトリに配置された秘密鍵(らしいと思われる)以下のファイルを検知します。

- ファイル名が.pemまたは.keyで終わる
- 最初の行が “—”で開始され、“PRIVATE KEY”という文字列を含んでいる

Lintは、assetsあるいはresディレクトリに上で述べた条件に合致するファイルを検知すると、次のようなメッセージを出力します。

-   Lintの主力結果(Error)  
    "The \`(filename)\` file seems to be a private key file. Please make sure not to embed this in your APK file."

ソースコードに埋め込まれた秘密鍵についてはメッセージを出力しませんが、リバースエンジニアリングを用いて秘密鍵を取り出すことは可能なので、同様のリスクがあります。
また、Lintの検知条件に合致しないように秘密鍵を格納しても、攻撃者が解析する際の妨げにはなりません。

