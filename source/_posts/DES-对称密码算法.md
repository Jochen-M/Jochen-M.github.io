---
title: DES 对称密码算法
date: 2017-06-12 23:52:37
tags:
    - Java
    - Safe
categories:
    - Java
---

*对称加密算法*是应用较早的加密算法，技术成熟。在对称加密算法中，数据发信方将明文（原始数据）和加密密钥一起经过特殊加密算法处理后，使其变成复杂的加密密文发送出去。收信方收到密文后，若想解读原文，则需要使用加密用过的密钥及相同算法的逆算法对密文进行解密，才能使其恢复成可读明文。在对称加密算法中，使用的密钥只有一个，发收信双方都使用这个密钥对数据进行加密和解密，这就要求解密方事先必须知道加密密钥。

Java代码实现：
```
import javax.crypto.*;
import java.io.*;
import java.security.Key;
import java.security.Security;

public class DES {
    public static void main(String[] args){
        DES des = new DES();
        des.run();
    }

    public void run(){
        Security.addProvider(new com.sun.crypto.provider.SunJCE());
        String Algorithm = "DES";

        try{
            String plainText;
            // 读取明文
            try (BufferedReader br = new BufferedReader(
                new InputStreamReader(new FileInputStream("plaintext.txt")))) {
                plainText = br.readLine();
            }

            // 生成密钥
            KeyGenerator keygen = KeyGenerator.getInstance(Algorithm);
            SecretKey deskey = keygen.generateKey();

            // 密钥材料存储于本地
            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("deskey.dat"));
            out.writeObject(deskey);
            out.close();

            // 加密
            System.out.println("明文：" + plainText);
            Cipher c1 = Cipher.getInstance(Algorithm);
            c1.init(Cipher.ENCRYPT_MODE, deskey);
            byte[] cipherByte = c1.doFinal(plainText.getBytes());
            System.out.println("摘要：" + byte2hex(cipherByte));

            // 将密文写入文件
            try (PrintWriter pw = new PrintWriter("cipherText.txt")){
                pw.print(byte2hex(cipherByte));
            }

            // 重建密钥
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("deskey.dat"));
            Key deskey2 = (Key)in.readObject();
            in.close();

            // 解密
            c1 = Cipher.getInstance(Algorithm);
            c1.init(Cipher.DECRYPT_MODE, deskey2);
            byte[] decryptoByte = c1.doFinal(cipherByte);
            System.out.println("摘要：" + byte2hex(decryptoByte));
            System.out.println("解密：" + new String(decryptoByte));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String byte2hex(byte[] ba){
        String strout = "";
        for(int i = 0; i < ba.length; i++){
            int j = ba[i] < 0? ba[i] + 256 : ba[i];
            String str = Integer.toHexString(j);
            while(str.length() < 2)
                str = '0' + str;
            if(i < ba.length - 1)
                strout += (str + "-");
            else
                strout += str;
        }
        return strout.toUpperCase();
    }
}
```