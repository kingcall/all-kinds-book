### 常见的加解密算法



1.Base64 <可逆不安全！>



```
/**
 * Base64不是安全领域下的加解密算法，只是一个编码算法，
 * 通常用于把二进制数据编码为可写的字符形式的数据，
 * 特别适合在http，mime协议下的网络快速传输数据。
 * UTF-8和GBK中文的Base64编码结果是不同的。
 * 采用Base64编码不仅比较简短，同时也具有不可读性，
 * 即所编码的数据不会被人用肉眼所直接看到，但这种方式很初级，
 * 很简单。Base64可以对图片文件进行编码传输。
 */
```

2.MD5 <不可逆！>



```
/**
 * 散列算法（签名算法）有：MD5、SHA1、HMAC
 * 用途：主要用于验证，防止信息被修。具体用途如：文件校验、数字签名、鉴权协议
 */
```

3.AES <可逆！>



```
/**
 * 对称性加密算法有：AES、DES、3DES
 * 用途：对称加密算法用来对敏感数据等信息进行加密
 *
 * AES 是一种可逆加密算法，对用户的敏感信息加密处理 \
 * 对原始数据进行AES加密后，再进行Base64编码转化；
 */
```

------



```
一、MD5
```



```
public class MD5Enscript extends UDF {

    //md5+盐(salt)非对称加密
    private static final String SALT = "KGC#kb08@20200511@xxq@chy#=";

    public String evaluate(String pwd) throws IOException {
        return DigestUtils.md5Hex(SALT+pwd+SALT);
    }

}
```



```
二、Base64Encrypt
```



```
public class Base64Encrypt extends UDF {

    public String evaluate(String str) throws UnsupportedEncodingException {
        return Base64.encodeBase64String(str.getBytes("UTF-8"));
    }

}
```



```
public class Base64Decrypt extends UDF {

    public String evaluate(String str) throws UnsupportedEncodingException {
        return new String(Base64.decodeBase64(str));
    }
}
```



```
三、AESSuper
```



```
public class AESSuper {

    private static final String SALT = "#./*KGC#kb08\\u1009889";
    private static final String ALGORITHM = "AES";
    private static final String CHAR_SET = "UTF-8";
    private static SecretKeySpec sks;
    private static Cipher cipher;

    static {
        try {
            KeyGenerator generator = KeyGenerator.getInstance(ALGORITHM);
            generator.init(128,new SecureRandom(SALT.getBytes()));
            SecretKey secretKey = generator.generateKey();
            byte[] encoded = secretKey.getEncoded();
            sks = new SecretKeySpec(encoded,ALGORITHM);
            cipher = Cipher.getInstance(ALGORITHM);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static byte[] encrypt(String src) throws Exception{

        cipher.init(Cipher.ENCRYPT_MODE,sks);
        return cipher.doFinal(src.getBytes(CHAR_SET));
    }

    public static byte[] decrypt(byte[] src) throws Exception{
        cipher.init(Cipher.DECRYPT_MODE,sks);
        return cipher.doFinal(src);
    }

}
```



```
public class AESEncrypt extends UDF {

    public String evaluate(String src) throws Exception {

        return Base64.encodeBase64String(AESSuper.encrypt(src));
    }

}
```



```
public class AESDecrypt extends UDF {

    public String evaluate(String src) throws Exception {
        return new String(AESSuper.decrypt(Base64.decodeBase64(src.getBytes())));
    }

}
```





### Hive 自带的加解密函数

```sql
 select  base64(cast('abcd' as binary));
```

![image-20201230093725059](/Users/liuwenqiang/Library/Application%20Support/typora-user-images/image-20201230093725059.png)

```sql
select unbase64('YWJjZA==');
```

![image-20201230093808541](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230093808541.png)