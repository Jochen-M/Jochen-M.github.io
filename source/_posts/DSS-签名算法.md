---
title: DSS 密钥生成与数字签名
date: 2017-06-13 00:04:52
tags:
    - Java
    - Safe
categories:
    - Algorithm
---

```
import java.io.*;
import java.security.*;

public class DSS {
    public static void main(String[] args){
        DSS dss = new DSS();
        dss.run();
    }

    public void run(){
        // 生成密钥对，如果已经生成过，本过程就可以跳过
        // 发送方的私钥文件prikey.dat要保存在本地，而公钥文件pubkey.dat则发送给接收方
        if((new File("prikey.dat")).exists() == false){
            if(generatekey() == false){
                System.out.println("生成密钥对失败");
                return;
            }
        }

        // 发送方从文件中读入私钥，对消息进行签名后保存在一个文件（msgwithsign.dat）中
        // 然后再把msgwithsign.dat发送给接收方，
        // 数字签名可以放进msgwithsign.dat文件中，也可以分别发送
        try{
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("prikey.dat"));
            PrivateKey prikey = (PrivateKey) in.readObject();
            in.close();

            // 读入要签名的信息
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(new FileInputStream("plaintext.txt")));
            String msgwithsign = br.readLine();

            // 用私钥对信息生成数字签名
            Signature sign1 = Signature.getInstance("DSA");
            sign1.initSign(prikey);
            sign1.update(msgwithsign.getBytes());
            byte[] signofmsg = sign1.sign();    // 对信息的数字签名
            System.out.println("数字签名：" + byte2hex(signofmsg));

            // 此处把信息和数字签名保存在一个文件中
            ObjectOutputStream out = new ObjectOutputStream(
                    new FileOutputStream("msgwithsign.dat"));
            out.writeObject(msgwithsign);
            out.writeObject(signofmsg);
            out.close();
            System.out.println("签名并生成文件成功");
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("签名时发生错误！");
        }

        // 接收方通过某种方式得到发送方的公钥和签名文件，
        // 然后用公钥对签名进行验证
        try{
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("pubkey.dat"));
            PublicKey pubkey = (PublicKey) in.readObject();
            in.close();
            in = new ObjectInputStream(new FileInputStream("msgwithsign.dat"));
            String msg = (String) in.readObject();
            byte[] signofmsg = (byte[]) in.readObject();
            in.close();
            Signature sign2 = Signature.getInstance("DSA");
            sign2.initVerify(pubkey);
            sign2.update(msg.getBytes());
            if(sign2.verify(signofmsg)){
                System.out.println("消息内容：" + msg);
                System.out.println("签名有效");
            }else{
                System.out.println("非正常签名");
            }
        } catch(Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 生成密钥对的函数
     * @return
     */
    public boolean generatekey(){
        try{
            KeyPairGenerator keygen = KeyPairGenerator.getInstance("DSA");
            keygen.initialize(512);
            KeyPair keys = keygen.generateKeyPair();
            PublicKey pubkey = keys.getPublic();
            PrivateKey prikey = keys.getPrivate();
            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("prikey.dat"));
            out.writeObject(prikey);
            out.close();
            System.out.println("写入对象 prikeys ok");
            out = new ObjectOutputStream(new FileOutputStream("pubkey.dat"));
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
