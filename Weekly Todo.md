# Weekly Todo

## 2021.5.10-5.16

- [x] Leetcode 7

- [ ] 设计模式21

- [ ] 计组10篇

- [ ] 计组pdf25pages

- [ ] [计组coursera两课](https://www.coursera.org/learn/jisuanji-zucheng/home/welcome)

- [ ] JUC两个点

  `CurrentHashMap`

  `ThreadLocal`

- [ ] Docker

- [ ] Spring循环依赖解决

- [ ] Java序列化与反序列化

- [ ] `copy on writearrayset`

- [ ] [简易RPC实现](https://github.com/wangzheng0822/codedesign/tree/master/com/xzg/cd/rpc)

- [ ] 幂等

- [ ] 对称性加密和非对称性加密 李永乐视频

  > Your understanding of "public keys encrypt, private keys decrypt" is correct... for data/message ENCRYPTION. For digital signatures, it is the reverse. With a digital signature, you are trying to prove that the document signed by you came from you. To do that, you need to use something that only YOU have: your private key.
  >
  > A digital signature in its simplest description is a hash (SHA1, MD5, etc.) of the data (file, message, etc.) that is subsequently encrypted with the signer's private key. Since that is something only the signer has (or should have) that is where the trust comes from. EVERYONE has (or should have) access to the signer's public key.
  >
  > So, to validate a digital signature, the recipient
  >
  > 1. Calculates a hash of the same data (file, message, etc.),
  > 2. Decrypts the digital signature using the sender's PUBLIC key, and
  > 3. Compares the 2 hash values.
  >
  > If they match, the signature is considered valid. If they don't match, it either means that a different key was used to sign it, or that the data has been altered (either intentionally or unintentionally).

  

- [x] [JWT introduction](https://jwt.io/introduction) 

- [x] RSA公钥加密为什么每次生成的都不一样 ：[原因](https://blog.csdn.net/guyongqiangx/article/details/74930951)

  + 公钥私钥加密前都需要对数据进行填充 
    + 私钥：填充的某个部分的值为固定的值
    + 公钥：填充的某个部分的值为伪随机数



+ [ ] `java spi`

+ [ ] `swagger`

+ [ ] `Disruptor` 下个月再看

+ [ ] cookie是存储在哪的

+ [ ] 线程池`shutdown`方法

+ [ ] gc roots的对象有哪些

+ [ ] 压缩指针

+ [x] [强引用 弱引用 软引用 WeakReference weakhashmap]( https://www.baeldung.com/java-weakhashmap)

  `weakreference`: 

  > When an object in memory is reachable only by Weak Reference Objects, it becomes automatically eligible for GC.

  `strong reference`

  > That is, whenever an object is referenced by a *chain of strong Reference Objects*, it cannot be garbage collected.

- [ ] CDN
- [ ] `google guava eventbus`
- [ ] 有限状态机
- [ ] `event bus`实现



