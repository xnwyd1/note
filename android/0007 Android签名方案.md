
# 签名基础
- 摘要算法(密码散列函数)  
输出长度固定、结果确定、不可逆，MD5/SHA-1/SHA-256

- 加密算法  
   * 对称加密  
   加密和解密使用相同的秘钥
   * 非对称加密  
   加密和解密使用不同的秘钥，公钥和私钥，常用的有RSA算法

- 加密：用加密算法和公钥对明文加密，用私钥解密  
- 数字签名：私钥加密，公钥解密

签名过程如下图：  
![image](https://note.youdao.com/yws/api/personal/file/WEBea5adb0073c2c610d45574dd82089e78?method=download&shareKey=01b0bdab4cf70f56af2e50595dcc73a1)
<br/>
<br/>

- 数字证书  
目的是为了保证公钥的安全可信，数字证书一般包括：  
*证书的发布机构(Issuer)*  
*证书的有效期(Validity)*  
*消息发送方的公钥*  
*证书所有者(Subject)*  
*数字签名所使用的算法*  
*数字签名*  
如何保证公钥可信：先对公钥和证书信息进行消息摘要，再和数字签名里的摘要做对比，如果相同则表示公钥是可信的，如下图所示：

![image](https://note.youdao.com/yws/api/personal/file/WEBca27d65950482404c410c8c0950a5b0e?method=download&shareKey=189d45e2f904a2b28fe5b6e9a275b11b)
<br/>
<br/>
<br/>
<br/>

# APK Signature Scheme v2
Android 7.0开始使用V2签名，它与V1的区别是增加了一个APK Siging Block，如下所示:  

![image](https://note.youdao.com/yws/api/personal/file/WEB979b7203540e956200a08ffdb73d6e1c?method=download&shareKey=9ffc8b6055d0836d00ac76fe5ea34ae2)

---
APK会被分为以下4个区块：  
1. Contents of ZIP entries
2. APK Signing Block
3. ZIP Central Directory
4. ZIP End of Central Directory

APK Signing Block存储了其他3个区块的摘要信息，在验签的时候，会计算其他3个区块的摘要，与APK Signing Block的对比，不一样则验签失败。因此对其他3个区块的任意修改，都会导致验签不通过。
另外，V2验签不通过时，系统会尝试进行V1验签。  

- 7.1代码

 ApkSignatureSchemeV2Verifier.java

```java
private static X509Certificate[][] verify(RandomAccessFile apk)
            throws SignatureNotFoundException, SecurityException, IOException {
        SignatureInfo signatureInfo = findSignature(apk);//获取签名数据
        return verify(apk.getFD(), signatureInfo);
    }

```
signatureInfo中包括 整个APK Signing Block的内容，签名块的位置信息，核心目录块的的位置信息，目录结束标识块的位置及内容。

```java
private static X509Certificate[][] verify(
            FileDescriptor apkFileDescriptor,
            SignatureInfo signatureInfo) throws SecurityException {
        int signerCount = 0;
        Map<Integer, byte[]> contentDigests = new ArrayMap<>();
        List<X509Certificate[]> signerCerts = new ArrayList<>();
        CertificateFactory certFactory;
        try {
            certFactory = CertificateFactory.getInstance("X.509");
        } catch (CertificateException e) {
            throw new RuntimeException("Failed to obtain X.509 CertificateFactory", e);
        }
        ByteBuffer signers;
        try {
            signers = getLengthPrefixedSlice(signatureInfo.signatureBlock);
        } catch (IOException e) {
            throw new SecurityException("Failed to read list of signers", e);
        }
        while (signers.hasRemaining()) {
            signerCount++;
            try {
                ByteBuffer signer = getLengthPrefixedSlice(signers);
                X509Certificate[] certs = verifySigner(signer, contentDigests, certFactory);
                signerCerts.add(certs);
            } catch (IOException | BufferUnderflowException | SecurityException e) {
                throw new SecurityException(
                        "Failed to parse/verify signer #" + signerCount + " block",
                        e);
            }
        }

        if (signerCount < 1) {
            throw new SecurityException("No signers found");
        }

        if (contentDigests.isEmpty()) {
            throw new SecurityException("No content digests found");
        }

        verifyIntegrity(
                contentDigests,
                apkFileDescriptor,
                signatureInfo.apkSigningBlockOffset,
                signatureInfo.centralDirOffset,
                signatureInfo.eocdOffset,
                signatureInfo.eocd);

        return signerCerts.toArray(new X509Certificate[signerCerts.size()][]);
    }
```



---
- **ZIP文件格式参考**  

```
[local file header + file data + data descriptor]{1,n} + central directory + end of central directory record  
[文件头+文件数据+数据描述符]{此处可重复n次}+核心目录+目录结束标识
```
当压缩包中有多个文件时，就会有多个[文件头+文件数据+数据描述符]

**central directory**: 记录了压缩文件的目录信息，在这个数据区中每一条纪录对应在压缩源文件数据区中的一条数据  
**end of central directory**: 目录结束标识存在于整个归档包的结尾，用于标记压缩的目录数据的结束。每个压缩文件必须有且只有一个EOCD记录  

详细可参考[https://blog.csdn.net/a200710716/article/details/51644421](https://blog.csdn.net/a200710716/article/details/51644421)
<br/>
<br/>
<br/>

# 百富签名
百富签名会在apk尾部添加284个字节，导致V2验签失败，解决办法：修改验签时候的zip读取，让zip尾部往前移动284个字节。  
```diff patch
--- a/android/system/core/libziparchive/zip_archive.cc
+++ b/android/system/core/libziparchive/zip_archive.cc
@@ -268,7 +268,9 @@ static int32_t MapCentralDirectory0(const char* debug_file_name, ZipArchive* arc
   if (calculated_length != file_length) {
     ALOGW("Zip: %" PRId64 " extraneous bytes at the end of the central directory",
           static_cast<int64_t>(file_length - calculated_length));
-    return kInvalidFile;
+    //[FEATURE]-Del-BEGIN by (wangyingdong@paxsz.com), 2020/07/08
+    //return kInvalidFile;
+    //[FEATURE]-Del-END by (wangyingdong@paxsz.com), 2020/07/08
   }
 
   /*

```

```diff patch
--- a/android/frameworks/base/core/java/android/util/apk/ZipUtils.java
+++ b/android/frameworks/base/core/java/android/util/apk/ZipUtils.java
@@ -43,6 +43,8 @@ abstract class ZipUtils {
 
     private static final int UINT16_MAX_VALUE = 0xffff;
 
+    private static final int PAX_SIGNATURE_SIZE = 284;
+
     /**
      * Returns the ZIP End of Central Directory record of the provided ZIP file.
      *
@@ -130,6 +132,12 @@ abstract class ZipUtils {
         }
         // EoCD found
         buf.position(eocdOffsetInBuf);
+        
+        //paxsz@20200708: {
+        if (buf.capacity() - eocdOffsetInBuf == (PAX_SIGNATURE_SIZE + ZIP_EOCD_REC_MIN_SIZE)) {
+            buf.limit(eocdOffsetInBuf + ZIP_EOCD_REC_MIN_SIZE);
+        }
+        // } paxsz
         ByteBuffer eocd = buf.slice();
         eocd.order(ByteOrder.LITTLE_ENDIAN);
         return Pair.create(eocd, bufOffsetInFile + eocdOffsetInBuf);
@@ -170,6 +178,12 @@ abstract class ZipUtils {
                 if (actualCommentLength == expectedCommentLength) {
                     return eocdStartPos;
                 }
+                
+                //paxsz@20200708: {
+                else if (expectedCommentLength == PAX_SIGNATURE_SIZE) {
+                    return eocdStartPos;
+                }
+                // } paxsz
             }
         }
```