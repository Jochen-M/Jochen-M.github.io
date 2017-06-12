---
title: MD5 消息摘要密码算法
date: 2017-06-13 00:03:12
tags:
    - Java
    - Safe
categories:
    - Java
---

```
import java.io.*;
import java.security.MessageDigest;

public class MD5 {
    public static void main(String[] args){
        try {
            // 读取明文
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(new FileInputStream("plaintext.txt")));
            String message1 = br.readLine();
            br.close();
            System.out.println("消息：" + message1);

            // 生成一个信息摘要类，定义消息摘要算法
            MessageDigest md1 = MessageDigest.getInstance("MD5");

            // 添加消息
            md1.update(message1.getBytes());

            // 计算摘要
            byte[] digest = md1.digest();
            System.out.println("摘要：" + byte2hex(digest));

            // 将摘要写入文件
            try (PrintWriter pw = new PrintWriter("digest.txt")){
                pw.print(byte2hex(digest));
            }

            // 接收方用相同的方法初始化，添加消息，然后比较摘要是否相同
            br = new BufferedReader(
                    new InputStreamReader(new FileInputStream("plaintext.txt")));
            String message2 = br.readLine();
            br.close();
            MessageDigest md2 = MessageDigest.getInstance("MD5");
            md2.update(message2.getBytes());
            System.out.println("验证结果：" + md2.isEqual(digest, md2.digest()));
        } catch (Exception e){
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