# Using the result of check permission calls (UseCheckPermission)

## 警告されている問題点

呼び出し元へのパーミッション付与を「実質的に」確認していないため、重要情報の漏洩等が発生するリスクがあります。

## 対策のポイント

- コンポーネントの機能及び情報へのアクセスを許可する際は、呼び出し元のパーミッションを確認する

## 対策の具体例

パーミッションを確認するメソッドには挙動の異なる２種類があります。実装のスタイルに合わせて使い分けてください。

### パーミッションが付与されていないと例外が発生する

enforcePermission系のメソッドは、指定するパーミッションが付与されていない場合に例外を発生させます。
以下の例では、パーミッションが付与されていない呼び出し元に例外が返ります。  

```java
public void onCreate(Bundle savedInstanceState) {
    ...
    // 呼び出し元に位置情報パーミッションが付与されていなければ例外が発生します
    enforceCallingPermission(Manifest.permission.READ_CONTACTS,
                             "Requires permission " + Manifest.permission.READ_CONTACTS);

    // パーミッションが付与されていた場合の処理
    ...
}
```

### パーミッションの付与の状態を返却する

checkPermission系のメソッドは、指定するパーミッションの検査結果を値で返しますので、戻り値を確認して処理を適切に行ってください。
以下の例では、パーミッションが適切に付与されている場合に位置情報が返ります。

```java
    Intent userData = new Intent();
    userData.putExtra(...);
    ...
    
    if (checkCallingPermission(Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
        // 呼び出し元に位置情報のパーミッションが付与されている場合にのみ位置情報を付加する
        userData.putExtra("location", locationData);
    }
    
    setIntent(userData);
    setResult(RESULT_OK);
```

## 不適切な例

次の例では、checkCallingPermissionの戻り値に応じた処理をしていないため、呼び出し元のパーミッション付与の状態にかかわらず、常に位置情報を返してしまいます。

```java
    Intent userData = new Intent();
    userData.putExtra(...);
    ...
    // メソッドの結果を判定していない
    checkCallingPermission(Manifest.permission.ACCESS_COARSE_LOCATION);
    // 位置情報のパーミッションを付与されていないアプリにも位置情報を渡してしまう
    userData.putExtra("location", locationData);
    
    setIntent(userData);
    setResult(RESULT_OK);
```

__enforce__CallingPermissionメソッドの間違いで__check__CallingPermissionメソッドを使用すると、このようなことになります。

Lintは、checkCallingPermissionメソッドなどの戻り値が使用されていないことを検知すると、次のようなメッセージを出力します。

  - Lint 結果(Warning)  
    "The result of checkCallingPermission is not used; did you mean to call enforceCallingPermission."

## 外部リンク

  - [Context | Android Developers][1]  
  - [Androidアプリのセキュア設計・セキュアコーディングガイド 5.2.3.4. Permission の再委譲問題][2]  
  - [Felt, Adrienne Porter, et al. “Permission Re-Delegation: Attacks and Defenses.” USENIX Security Symposium. Vol. 30. 2011.][3]  

[1]: https://developer.android.com/reference/android/content/Context.html
[2]: http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#permission%E3%81%AE%E5%86%8D%E5%A7%94%E8%AD%B2%E5%95%8F%E9%A1%8C
[3]: https://www.usenix.org/event/sec11/tech/full_papers/Felt.pdf

