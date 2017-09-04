---
title: Vigenere 密码算法
date: 2017-06-12 23:46:05
tags:
    - Java
    - Safe
categories:
    - Algorithm
---

### Vigenere密码系统
多表代换密码系统的代表是Vigenere密码系统。Vigenere密码系统由法国密码学家Blaise de Vigenere于1858年提出的一种算法。

Java代码实现：
```
import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class Vigenere {
    private String zone;                                    // 明文和密文空间
    private String keyword;                                 // 密钥单词
    private List<Integer> key = new ArrayList<>();          // 密钥
    private List<String> plainText = new ArrayList<>();     // 明文
    private List<String> cipherText = new ArrayList<>();    // 密文
    private List<String> decryptText = new ArrayList<>();   // 破解的密文

    /**
     * 构造函数
     * 初始化明文和密文空间
     * 初始化密钥
     * @param k
     */
    public Vigenere(String k){
        StringBuffer sb = new StringBuffer();
        for(int i = 0; i <= 126; i++){
            sb.append((char)i);
        }
        this.zone = sb.toString();

        this.keyword = (k == null)? "computer" : k;
        for(int i = 0; i < this.keyword.length(); i++){
            this.key.add(new Integer(keyword.charAt(i)));
        }
    }

    /**
     * 从文件中读取明文
     * @throws IOException
     */
    public void readPlainText() throws IOException {
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(new FileInputStream("plaintext.txt")))){
            String line;
            while((line = br.readLine()) != null){
                this.plainText.add(line + "\n");
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 将密文写入文件
     */
    public void writeCipherText(){
        try (PrintWriter pw = new PrintWriter("cipherText.txt")){
            for(String ct : this.cipherText){
                pw.print(ct);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 将破解后的密文写入文件
     */
    public void writeDecryptText(){
        try (PrintWriter pw = new PrintWriter("decryptText.txt")){
            for(String dt : this.decryptText){
                pw.print(dt);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 加密
     */
    protected void encrypt(){
        int cycle = this.key.size();
        int totalLength = 0;
        for(String pt : this.plainText){
            StringBuffer ct = new StringBuffer();
            for(int j = 0; j < pt.length(); j++){
                ct.append((char)((pt.charAt(j) + totalLength % cycle) % 256));
                totalLength++;
            }
            this.cipherText.add(ct.toString());
        }
    }

    /**
     * 解密
     */
    public void decrypt(){
        int cycle = this.key.size();
        int totalLength = 0;
        for(String ct : this.cipherText){
            StringBuffer dt = new StringBuffer();
            for(int j = 0; j < ct.length(); j++){
                dt.append((char)((ct.charAt(j) - totalLength % cycle) % 256));
                totalLength++;
            }
            this.decryptText.add(dt.toString());
        }
    }

    public static void main(String[] args) throws IOException {
        Vigenere v = new Vigenere(null);
        v.readPlainText();
        v.encrypt();
        v.writeCipherText();
        v.decrypt();
        v.writeDecryptText();
    }
}
```
