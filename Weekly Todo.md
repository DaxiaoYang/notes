# Weekly Todo

## 2021.5.10-5.16

- [x] Leetcode 7

- [ ] 设计模式21

- [ ] 计组10篇

- [ ] 计组pdf25pages

- [ ] JUC两个点

  `CurrentHashMap`

  `ThreadLocal`

- [ ] Docker

- [ ] Spring循环依赖解决

- [ ] Java序列化与反序列化

- [x] `copy on writearrayset`

  `add`方法时加锁的 往数组中添加一个元素的时候 先创建一个新的数组 往新数组中添加元素 最后替换引用 使得数组指针指向新对象 然后解锁

  `get`方法直接获取当前对象的数组

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



+ [x] `java spi`

  [spi in sprintboot-starter](https://juejin.cn/post/6844903890173837326)

  `serviceLoader`通过反射创建实例

+ [ ] `swagger`

+ [ ] `Disruptor` 下个月再看

+ [x] cookie是存储在哪的

  存储在文件中

+ [ ] 线程池`shutdown`方法

+ [x] gc roots的对象有哪些

  + 与栈帧相关的各种对象

  + 当前被加载的Java类

  + Java 类的引用类型静态变量。

  + 运行时常量池里的引用类型常量（String 或 Class 类型）。

  + JVM 内部数据结构的一些引用，比如 sun.jvm.hotspot.memory.Universe 类。

  + 用于同步的监控对象，比如调用了对象的 wait() 方法。

  + JNI handles，包括 global handles 和 local handles

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





## 2021.5.17-5.23

+ [x] leetcode7+

+ [x] design pattern 21+

+ [ ] computer organization 14

+ [x] 复习一下最小接口原则

+ [ ] 敏捷开发

+ [x] Volatile to reference type

  `volatile` acts either on a primitive variable or on a reference variable. There’s no relation between `volatile` and the object referred to by the (reference) variable.

+ [ ] `simple dateformat` 线程安全问题 看下源码

+ [ ] uml图

+ [ ] 线程被中断后执行

+ [ ] https://redis.io/topics/persistence

+ [ ] 字符串匹配算法

+ [x] 看下模板和策略模式

+ [ ] 看下lambda那块的内容

+ [ ] `protocol buffer`

+ [ ] `? extends ? super`复习一下这个

