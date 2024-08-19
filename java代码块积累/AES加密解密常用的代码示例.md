### AES

AES是 对称加密。

对称加密是指加密和解密使用相同的密钥的加密算法。

非对称加密是指加密和解密使用不同的密钥的加密算法。


### AES加密解密

- 加密模式，有 ECB模式 和 CBC 模式等等，ECB 不需要 iv偏移量，而CBC需要。
- 密钥，可以自定义。

- 填充方式，有 PKCS5 、PKCS7、NoPadding 。。

- 输出格式，可以有 16进制的 Hex ，或者是 Base64。



### 在线加密解密的网站：

https://tool.hiofd.com/aes-encrypt-online/

如果不确定，可以使用 在线加密解密的网站，判断是哪一种模式。



### Hutool依赖：



```
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.6</version>
</dependency>
```



### Hutool 的 AES：

Hutool 的 AES， 默认加密方式为ECB  ，默认的填充方式为 PKCS5。



### 加密，生成16进制的 Hex字符串：

```
    /**
     * 加密。生成16进制格式的 Hex
     *
     */
    public static String encryptHex(String password, String key) {

        byte[] keyBytes = key.getBytes();
        // 构建
        SymmetricCrypto symmetricCrypto = new SymmetricCrypto(SymmetricAlgorithm.AES, keyBytes);
        // 加密，生成16进制格式
        return symmetricCrypto.encryptHex(password);
    }
```



### 解密，根据16进制的Hex解密：

```
    /**
     * 解密。根据16进制的Hex解密
     */
    public static String decryptStr(String password, String key) {
        SymmetricCrypto symmetricCrypto = new SymmetricCrypto(SymmetricAlgorithm.AES, key.getBytes());
        password = symmetricCrypto.decryptStr(password);
        return password;
    }
```



### AES加密解密，16进制的 Hex 示例：

```
        String password = "";
        String key = "自定义key";

        String encryptHexPassword = encryptHex(password, key);
        System.out.println("加密:" + encryptHexPassword);

        String decryptStr = decryptStr(encryptHexPassword, key);
        System.out.println("解密:" + decryptStr);
```





### 加密，输出 base64编码的字符串。

```
    /**
     * 加密，输出 base64编码的字符串。
     *
     */
    public static String encryptBase64(String password, String key) {

        byte[] keyBytes = key.getBytes();
        // 构建
        SymmetricCrypto symmetricCrypto = new SymmetricCrypto(SymmetricAlgorithm.AES, keyBytes);
        // 加密
        byte[] encrypt = symmetricCrypto.encrypt(password);
        //使用BASE64对加密后的二进制数组进行编码
        return Base64.getEncoder().encodeToString(encrypt);

    }
```



### 解密，使用 base64编码的字符串解密

```
    /**
     * 解密，使用 base64编码的字符串解密。
     *
     */
    public static String decryptBase64(String password, String key) {
        SymmetricCrypto symmetricCrypto = new SymmetricCrypto(SymmetricAlgorithm.AES, key.getBytes());
        byte[] decrypt = symmetricCrypto.decrypt(password);
        return new String(decrypt);
    }
```





### AES加密解密，使用Base64示例：

```
        String encryptHexPassword = encryptBase64(password, key);
        System.out.println("加密:" + encryptHexPassword);

        String decryptBase64 = decryptBase64(encryptHexPassword, key);
        System.out.println("解密:" + decryptBase64);
```
