# Android Lint 利用ガイドライン

## Android Lintとは

Android LintはAndroid Development Tools16から導入された、ソースチェックツールです。
Androidアプリケーションのソースから潜在的なものも含めて問題を検出し、警告してくれます。
このコンテンツでは、その中からセキュリティに分類されるものについて具体的な対策を示しながら解説をします。

本コンテンツは、主に2018年2月頃の情報に基づいて記載しております。
それ以降に発表された情報については随時更新していく予定ですが、最新の情報は[Android Developers](https://developer.android.com/)などをご参照ください。

## Android Lint のセキュリティメッセージ

### A

 [addJavascriptInterface Called](AddJavascriptInterface.md)
 [AllowBackup/FullBackupContent Problems](AllowBackup.md)

### C

 [Call to SSLCertificateSocketFactory.getInsecure()](SSLCertificateSocketFactoryGetInsecure.md)
 [Cipher.getInstance with ECB](GetInstance.md)
 [Code contains easter egg](EasterEgg.md)
 [Code might contain an auth leak](AuthLeak.md)
 [Content provider does not require permission](ExportedContentProvider.md)
 [Content provider shares everything](GrantAllUris.md)

### E

 [Exported service does not require permission](ExportedService.md)

### F

 [File.setReadable() used to make file world-readable](SetWorldReadable.md)
 [File.setWritable() used to make file world-writable](SetWorldWritable.md)

### H

 [Hardcoded value of android:debuggable in the manifest](HardcodedDebugMode.md)
 [Hardware Id Usage](HardwareIds.md)

### I

 [Incorrect constant](WrongConstant.md)
 [Insecure call to SSLCertificateSocketFactory.createSocket()](SSLCertificateSocketFactoryCreateSocket.md)
 [Insecure HostnameVerifier #1](AllowAllHostnameVerifier.md)
 [Insecure HostnameVerifier #2](BadHostnameVerifier.md)
 [Insecure TLS/SSL trust manager](TrustAllX509TrustManager.md)
 [Invalid Permission Attribute](InvalidPermission.md)

### L

 [load used to dynamically load code](UnsafeDynamicallyLoadedCode.md)

### M

 [Missing @JavascriptInterface on methods](JavascriptInterface.md)

### N

 [Native code outside library directory](UnsafeNativeCodeLocation.md)

### O

 [openFileOutput() or similar call passing MODE_WORLD_READABLE](WorldReadableFiles.md)
 [openFileOutput() or similar call passing MODE_WORLD_WRITEABLE](WorldWritableFiles.md)

### P

 [Packaged private key](PackagedPrivateKey.md)
 [Potential Multiple Certificate Exploit](PackageManagerGetSignatures.md)
 [PreferenceActivity should not be exported](ExportedPreferenceActivity.md)

### R

 [Receiver does not require permission](ExportedReceiver.md)

### S

 [signatureOrSystem permissions declared](SignatureOrSystemPermissions.md)

### U

 [Unprotected SMS BroadcastReceiver](UnprotectedSMSBroadcastReceiver.md)
 [Unsafe Protected BroadcastReceiver](UnsafeProtectedBroadcastReceiver.md)
 [Using a fixed seed with SecureRandom](SecureRandom.md)
 [Using HTTP instead of HTTPS](UsingHttp.md)
 [Using setJavaScriptEnabled](SetJavascriptEnabled.md)
 [Using the result of check permission calls](UseCheckPermission.md)

### V

 [Vulnerable Cordova Version](VulnerableCordovaVersion.md)

### W

 [Weak RNG](TrulyRandom.md)
