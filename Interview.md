# 计算机基础

## 计算机网络

### 传输层

<img src="https://gitee.com/seazean/images/raw/master/JavaSE/三次握手.png" style="zoom:50%;" />

<img src="https://gitee.com/seazean/images/raw/master/JavaSE/四次挥手.png" alt="四次挥手" style="zoom: 67%;" />



* **不携带数据的ACK不会超时重传**



* **为什么要进行三次握手？**

  **原因一**：为了告诉双方序列号。tcp是全双工可靠的传输协议。全双工意味着双方能够同时向对方发送数据。可靠意味着我发送的数据必须确认对方完整收到了。tcp是通过序列号来保证这两种性质的，**三次握手就是互换序列号**的一次过程。

  1. A 发送同步信号**SYN** + **A's Initial sequence number**
  2. B 确认收到A的同步信号，并记录 A's ISN 到本地，命名 **B's ACK sequence number**
  3. B发送同步信号**SYN** + **B's Initial sequence number**
  4. A确认收到B的同步信号，并记录 B's ISN 到本地，命名 **A's ACK sequence number**

  * 很显然2和3 这两个步骤可以合并，**只需要三次握手，**可以提高连接的速度与效率。

  **原因二**：第三次握手是为了防止失效的连接请求到达服务器，让服务器错误打开连接。客户端发送的连接请求如果在网络中滞留，那么就会隔很长一段时间才能收到服务器端发回的连接确认。客户端等待一个超时重传时间之后，就会重新请求连接。但是这个滞留的连接请求最后还是会到达服务器，如果不进行三次握手，那么服务器就会打开两个连接。如果有第三次握手，客户端会忽略服务器之后发送的对滞留连接请求的连接确认，不进行第三次握手，因此就不会再次打开连接



* **三次握手的第三次握手发送ACK能携带数据吗？**

  答：可以。第三次握手，在客户端发送完ACK报文后，就进入ESTABLISHED状态，当服务器收到这个，服务器变为ESTABLISHED状态，可以直接处理携带的数据。





* **为什么TCP4次挥手时等待为2MSL？**

  **原因一：**A发送完释放连接的应答并不知道B是否接到自己的ACK，所以有两种情况
  1）如果B没有收到自己的ACK，会超时重传FIN，那么A再次接到重传的FIN，会再次发送ACK
  2）如果B收到自己的ACK，也不会再发任何消息，包括ACK
  无论是1还是2，A都需要等待，要取这两种情况等待时间的最大值，以应对最坏的情况发生，这个最坏情况是：去向ACK消息最大存活时间（MSL) + 来向FIN消息的最大存活时间(MSL)，
  这就是**2MSL( Maximum Segment Life)**。等待2MSL时间，A就可以放心地释放TCP占用的资源、端口号，此时可以使用该端口号连接任何服务器。

  **原因二：**等待一段时间是为了让本次连接持续时间内所产生的报文都从网络中消失，否则存活在网络里的老的TCP报文可能与新TCP连接报文产生冲突（比如连接同一个端口），为避免此种情况，需要耐心等待网络老的TCP连接的活跃报文全部消失，2MSL时间可以满足这个需求（尽管非常保守）



* **为什么连接的时候是三次握手，关闭的时候却是四次握手？**

  答：因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。





***



## HTTP

* **对称加密和非对称加密**

  对称加密：加密和解密使用同一个秘钥，把密钥转发给需要发送数据的客户机，中途会被拦截（类似于把带锁的箱子和钥匙给别人，对方打开箱子放入数据，上锁后发送）

  * 优点：运算速度快
  * 缺点：无法安全的将密钥传输给通信方

  非对称加密：加密和解密使用不同的秘钥，一把作为公开的公钥，另一把作为私钥，公钥公开给任何人（类似于把锁和箱子给别人，对方打开箱子放入数据，上锁后发送）

  * 优点：可以更安全地将公开密钥传输给通信发送方
  * 缺点：运算速度慢

* **使用对称加密和非对称加密的方式传送数据**
  
  * 使用非对称密钥加密方式，传输对称密钥加密方式所需要的 Secret Key，从而保证安全性;
  * 获取到 Secret Key 后，再使用对称密钥加密方式进行通信，从而保证效率
  
  思想：锁上加锁
  
  
  
* **HTTP1.1新特性**

  默认是长连接、支持流水线、支持同时打开多个 TCP 连接、支持虚拟主机、支持分块传输编码
  新增状态码 100、新增缓存处理指令 max-age

  

* **Get和POST比较**

  作用：GET 用于获取资源，而 POST 用于传输实体主体
  
  参数：GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体中。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看
  
  安全：安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的。GET方法是安全的，而POST不是，因为 POST 的目的是传送实体主体内容
  
  * 安全的方法除了 GET 之外还有：HEAD、OPTIONS
  * 不安全的方法除了 POST 之外还有 PUT、DELETE
  
  幂等性：同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。所有的安全方法也都是幂等的。在正确实现条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，POST 方法不是
  
  可缓存：如果要对响应进行缓存，需要满足以下条件
  
  * 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的
  * 响应报文的状态码是可缓存的，包括：200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501
  * 响应报文的 Cache-Control 首部字段没有指定不进行缓存



***



## 操作系统

### 进程线程

* 操作系统？

  控制和管理计算机硬件与软件资源的，并合理的组织和调度计算机工作的程序

* 什么是系统调度？

  在用户程序中调用操作系统提供的核心态级别的子功能，结合用户态和核心态区别回答，一般使用陷入（trap），按调用功能分为：设备管理、文件管理、进程控制、进程通信、内存管理，

* 进程线程？

  进程：程序是静止的，进程是程序的一次执行过程，是系统资源分配的基本单位

  线程：轻量级进程，是CPU的执行单元，是独立调度的最小单位，只拥有一点必不可少的资源

  关系：一个进程中包含多个线程，线程之间共享进程的资源，进程之间是相互独立

  区别：资源、并发、切换、通信、

* 进程通信的方式？

  同一台计算机的进程通信称为 IPC（Inter-process communication）
  不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如 HTTP














