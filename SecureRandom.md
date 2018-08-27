# Using a fixed seed with SecureRandom (SecureRandom)

## 警告されている問題点

疑似乱数生成器(SecureRandom)のシードに不適切な値を使用しているため、乱数の生成が予測される可能性が高くなります。

## 対策のポイント

- SecureRandomに不適切なシードを設定しない

## 対策の具体例

### シードを設定しない

デフォルトで推測困難なシードが設定されるため、特別な理由がない限りSecureRandomにシードを設定する必要はありません。
コンストラクタでシードを設定せず、setSeedメソッドも使用しないでください。

```java
    // 引数のないコンストラクタを使用する
    Random r = new SecureRandom();

    // r.setSeed()を使用しない
    ...
    byte bytes[]  = new byte[16];
    r.nextBytes(bytes);	// 乱数のバイト列を取得
```

Note: Android 8.0 (API 26)以降ではシードに依らず乱数を生成するため、setSeedメソッドが従来通りに機能しません。

## 不適切な例

### SecureRandomのコンストラクタで定数や時刻のシードを設定する

下の例のように、コンストラクタにバイト列を渡してシードを設定すると、予測可能な乱数が生成されることになります。

```java
    Random r = new SecureRandom(new byte[] { 1, 2, 3} ); // 毎回同じ乱数が生成される
```

```java
    Random r = new SecureRandom(System.currentTimeMillis()); // 時刻を利用していると予測されると容易にシードが推定される
```

ただし、Lintはコンストラクタでの不適切なシードの設定を検知することができません。

### setSeedメソッドで定数や時刻のシードを設定する

setSeedメソッドで定数や時刻をシードに設定した場合も、生成される乱数の性質は上記の例と同様になります。
定数値のシードは常に同じ擬似乱数列を生成し、時刻のシードは容易に推測が可能です。

```java
    Random r = new SecureRandom();

    // 定数のシードを与えている
    r.setSeed(new byte[] {
        0x04, 0xFD, 0xA8, 0x36, 0x9A, 0x75, 0x0A, 0xEC,
        0x77, 0x1B, 0x8A, 0x64, 0x3D, 0x19, 0xDE, 0xAD,
        0xA2, 0xF9, 0x66, 0x64, 0x0A, 0x3B, 0xBE, 0xEF,
        0x6E, 0x69, 0x42, 0x33, 0x0A, 0x91, 0xF7, 0x57
    });
```

```java
    Random r = new SecureRandom();

    // 現在時刻 (ミリ秒単位) をシードに与えている
    r.setSeed(System.currentTimeMillis());
```

Lintは、前者の例のようにシードに定数値の設定を検知すると、次のようなメッセージを出力します。

- Lint出力(Warning)  
   " Do not call \`setSeed()\` on a \`SecureRandom\` with a fixed seed: it is not secure. Use \`getSeed()\`."
    ただし、クラス定数やインスタンス定数をシードに設定した場合には検知できません。

Lintは、後者の例のようにシードに現在時刻の設定を検知すると、次のようなメッセージを出力します。

-   Lint出力(Warning)  
    "It is dangerous to seed \`SecureRandom\` with the current time because that value is more predictable to an attacker than the default seed."
    System.nanoTime()をシードに設定した場合も同様に検知します。

## 外部リンク

- [SecureRandom | Android Developers][1]  
- [MSC63-J. Ensure that SecureRandom is properly seeded][2]  
- [RFC 4086: Randomness Requirements for Security][3]  
- [NIST SP 800-22 Rev. 1a][4]    


[1]:https://developer.android.com/reference/java/security/SecureRandom.html
[2]:https://www.securecoding.cert.org/confluence/display/java/MSC63-J.+Ensure+that+SecureRandom+is+properly+seeded
[3]:https://tools.ietf.org/html/rfc4086
[4]:https://csrc.nist.gov/publications/detail/sp/800-22/rev-1a/final

