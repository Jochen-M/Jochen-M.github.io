---
title: RSA 公钥密码算法
date: 2017-06-12 23:59:56
tags:
    - Java
    - Safe
categories:
    - Java
---

与数字签名不同，公钥密码算法中用公钥进行加密，用私钥进行解密。

Java代码实现：
```
import java.io.*;
import java.math.BigInteger;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

public class RSA {
    public static void main(String[] args){
        RSA rsa = new RSA();
        rsa.run();
    }

    public void run(){
        // 生成密钥对，如果已经生成过，本过程就可以跳过
        if((new File("RSAprikey.dat")).exists() == false){
            if(generatekey() == false){
                System.out.println("生成密钥对失败");
                return;
            }
        }

        try{
            // 读入公钥文件，重建公钥
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("RSApubkey.dat"));
            RSAPublicKey pubkey = (RSAPublicKey) in.readObject();
            in.close();

            // 从公钥中读出参数e和n
            BigInteger e = pubkey.getPublicExponent();
            BigInteger n = pubkey.getModulus();
            System.out.println("公钥参数e：" + e);
            System.out.println("公钥参数n：" + n);

            String plainText;
            // 读取明文
            try (BufferedReader br = new BufferedReader(
                    new InputStreamReader(new FileInputStream("plaintext.txt")))) {
                plainText = br.readLine();
            }

            // 给定明文编码成大整数
            byte plainbytes[] = plainText.getBytes("UTF8");
            BigInteger plainBI = new BigInteger(plainbytes);
            System.out.println("明文：" + plainText);
            System.out.println("明文编码：" + plainBI.toString());

            // 计算密文
            BigInteger cipherBI = plainBI.modPow(e, n);
            System.out.println("密文编码：" + cipherBI.toString());

            // 把密文保存到本地文件中
            String cipherText = cipherBI.toString();
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(
                    new FileOutputStream("RSACipher.dat")));
            out.write(cipherText, 0, cipherText.length());
            out.close();
        } catch(Exception e){
            e.printStackTrace();
            System.out.println("加密时发生错误！");
        }

        try{
            // 通过私钥文件重建私钥
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("RSAprikey.dat"));
            RSAPrivateKey prikey = (RSAPrivateKey) in.readObject();
            in.close();

            //获取私钥参数
            BigInteger d = prikey.getPrivateExponent();
            BigInteger n = prikey.getModulus();
            System.out.println("私钥参数d：" + d);
            System.out.println("私钥参数n：" + n);

            // 读出密文
            BufferedReader incipher = new BufferedReader(new InputStreamReader(
                    new FileInputStream("RSACipher.dat")));
            String cipherText = incipher.readLine();
            incipher.close();
            BigInteger cipherBI = new BigInteger(cipherText);

            // 解密
            BigInteger plainBI = cipherBI.modPow(d, n);
            System.out.println("解出明文的编码：" + plainBI);
            byte[] plainbytes = plainBI.toByteArray();
            System.out.print("解出的明文：");
            for(int i = 0; i < plainbytes.length; i++){
                System.out.print((char) plainbytes[i]);
            }
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    public boolean generatekey(){
        try{
            KeyPairGenerator keygen = KeyPairGenerator.getInstance("RSA");
            keygen.initialize(1024);
            KeyPair keys = keygen.generateKeyPair();
            PublicKey pubkey = keys.getPublic();
            PrivateKey prikey = keys.getPrivate();
            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("RSAprikey.dat"));
            out.writeObject(prikey);
            out.close();
            System.out.println("写入对象 prikeys ok");
            out = new ObjectOutputStream(new FileOutputStream("RSApubkey.dat"));
            out.writeObject(pubkey);
            out.close();
            System.out.println("写入对象 pubkeys ok");
            System.out.println("生成密钥对成功");
            return true;
        }catch(Exception e){
            e.printStackTrace();
            System.out.println("生成密钥对失败");
            return false;
        }
    }
}
```