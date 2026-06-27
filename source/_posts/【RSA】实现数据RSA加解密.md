---
title: 【RSA】实现数据RSA加解密
date: 2023-05-16 12:30:02
tags:
  - RSA
category: 
  - 后端
---



## 数据安全加密简介

在项目的功能中，涉及密码的输入，都应该使用相应的[加密算法](https://so.csdn.net/so/search?q=加密算法&spm=1001.2101.3001.7020)来对传输的密码进行加密。加密的算法有很多，通常分为两种：[对称加密](https://so.csdn.net/so/search?q=对称加密&spm=1001.2101.3001.7020)和非对称加密

### 非对称加密算法

是指加密秘钥和解密秘钥不同。常见的非对称密钥加密算法：**RSA算法**，具有**数字签名和验证**的功能

### 其他

#### 数字签名

- 发送者不能否认发送的信息，接受者不能篡改接受的消息
- **确认了文件已签署这一事实**
- **确定了文件是真的这一事实**

## 加解密代码

- 从安全性进行考虑，在数据库的密码肯定是以**密文进行存储的**
- 在前端输入密码，向后端进行传输时，应该要**加密完后再向后端传输**
- 将从数据库取出来的密码密文和前端传来的密码密文**分别解码后再进行比较**

### 后端生成公私钥

**RSAUTil工具类**

```java
package com.cwh.javademo.utils;

import org.apache.tomcat.util.codec.binary.Base64;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import java.io.UnsupportedEncodingException;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

public class RSAUtil {

    private static final int DEFAULT_RSA_KEY_SIZE = 2048;

    private static final String KEY_ALGORITHM = "RSA";

    private static final String SIGNATURE_ALGORITHM = "MD5withRSA";

    /**
     * 生成RSA 公私钥,可选长度为1025,2048位.
     */
    public static Map<String, String> generateRsaKey(int keySize) {
        Map<String, String> result = new HashMap<>(2);
        try {
            KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(KEY_ALGORITHM);

            // 初始化密钥对生成器，密钥大小为1024 2048位
            keyPairGen.initialize(keySize, new SecureRandom());

            // 生成一个密钥对，保存在keyPair中
            KeyPair keyPair = keyPairGen.generateKeyPair();

            // 得到公钥字符串
            result.put("publicKey", new String(Base64.encodeBase64(keyPair.getPublic().getEncoded())));

            // 得到私钥字符串
            result.put("privateKey", new String(Base64.encodeBase64(keyPair.getPrivate().getEncoded())));

        } catch (GeneralSecurityException e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * RSA私钥解密
     *
     * @param str        解密字符串
     * @param privateKey 私钥
     * @return 明文
     */
    public static String decrypt(String str, String privateKey) {
        //64位解码加密后的字符串
        byte[] inputByte;
        String outStr = "";
        try {
            inputByte = Base64.decodeBase64(str.getBytes("UTF-8"));
            //base64编码的私钥
            byte[] decoded = Base64.decodeBase64(privateKey);
            RSAPrivateKey priKey = (RSAPrivateKey) KeyFactory.getInstance("RSA").generatePrivate(new PKCS8EncodedKeySpec(decoded));
            //RSA解密
            Cipher cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.DECRYPT_MODE, priKey);
            outStr = new String(cipher.doFinal(inputByte));
        } catch (UnsupportedEncodingException | NoSuchPaddingException | InvalidKeyException | IllegalBlockSizeException | BadPaddingException | InvalidKeySpecException | NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        return outStr;
    }

    /**
     * RSA公钥加密
     *
     * @param str       需要加密的字符串
     * @param publicKey 公钥
     * @return 密文
     * @throws Exception 加密过程中的异常信息
     */
    public static String encrypt(String str, String publicKey) throws Exception {
        //base64编码的公钥
        byte[] decoded = Base64.decodeBase64(publicKey);
        RSAPublicKey pubKey = (RSAPublicKey) KeyFactory.getInstance("RSA").generatePublic(new X509EncodedKeySpec(decoded));
        //RSA加密
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, pubKey);
        String outStr = Base64.encodeBase64String(cipher.doFinal(str.getBytes("UTF-8")));
        return outStr;
    }
}    
```

**测试**

```java
public static void main(String[] args) throws Exception {
    
    Map<String, String> result = generateRsaKey(DEFAULT_RSA_KEY_SIZE);
    String publicKey = result.get("publicKey");
    String privateKey = result.get("privateKey");
    System.out.println("公钥为：" + result.get("publicKey"));
    System.out.println("私钥为：" + result.get("privateKey"));
    
    // 加密(123 + RSA公钥) => [date+publicKey]
    String datePublicKeyStr = encrypt("123", publicKey);
    System.out.println(datePublicKeyStr);
    
    // 解密[date+privateKey]+ publicKey => 123
    System.out.println("数据为："+decrypt(datePublicKeyStr, privateKey));//123
}
```

**输出**

```shell
公钥为：MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAm49hhAFfZ6otIpOsClEKd71H4FKkEOXuyOAqt0ExPt4ECDicbZmyx5juCKDf91tAZ4z9twyU0Rm+0sZK7xQ5E30B4jgs4pPTljjG6m5QeDC/OOs/lsby2JbyKmWmDkDafrDnjXsKbqRnhkJsaadT2xab+s8q1cHNKsmQnlPf+2r4e6FrVEknFotlkzeR3VfFG7/HhhAVE/znzruMENY0P8oeVc87wA4WLyu+dn6rUpPzCslmajONMNR3xxc2J2rLZb3ePUAY8PQWaUgyGmYBEd92uyQOKx95vxJpCnYl75y/1tDs79wyqTSB4FySs8aulj6bWwtJC1q8HUoD6/5L5QIDAQAB
私钥为：MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCbj2GEAV9nqi0ik6wKUQp3vUfgUqQQ5e7I4Cq3QTE+3gQIOJxtmbLHmO4IoN/3W0BnjP23DJTRGb7SxkrvFDkTfQHiOCzik9OWOMbqblB4ML846z+WxvLYlvIqZaYOQNp+sOeNewpupGeGQmxpp1PbFpv6zyrVwc0qyZCeU9/7avh7oWtUSScWi2WTN5HdV8Ubv8eGEBUT/OfOu4wQ1jQ/yh5VzzvADhYvK752fqtSk/MKyWZqM40w1HfHFzYnastlvd49QBjw9BZpSDIaZgER33a7JA4rH3m/EmkKdiXvnL/W0Ozv3DKpNIHgXJKzxq6WPptbC0kLWrwdSgPr/kvlAgMBAAECggEASxbsGH9dITopLV6hFh3GcsRAdM0Pe0Syfe6PUAQ9FD6xLZK+F72wad6tUCbv1YQL07BgLEL7du/5h97F/yAA6SJXjW5WZEy9Pu9LPZBgcZP+SihseoiwYtKCNOr6PDkb/zm/nDC/eWcMvedEU7+8n64LPwdPgv1Y3wWLNJICNWbNVLVlEoIuhIniFnigL5lWDIjhE6Ualvym9cB2VSW+1V+OnhNg1ah+DKN6cDB9BEB92sqL9z9+f7uQDrb3S9njjIBC4uy59nhgxF8EkVnZYosOmsifmFO1HJcTiDS1VH6xseqaAtPG0nCyPWGAzC6z6wg8pid7LaWGp9XeXHZ1IQKBgQDVN6Dvmp2u3YpG7aWKvXPFYH3xF36WdSwUwKjbtGGGvlYDhmHEZMCksHM2IFhFwXIA0HapEnz6EdHoUEGqH594JEbnmxtnhNz0i0eJGM99qkcyFiyCqxvYVWABtQkDjmdMG4r9IArO5w3Xq1gYQd0WaQ+En7gpgtVIFWq3YLSxKQKBgQC6xhBIAAOebb8ro9+UQrc1/cz++msFm3rP0JWSuOKrIjhb/ReuMqkK4B2SiynLOXsS70OLU2VxfNkMcIF7648Gty9Z+ydZJlkfg62Ttl3aZt76PKnW7caWM9+bM/ypwyZViMI1+gdy+F2CTacNHMcD5TGNKuJWizG0HFuUORhwXQKBgCUwgcKpHk3M2HyMoO41I0dPEEiIB41ovJqWDB3eNZCSDGCrBMyDolJXcJEFTUBFgIQB2GCfF+tqRkmWDg4FXARRl4h4Nmx08TE6Rez0xeQuWiKzWWolPEMciRVjJUJYiU3uE+0YtKnoBTMT8NayTkTFaG6CiVW8O0VKbwWzOXEZAoGACEauggI+Js8GIZDpX1B1fdb5NnMyOtVg48SCXDYFFUA27xyP6BAmnWzA4rV37KFnardfbtULMbQuifaSRkNx2wJS/tG6NKEWYecb0efK0NquFriJbhSrMAysY9wx3fPfxvqAYJPrsJSA0D1QoawcxXdqcq7ryJnyYeC/zhmZk6ECgYAFQ2igNaIx+bYPH7aBMJK07M0GoOOs9P3RqukBtrh/4xHt2snmTib/lHVDut0PijTVBa5ZzkoKQwVxPEHv9nFWENfGHfhhu0nw09SVbCoPnOhPpKCUs5fzbULUmAhlrQ1EP8rm7yP0c1hbcw+bWVcCPmLdWwnzKeaHkt8mpk4yYg==
GOKZyWCzarols9yZVrsdtC8eH8E4uWAiVWlGL9pUE36kSP6XT9YOYj98LxOnEv0QycSbGKzeHf8Xq+kJuGrSevxyOEHCc/yWpUPVglRXkdkLpffIvU53Eg4KI2SUX/MdIbr2HJ1dMKFHLdR0f/JBC03Vne/9dyxmWmlaABGVc9MnEOqEySpDaoLjLL85w7rzHaA68MgNKKROGjxwXDgdcQxb0diM2cHe/NSbBJ4nQEY7ud38noFNXif/g6dXNB/xRS5Jo4B7ad4yqoFWeUFGnp+mvP/Y3R7fLKU/F09ZRSye/zTh1W8pdExcNVcHSNFoPFSHgYo/cVsrEJCT3LtsgQ==
123
```

### 前端加密数据上传

**jsencrypt介绍**

jsencrypt用于对**字符串**进行加密，且适用于非对称的加密，如：RSA等 要注意的是：加密的内容**只能是字符串类型**

首先将公钥放置在前端进行加密，加密后再向后端传输

**1、引入jsencrypt**

打开终端，输入如下命令

```js
npm install jsencrypt --dep
```

**2、在main.js中配置**

```js
// 引入全局签名加密法
import JsEncrypt from 'jsencrypt'
/**
 * 配置全局的加密方法
 * @param obj 需要加密的字符串
 */
Vue.prototype.$encruption = function (obj) {
  let encrypt = new JsEncrypt()
  encrypt.setPublicKey('公钥')  // 放置自己的公钥
  return encrypt.encrypt(obj)
}
```

**3、使用全局方法**

```js
this.$encruption('密码')
```
