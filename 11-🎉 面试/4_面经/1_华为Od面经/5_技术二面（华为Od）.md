---
文章标题: "[[5_技术二面（华为Od）]]" 
文章作者: Dakkk
文章概要: |
  本文详细记录了华为OD技术二面经验，涵盖了自我介绍、项目经验深入交流、Java后端经典八股文面试（多线程容器、Spring Boot、MyBatis缓存、Redis数据结构与应用），以及一道中等难度算法题（最长回文子串）的考察，为求职者提供了宝贵参考。
文章标签:
- "Java面试"
- "华为OD"
- "后端开发"
- "八股文"
- "算法"
- "Spring Boot"
- "MyBatis"
- "Redis"
- "多线程"
相关文章:
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[1_Java学习路线]]"
- "[[1_Part1]]"
- "[[10_获取某个标签下的文章列表——分页接口开发]]"
文章分类: "🎉 面试"
文章路径: "11-🎉 面试/4_面经/1_华为Od面经/5_技术二面（华为Od）.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

主要内容：
- 自我介绍
- 介绍工作项目（简历上的项目）
- 常规八股
- 手撕算法

## 自我介绍

之前参考（隔壁）星球大佬`我就是贺生啊`的[文章)](https://wx.zsxq.com/dweb2/index/topic_detail/584542884844544)写过一个模版，主要有以下内容
- 个人介绍（学历、性格、爱好、**突出喜欢专研编程相关的技术**）
- 个人成长（校招说学习经历，社招说工作经历，**还是要突出编程相关的经历**）
	- 补充社招的内容：通过自己的实际项目来说成长！！
## 1 介绍工作项目（主要）

**社招的兄弟，脸皮厚点，把自己写的项目当作公司的项目来说就好了**

主要问的内容如下，问的比较多就不说了：
- 先说一遍项目的的流程（自己做的流程肯定知道吧，不过专注于后端/前端，别前后端都说）
- 再说项目中遇到的问题，以及如何解决的（不细说了）
- 最后说项目自己做了那些优化（不细说了）

个人思考/建议：
- 做项目的时候，思考一下别人为什么用这个东西，有什么好处，可以用其他自己知道的吗？
- 遇到bug的时候，记录下来，回顾一下
- 也是在做项目的时候，记录自己感觉可以优化的点，然后在做项目的时候进行优化，或者做完项目的时候再优化
- **关键点：** 上述的过程都要做文档沉淀！！！以后问到了就会回答了

## 常规八股（这次问的少，不过比较深入）

**1. 存储容器是线程安全的？**`（上次问过了，这次准确答出）`

同步容器类：使用了synchronized
- Vector
- HashTable

并发容器类：
- ConcurrentHashMap：分段
- CopyOnWriteArrayList：写时复制
- CopyOnWriteArraySet：写时复制

Queue
- ConcurrentLinkedQueue：使用非阻塞式的方式实现的基于链接节点的无界的线程安全队列，性能非常好。（java.util.concurrent.BlockingQueue 接口代表了线程安全的队列）
- ArrayBlockingQueue：基于数组的有界阻塞队列
- LinkedBlockingQueue：基于链表的有界阻塞队列
- PriorityBolckingQueue：表示优先级的无界阻塞队列，即该阻塞队列中的元素可自动排序。默认情况下，元素采用自然升序
- DelayQueue：一种延时获取元素的无界阻塞队列
- SynchronousQueue：不存储元素的阻塞队列。每个put操作必须等待一个take操作，否则不能继续添加元素；内部起始没有任何一个元素，容量是0

Deque接口定义了双向队列，双向队列允许在队列头和尾部继续入队出队操作
- ArrayDeque：基于数组的双向非阻塞队列
- LinkedBlockingDeque：基于链表的双向阻塞队列

Sorted容器
- ConcurrentSkipListMap：是TreeMap的线程安全版本
- ConcurrentSkipListSet：是TreeSet的线程安全版本

**2. Springboot你了解吗？经常用哪些注解呢？**`按照自己的理解说了`

- 先回答了什么是Springboot
	- Spring Boot 已经建立在现有 spring 框架之上。使用 spring 启动，我们避免了之前我们必须做的所有样板代码和配置。因此，Spring Boot 可以 帮助我们以最少的工作量，更加健壮地使用现有的 Spring 功能。

- 又回答了Springboot的优点
	- 减少开发，测试时间和努力。
	- 使用 JavaConfig 有助于避免使用 XML。
	- 避免大量的 Maven 导入和各种版本冲突。
	- 通过提供默认值快速开始开发。
	- 没有单独的 Web 服务器需要。这意味着你不再需要启动 Tomcat，Glassfish 或其他任何东 西。
	- 需要更少的配置 因为没有 web.xml 文件

- 最后回答了一些常用注解
	- 大家都用过的，就不多说了！

**3. Mybatis的一二级缓存知道吗？**`按照自己的理解说了`

- 一级缓存：基于PerpetualCache的HashMap本地缓存，其存储作用域为SqlSession,各个SqlSession之间的缓存相互隔离，当Session flush或close之后，该SqlSession中的所有Cache就将清空，MyBatis默认打开一级缓存
- 二级缓存与一级缓存其机制相同，默认也是采用PerpetualCache,HashMap存储，不同之处在于其存储作用域为Mapper(Namespace),可以在多个SqlSession之间共享，并且可自定义存储源，如Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口（可用来保存对象的状态），可在它的映射文件中配置
- 当开启二级缓存后，数据的查询执行的流程就是二级缓存->一级缓存->数据库。
- 缓存更新机制：当某一个作用域(一级缓存Session/二级缓存Mapper)进行了C/U/D操作（创建、更新、删除）后，默认该作用域下所有select中的缓存将被clear。

**4. Mybatis你是如何理解的呢？**`按照自己的理解说了

- 先说了什么是Mybatis
	- MyBatis是一个ORM（对象关系映射）框架，它内部封装了JDBC,开发时只需要关注SQL语句本身，不需要花费精力去处理加载驱动，创建连接，创建statement:等复杂的过程。开发人员不需要编写原生态sql,可以严格控制Sq执行性能，灵活度高。
	- MyBatis可以使用ml或者注解来配置映射原生信息，将PO]O映射成数据库中的记录，避免了几乎所有的]DBC代码和手动设置的参数以及获取结果集。

- 又说了Mybatis 的优点
	- 基于SQL语句编程，比较灵活，不会对应用程序或数据库现有设计造成影响，SQL写在XML里，解除sql语句和业务代码的耦合，便于统一管理，而且语句在XML里，可以复用
	- 与传统JDBC相比，减少了很多代码量，消除了大量代码冗余，不需要手动开关SqlSession的连接
	- 使用JDBC驱动连接数据库，所以JDBC支持的数据库，Mybatis都支持
	- 与Spring完美集成
	- 提供映射标签，支持对象与数据库ORM字段关系映射；对象关系也可以映射

- Mybatis的缺点
	- SQL语句依赖，需要开发人员具备一定的SQL知识，另外，如果数据库模式发生变化，还需要手动修改SQL语句
	- XML配置文件冗长，会导致一些维护问题，如果还使用注解配置，代码可能会混乱
	- 缺乏自动化创建，相比其他ORM框架，Mybatis不支持自动创建表和字段

- 最后又聊了聊Mybatis Plus
	- 主要是什么是MP
	- MP有什么优点和缺点

- 具体的应用场景就不多说了，懂得，哈哈哈哈


**5. Redis中的list和map数据类型了解吗？说说应用场景**`按照自己的理解说了`

因为伙伴匹配项目用redis作为旁路缓存了，主要用的是String类型的数据，就问了问其他数据结构有用过吗？

我说没用过，不过了解他的底层结构，也知道应用的场景

**List**
- 底层结构：
	- 3.2版本之前，List对象有两种编码方式，一种是ZIPLIST，另一种是LINKEDLIST
		- 列表对象保存的所有字符串对象`长度都小于64字节` 或者 列表对象元素`个数少于512个`，则使用ZIPLIST，否则使用LINKEDLIST
		- ZIPLIST（压缩列表）没有细说了
	- 3.2版本就引入了QUICKLIST。QUICKLIST其实就是ZIPLIST和LINKEDLIST的结合体
		- 当数据较少的时候，QUICKLIST的节点就只有一个，此时其实相当于就是一个ZIPLIST
		- 当数据很多的时候，则同时利用了ZIPLIST和LINKEDLIST的优势
- 使用场景
	- **朋友圈点赞**：发朋友圈的人用key表示，点赞的人为value,点赞操作对应rpush,取消点赞操作可以对应lrem。评论信息可以通过list去查询关系型数据库。
	- **消息队列**：list类型的lpop和rpush（或者反过来，lpush和rpop）能实现队列的功能，故而可以用Redis的list类型实现简单的点对点的消息队列。不推荐在实战中这么使用，因为现在已经有Kafka、NSQ、RabbitMQ等成熟的消息队列了，它们的功能已经很完善了。
	- **排行榜**：list类型的lrange命令可以分页查看队列中的数据。可每隔一段时间计算一次的排行榜存储在list类型中的数据；
	- 一般要求顺序的业务中，都用list来实现；

**Map/Hash**
- 底层结构
	- 压缩列表：
		1. Hash对象保存的所有值和键的长度都小于64字节；
		2. Hash对象元素个数少于512个。
	- 上述两个条件任何一条都不满足，编码结构就用HASHTABLE
		- HASHTABLE没有细说了，底层结构（字段），渐进式扩容，缩容没提了
- 应用场景：
	- 购物车：用户的id为key，商品id为field，商品数量为value
	- 存储对象：
		- 因为有些使用使用String（key-value=json）的时候，对象的某个属性频繁修改，每次修改都需要将整个对象重新JSON序列化，不够灵活；
		- 但是我们使用HASH，将经常发生变化的属性，存放在value里，json对象放在field中，就可以灵活的修改属性了，如：商品的价格、销量、关注数、评价数
## 3 手撕算法

[5. 最长回文子串 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-palindromic-substring/description/)

这次是直接给了个leetcode原题的链接了

- 分析了一下这个代码的实现思路
- 还有其他方式解决这个问题吗？`没答出来`

直接贴代码
```java
class Solution {
    public String longestPalindrome(String s) {
        // 思路：中心拓展法
        // ---> 中心向两边拓展，分奇数和偶数的情况，只要相同就继续拓展
        // ---> 直到不同为止
        for(int i=0;i<s.length();i++){
            // max记录最长回文串的长度，start记录最长回文串的起始位置
            int max = 0,start = 0;
            // 分奇数和偶数的情况
            for(int j;j<2;j++){
                // 左指针
                int left = i;
                // 右指针
                int right = i+j;
                // 循环从中心向两边拓展
                while（left>=0 && right<s.length() && s.charAt(left)==s.charAt(right)){
                    left--;
                    right++;
                }
                // 更新最长回文串的长度和起始位置
                if(right-left-1>max){
                    // 长度
                    max = right - left - 1;
                    // 起始位置
                    start = left + 1;
                }
            }
        }
        return s.substring(start,start+max);
    }
}
```