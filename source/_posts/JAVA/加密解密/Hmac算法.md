---
title: Hmac算法
date: 2021-05-06 19:16:42
tags:
- Hash
- Hmac
---

# Hmac算法

存储用户的哈希口令时，要加盐存储，目的就在于抵御彩虹表攻击。

哈希算法：

```java
digest = hash(input)
```

正是因为相同的输入会产生相同的输出，我们加盐的目的就在于，使得输入有所变化：

```java
digest = hash(salt + input)
```

salt可以看作是一个额外的“认证码”，同样的输入，不同的认证码，会产生不同的输出。因此，要验证输出的哈希，必须同时提供“认证码”。

Hmac算法就是一种基于密钥的消息认证码算法，它的全称是Hash-based Message Authentication Code，是一种更安全的消息摘要算法。

Hmac算法总是和某种哈希算法配合起来用的。

```java
HmacMD5 ≈ md5(secure_random_key, input)
```

HmacMD5可以看作带有一个安全的key的MD5。使用HmacMD5而不是用MD5加salt，有如下好处：

- HmacMD5使用的key长度是64字节，更安全；
- Hmac是标准算法，同样适用于SHA-1等其他哈希算法；
- Hmac输出和原有的哈希算法长度一致。

为了保证安全，我们不会自己指定key，而是通过Java标准库的KeyGenerator生成一个安全的随机的key。下面是使用HmacMD5的代码：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        KeyGenerator keyGen = KeyGenerator.getInstance("HmacMD5");
        SecretKey key = keyGen.generateKey();
        // 打印随机生成的key:
        byte[] skey = key.getEncoded();
        System.out.println(new BigInteger(1, skey).toString(16));
        Mac mac = Mac.getInstance("HmacMD5");
        mac.init(key);
        mac.update("HelloWorld".getBytes("UTF-8"));
        byte[] result = mac.doFinal();
        System.out.println(new BigInteger(1, result).toString(16));
    }
}
```

和MD5相比，使用HmacMD5的步骤是：

1. 通过名称`HmacMD5`获取`KeyGenerator`实例；
2. 通过`KeyGenerator`创建一个`SecretKey`实例；
3. 通过名称`HmacMD5`获取`Mac`实例；
4. 用`SecretKey`初始化`Mac`实例；
5. 对`Mac`实例反复调用`update(byte[])`输入数据；
6. 调用`Mac`实例的`doFinal()`获取最终的哈希值。

数据库中的存储表变成了：

| username | secret_key (64 bytes) | password                         |
| :------- | :-------------------- | :------------------------------- |
| bob      | a8c06e05f92e...5e16   | 7e0387872a57c85ef6dddbaa12f376de |
| alice    | e6a343693985...f4be   | c1f929ac2552642b302e739bc0cdbaac |
| tim      | f27a973dfdc0...6003   | af57651c3a8a73303515804d4af43790 |

```java
public class Main {
    public static void main(String[] args) throws Exception {
        byte[] hkey = new byte[] { 106, 70, -110, 125, 39, -20, 52, 56, 85, 9, -19, -72, 52, -53, 52, -45, -6, 119, -63,
                30, 20, -83, -28, 77, 98, 109, -32, -76, 121, -106, 0, -74, -107, -114, -45, 104, -104, -8, 2, 121, 6,
                97, -18, -13, -63, -30, -125, -103, -80, -46, 113, -14, 68, 32, -46, 101, -116, -104, -81, -108, 122,
                89, -106, -109 };

        SecretKey key = new SecretKeySpec(hkey, "HmacMD5");
        Mac mac = Mac.getInstance("HmacMD5");
        mac.init(key);
        mac.update("HelloWorld".getBytes("UTF-8"));
        byte[] result = mac.doFinal();
        System.out.println(Arrays.toString(result));
        // [126, 59, 37, 63, 73, 90, 111, -96, -77, 15, 82, -74, 122, -55, -67, 54]
    }
}
```

