---
文章标题: "[[2_Map的源码分析]]" 
文章作者: Dakkk
文章概要: |
  文章对比了Java Map接口及其HashMap、LinkedHashMap、TreeMap等主要实现类的特性与用法。重点深入分析了HashMap在JDK7和JDK8底层源码，包括其数组、链表、红黑树数据结构、哈希碰撞处理、扩容机制及`put`方法流程，并简述了Set与Map的关系。
文章标签:
- "Java Collections"
- "HashMap"
- "LinkedHashMap"
- "TreeMap"
- "Map"
- "源码分析"
- "数据结构"
- "JDK7"
- "JDK8"
相关文章:
- "[[13_数据结构与集合源码]]"
- "[[7_集合Map]]"
- "[[2_ Collection]]"
- "[[5_集合List]]"
- "[[2_鱼骨头聊算法]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/05-💡 实用技巧/2_Map的源码分析.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2024-08-11 18:15:12
---

## 1 Map及其实现类对比  

储存一对一对的数据（entry：key-value 键值对，类似于高中的函数）  

- `HashMap（主要实现类）`：线程不安全，效率高；可以添加null的key和value值；底层使用数组，数组+单向链表+红黑树结构存储（jdk8）

- `LinkedHashMap` ：是HashMap的子类，在HashMap使用的数据结构的基础上，增加了一对双向链表，用于记录添加的元素的先后顺序，进而我们在遍历元素的时候，就可以按照添加的顺序显示  

    开发中，对于频繁的遍历操作，建议使用此类

- `TreeMap` ：底层使用红黑树储存；可以按照添加的key-value中的key元素的制定的属性的大小顺序进行遍历，需要考虑使用自然排序和定制排序  

- `Maptable` ：古老实现类；线程安全的，效率低；不可以添加null的key和value；底层使用数组和单向链表储存  

    - `Properties` ：其key和value都是String类型的。常用来处理属性文件

\[面试题] 

- 区分HashMap和Hashtable  

- 区分HashMap和LinkedHashMap  

- HashMap的底层实现（①new HashMap；②put（key，value））  

## 2  HashMap中元素的特点（熟悉！！！）  

- 所有的key彼此之间是不可重复的、无序的。所以key就构成了一个Set集合。---> key所在的类要求重写hashCode()和equals()方法  

- 所有的value彼此之间是可重复、无序的。所以value就构成了一个Collection集合。 ---> value所在的类要重写equals()方法 

- 一个key-value就构成了一个entry(node)  

- 所有的entry彼此之间是不可重复的、无序的。所以entry就构成了一个Set集合。  

## 3 Map中的常用方法  

小结:（记住！！！）  

- 增: put(Object key,Object value)  /  putAll(Map m)  

- 删: Object remove(Object key)  

- 改: put(Object key,Object value)  /  putAll(Map m)  

- 查: Object get(Object key)  

- 插: 无序的，无法插入  

- 长度: size()  

- 遍历:  

    - 遍历key集: Set keySet()  

    - 遍历value集: Collection values()  

    - 遍历entry集: Set entrySet()   ***注意key和value是entry这个内部类的属性！！！***

```java

    //遍历entry集合  

    @Test  

    public void test4(){  

        HashMap map = new HashMap();  

        //添加put(Object key, Object value)  

        map.put("AA",56);  

        map.put(67,"Tom");  

        map.put("BB",78);  

        map.put(new Person("Jerry",12),56);  

        Set set = map.entrySet();  

//        for (Object o : set){  

//            System.out.println(o);  

//        }  

//  

//        Set set1 = map.keySet();  

//        for (Object o: set1){  

//            System.out.println(o + "--->" + map.get(o));  

//        }  

        //利用内部类/内部接口  

        Iterator iterator = set.iterator();  

        while (iterator.hasNext()){  

            Map.Entry entry = (Map.Entry) iterator.next();  

            System.out.println(entry.getKey() + "--->" + entry.getValue());  

        }  

    }

```

## 4 TreeMap的使用（了解）  

- 底层使用红黑树结构  

- 可以按照添加的key-value中的key元素的指定的属性的大小顺序进行遍历  

- 需要考虑使用自然排序或定制排序  

- 要求！！！ 添加到key中的元素必须是一个类型的对象  

## 5 Hashtable与Properties的使用（重点的，在io中比较重要）  

- Properties是Hashtable的子类，其key和value都是String类型的，常用来处理属性文件。

- Properties的使用实例

  ```java

  public class PropertiesTest {

      @Test

      public void test() throws IOException { //注意：因为设计到流的操作，为了确保流能关闭，建议使用try-catch-finally

          //方式1：数据和代码耦合度高；如果修改的话，需要重写的编译代码、打包发布，繁琐

          //数据

  //        String name = "Tom";

  //        String password = "abc123";

          //代码：用于操作name,password

          //...

          //方式2：将数据封装到具体的配置文件中，在程序中读取配置文件中的信息。实现了

          //数据和代码的解耦；由于我们没有修改代码，就省去了重新编译和打包的过程。

          File file = new File("info.properties"); //注意，要提前创建好

  //        System.out.println(file.getAbsolutePath());

          FileInputStream fis = new FileInputStream(file);

          Properties pros = new Properties();

          pros.load(fis); //加载流中的文件中的数据

          //读取数据

          String name = pros.getProperty("name");

          String pwd = pros.getProperty("password");

          System.out.println(name + ":" + pwd);

          fis.close();

      }

  }

  ```

## 6 HashMap源码分析！！！ 

**链表法就是将相同hash值的对象组织成一个链表放在hash值对应的槽位；开放地址法是通过一个探测算法，当某个槽位已经被占据的情况下继续查找下一个可以使用的槽位。很显然我们使用的不是开放地址法。**

### 6.1 JDK7中的源码

- `添加/修改的过程`：  

- 将(key1,value1)添加到当前的map中  

- 首先，需要调用<font color="#d83931">key1所在类的hashCode()方法</font>，计算key1对应的哈希值1，此哈希值1经过某种算法(hash())之后，得到哈希值2。  

- 哈希值2再经过某种算法(indexFor())之后，就确定了(key1,value1)其在数组table中的索引位置i  

    - 如果此*索引位置i的数组上没有元素*，则(key1,value1)添加成功   **--->情况1**  

    - 如果此索引位置i的数组上有元素(key2,value2),则需要继续比较key1和key2的哈希值2    **---> 哈希冲突**  

        - 如果key1的哈希值2与key2的*哈希值2不相同*，则(key1,value1)添加成功。 **--->情况2**  

        - 如果key1的哈希值2与key2的*哈希值2相同*，则需要继续比较key1和key2的equals(),要调用key1所在*类的equals()*，将key2作为参数传递进去  

            - 调用equals()，返回false：则(key1,value1)添加成功   **--->情况3**  

            - 调用equals(),返回true，则key1和key2是相同的。默认情况下value1替换原有value2  

- 说明：  

- ***情况1***：将(key1,value1)存放到数组的索引i的位置  

- ***情况2，情况3***：(key1,value1)元素与现有的(key2,value2)构成单向链表结构，(key1,value1)指向(key2,value2)  

- 随着不断的添加元素，在满足如下条件的情况下，则会进行扩容  

- (size >= threshold) && (null != table\[bucketIndex]) //bucketIndex=i，`即超过12且出现链表了！！！`  

- 当元素的个数达到临界值（->数组的长度 * 加载因子（16\*0.75=12））时，就考虑扩容。默认临界值为12  

- 默认扩容为原来的2倍。

- get和remove的过程

```java

/*

① 计算key1的hash值，用这个方法hash(key1)

② 找index = table.length-1 & hash;

map.get(key1);

③ 如果table[index]不为空，那么就挨个比较哪个Entry的key与它相同，就返回它的value

map.remove(key1);

③ 如果table[index]不为空，那么就挨个比较哪个Entry的key与它相同，就删除它，把它前面的Entry的next的值修改为被删除Entry的next

*/

```

#### 6.1.1 属性

```java

//table数组的默认初始化长度

static final int DEFAULT_INITIAL_CAPACITY = 16;

//哈希表

transient Entry<K,V>[] table;

//哈希表中key-value的个数

transient int size;

//临界值、阈值（扩容的临界值）

int threshold;

//加载因子

final float loadFactor;

//默认加载因子

static final float DEFAULT_LOAD_FACTOR = 0.75f;

```

#### 6.1.2 构造器

```java

public HashMap() {

    //DEFAULT_INITIAL_CAPACITY：默认初始容量16

   //DEFAULT_LOAD_FACTOR：默认加载因子0.75

    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);

}

----------------------------

public HashMap(int initialCapacity, float loadFactor) {

    //校验initialCapacity合法性

    if (initialCapacity < 0)

        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);

    //校验initialCapacity合法性

    if (initialCapacity > MAXIMUM_CAPACITY)

        initialCapacity = MAXIMUM_CAPACITY;

    //校验loadFactor合法性

    if (loadFactor <= 0 || Float.isNaN(loadFactor))

        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);

    //计算得到table数组的长度（保证capacity是2的整次幂）

    int capacity = 1;

    while (capacity < initialCapacity)

        capacity <<= 1;

//加载因子，初始化为0.75

    this.loadFactor = loadFactor;

    // threshold 初始为默认容量

    threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);

    //初始化table数组

    table = new Entry[capacity];

    useAltHashing = sun.misc.VM.isBooted() &&

                                       (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);

    init();

}

```

#### 6.1.3 put()方法

```java

public V put(K key, V value) {

    //如果key是null，单独处理，存储到table[0]中，如果有另一个key为null，value覆盖

    if (key == null)

        return putForNullKey(value);

    //对key的hashCode进行干扰，算出一个hash值2

    /*

      hashCode值        xxxxxxxxxx

      table.length-1    000001111

      hashCode值 xxxxxxxxxx  无符号右移几位和原来的hashCode值做^运算，使得hashCode高位二进制值参与计算，

                            也发挥作用，降低index冲突的概率。

    */

    int hash = hash(key);

    //计算新的映射关系应该存到table[i]位置，

    //i = hash & table.length-1，可以保证i在[0,table.length-1]范围内

    int i = indexFor(hash, table.length);

    //检查table[i]下面有没有key与我新的映射关系的key重复，如果重复替换value

    for (Entry<K,V> e = table[i]; e != null; e = e.next) {

        Object k;

        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {

            V oldValue = e.value;

            e.value = value;

            e.recordAccess(this);

            return oldValue; //如果put是修改操作，返回oldvalue

        }

    }

    /*

    情况一：Entry<K,V> e = table[i]; e = null; 即索引i处没有元素，直接添加

    情况二：e！=null，比较e.hash == hash，在单向链表中不断的e.next,哈希值2都不一样，则存放在最上层的位置（原有位置的e，作为new的entry的next）；

    情况三：e！=null，且e.hash == hash，比较(k = e.key) == key || key.equals(k)，在单向链表中不断的e.next,若此处仍然不一样，和情况二存放的位置一样

    情况四：if满足的情况下，更换value，返回oldvalue。

    */

    modCount++;

    //添加新的映射关系

    addEntry(hash, key, value, i);

    return null; //如果put是添加操作，返回的是null

}

-------------------------------

//如果key是null，直接存入[0]的位置

private V putForNullKey(V value) {

    //判断是否有重复的key，如果有重复的，就替换value

    for (Entry<K,V> e = table[0]; e != null; e = e.next) {

        if (e.key == null) {

            V oldValue = e.value;

            e.value = value;

            e.recordAccess(this);

            return oldValue;

        }

    }

    modCount++;

    //把新的映射关系存入[0]的位置，而且key的hash值用0表示

    addEntry(0, null, value, 0);

    return null;

}

-----------------------------------

final int hash(Object k) {

    int h = 0;

    if (useAltHashing) {

        if (k instanceof String) {

            return sun.misc.Hashing.stringHash32((String) k);

        }

        h = hashSeed;

    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by

    // constant multiples at each bit position have a bounded

    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);

    return h ^ (h >>> 7) ^ (h >>> 4);

}

---------------------------------

static int indexFor(int h, int length) {

    return h & (length-1);

}

-------------------------------------

void addEntry(int hash, K key, V value, int bucketIndex) {

    //判断是否需要库容

    //扩容：（1）size达到阈值（2）table[i]正好非空

    if ((size >= threshold) && (null != table[bucketIndex])) {

        //table扩容为原来的2倍，并且扩容后，会重新调整所有key-value的存储位置

        resize(2 * table.length);

        //新的key-value的hash和index也会重新计算

        hash = (null != key) ? hash(key) : 0;

        bucketIndex = indexFor(hash, table.length);

    }

//存入table中

    createEntry(hash, key, value, bucketIndex);

}

-----------------------------------

void createEntry(int hash, K key, V value, int bucketIndex) {

    Entry<K,V> e = table[bucketIndex];

    //原来table[i]下面的映射关系作为新的映射关系next

    table[bucketIndex] = new Entry<>(hash, key, value, e);

    //个数增加

    size++;

}

```

#### 6.1.4 Entry

```java

public class HashMap<K,V>{

    transient Entry<K,V>[] table;

    static class Entry<K,V> implements Map.Entry<K,V> {

        final K key;

        V value;

        Entry<K,V> next;

        int hash; //使用key得到的哈希值二进行赋值

        /**

         * Creates new entry.

         */

        Entry(int h, K k, V v, Entry<K,V> n) {

            value = v;

            next = n;

            key = k;

            hash = h;

        }

        //略

    }

}

```

### 6.2 6.2JDK8中的源码

![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c0cc7e0477469a12ad31977be400acca.png)


![|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/9e4389516be9ef88dfc19d187a6f1344.png)


- 创建HashMap的实例后，底层并没有初始化table数组，当首次添加(key,value)时，进行判断，如果发现table尚未初始化，则对数组进行初始化  

- HashMap，底层定义了`Node内部类`，替换了jdk7中的Entry内部类。意味着，我们创建的是Node\[]  

- 如果当前的(key,value)经过一系列的判断后，可以添加到当前的数组角标i中，若角标i中有元素，jdk7中是新的指向旧的（头插法），jdk8中是旧的指向新的（尾插法）`”七上八下“`  

- 数组+单向链表 ---> 数组+单向链表+红黑树。  

- 如果数组`索引i位置上的元素的个数达到8`，并且`数组的长度达到64时`，我们就将此索引i位置上的多个元素改为`使用红黑树`的结构进行储存  

- 红黑树进行put、get、remove操作的时间复杂度为O（logn），比单向链表的O（n）。性能更好  

- 什么时候使用红黑树变为单向链表：当使用红黑树的`索引i位置上的元素的个数低于6`的时候，就会将红黑树退化为单向链表

#### 6.2.1 属性

```java

static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认的初始容量 16

static final int MAXIMUM_CAPACITY = 1 << 30; //最大容量  1 << 30

static final float DEFAULT_LOAD_FACTOR = 0.75f;  //默认加载因子

static final int TREEIFY_THRESHOLD = 8; //默认树化阈值8，当链表的长度达到这个值后，要考虑树化

static final int UNTREEIFY_THRESHOLD = 6;//默认反树化阈值6，当树中结点的个数达到此阈值后，要考虑变为链表

//当单个的链表的结点个数达到8，并且table的长度达到64，才会树化。

//当单个的链表的结点个数达到8，但是table的长度未达到64，会先扩容

static final int MIN_TREEIFY_CAPACITY = 64; //最小树化容量64

transient Node<K,V>[] table; //数组

transient int size;  //记录有效映射关系的对数，也是Entry对象的个数

int threshold; //阈值，当size达到阈值时，考虑扩容

final float loadFactor; //加载因子，影响扩容的频率

```

#### 6.2.2 构造器

```java

public HashMap() {

    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted (其他字段都是默认值)

}

```

#### 6.2.3 put()方法

```java

public V put(K key, V value) {

    return putVal(hash(key), key, value, false, true);

}

**********************

static final int hash(Object key) {

    int h;

    //如果key是null，hash是0

//如果key非null，用key的hashCode值 与 key的hashCode值高16进行异或

// 即就是用key的hashCode值高16位与低16位进行了异或的干扰运算

/*

index = hash & table.length-1

如果用key的原始的hashCode值  与 table.length-1 进行按位与，那么基本上高16没机会用上。

这样就会增加冲突的概率，为了降低冲突的概率，把高16位加入到hash信息中。

*/

    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);

}

**********************************

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {

    Node<K,V>[] tab; //数组

    Node<K,V> p;  //一个结点

    int n, i; //n是数组的长度   i是下标

    //tab和table等价

//如果table是空的

    if ((tab = table) == null || (n = tab.length) == 0){

        n = (tab = resize()).length;

        /*

tab = resize();

n = tab.length;*/

/*

如果table是空的，resize()完成了①创建了一个长度为16的数组②threshold = 12

n = 16

*/

}

    //i = (n - 1) & hash ，下标 = 数组长度-1 & hash

//p = tab[i] 第1个结点

//if(p==null) 条件满足的话说明 table[i]还没有元素

    if ((p = tab[i = (n - 1) & hash]) == null){

        //把新的映射关系直接放入table[i]

        tab[i] = newNode(hash, key, value, null);

        //newNode（）方法就创建了一个Node类型的新结点，新结点的next是null

    }else {

        Node<K,V> e; K k;

        //p是table[i]中第一个结点

//if(table[i]的第一个结点与新的映射关系的key重复)

        if (p.hash == hash &&

            ((k = p.key) == key || (key != null && key.equals(k))))

            e = p;//用e记录这个table[i]的第一个结点

        else if (p instanceof TreeNode){ //如果table[i]第一个结点是一个树结点

            //单独处理树结点

            //如果树结点中，有key重复的，就返回那个重复的结点用e接收，即e!=null

            //如果树结点中，没有key重复的，就把新结点放到树中，并且返回null，即e=null

            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);

        }else {

            //table[i]的第一个结点不是树结点，也与新的映射关系的key不重复

//binCount记录了table[i]下面的结点的个数

            for (int binCount = 0; ; ++binCount) {

                //如果p的下一个结点是空的，说明当前的p是最后一个结点

                if ((e = p.next) == null) {

                    //把新的结点连接到table[i]的最后

                    p.next = newNode(hash, key, value, null);

                    //如果binCount>=8-1，达到7个时

                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st

                        //要么扩容，要么树化

                        treeifyBin(tab, hash);

                    break;

                }

                //如果key重复了，就跳出for循环，此时e结点记录的就是那个key重复的结点

                if (e.hash == hash &&

                        ((k = e.key) == key || (key != null && key.equals(k))))

                    break;

                p = e;//下一次循环，e=p.next，就类似于e=e.next，往链表下移动

            }

        }

        //如果这个e不是null，说明有key重复，就考虑替换原来的value

        if (e != null) { // existing mapping for key

            V oldValue = e.value;

            if (!onlyIfAbsent || oldValue == null)

                e.value = value;

            afterNodeAccess(e); //什么也没干

            return oldValue;

        }

    }

    ++modCount;

    //元素个数增加

//size达到阈值

    if (++size > threshold)

        resize(); //一旦扩容，重新调整所有映射关系的位置

    afterNodeInsertion(evict); //什么也没干

    return null;

}

*****************************

final Node<K,V>[] resize() {

    Node<K,V>[] oldTab = table; //oldTab原来的table

    //oldCap：原来数组的长度

    int oldCap = (oldTab == null) ? 0 : oldTab.length;

    //oldThr：原来的阈值

    int oldThr = threshold;//最开始threshold是0

    //newCap，新容量

//newThr：新阈值

    int newCap, newThr = 0;

    if (oldCap > 0) { //说明原来不是空数组

        if (oldCap >= MAXIMUM_CAPACITY) { //是否达到数组最大限制

            threshold = Integer.MAX_VALUE;

            return oldTab;

        }

        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&

                 oldCap >= DEFAULT_INITIAL_CAPACITY)

            //newCap = 旧的容量*2 ，新容量<最大数组容量限制

//新容量：32,64，...

//oldCap >= 初始容量16

//新阈值重新算 = 24，48 ....

            newThr = oldThr << 1; // double threshold

    }

    else if (oldThr > 0) // initial capacity was placed in threshold

        newCap = oldThr;

    else {               // zero initial threshold signifies using defaults

        newCap = DEFAULT_INITIAL_CAPACITY; //新容量是默认初始化容量16

        //新阈值= 默认的加载因子 * 默认的初始化容量 = 0.75*16 = 12

        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);

    }

    if (newThr == 0) {

        float ft = (float)newCap * loadFactor;

        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?

                  (int)ft : Integer.MAX_VALUE);

    }

    threshold = newThr; //阈值赋值为新阈值12，24.。。。

    //创建了一个新数组，长度为newCap，16，32,64.。。

    @SuppressWarnings({"rawtypes","unchecked"})

    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];

    table = newTab;

    if (oldTab != null) { //原来不是空数组

        //把原来的table中映射关系，倒腾到新的table中

        for (int j = 0; j < oldCap; ++j) {

            Node<K,V> e;

            if ((e = oldTab[j]) != null) {//e是table下面的结点

                oldTab[j] = null; //把旧的table[j]位置清空

                if (e.next == null) //如果是最后一个结点

                    newTab[e.hash & (newCap - 1)] = e; //重新计算e的在新table中的存储位置，然后放入

                else if (e instanceof TreeNode) //如果e是树结点

                    //把原来的树拆解，放到新的table

                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);

                else { // preserve order

                    Node<K,V> loHead = null, loTail = null;

                    Node<K,V> hiHead = null, hiTail = null;

                    Node<K,V> next;

                    //把原来table[i]下面的整个链表，重新挪到了新的table中

                    do {

                        next = e.next;

                        if ((e.hash & oldCap) == 0) {

                            if (loTail == null)

                                loHead = e;

                            else

                                loTail.next = e;

                            loTail = e;

                        }

                        else {

                            if (hiTail == null)

                                hiHead = e;

                            else

                                hiTail.next = e;

                            hiTail = e;

                        }

                    } while ((e = next) != null);

                    if (loTail != null) {

                        loTail.next = null;

                        newTab[j] = loHead;

                    }

                    if (hiTail != null) {

                        hiTail.next = null;

                        newTab[j + oldCap] = hiHead;

                    }

                }

            }

        }

    }

    return newTab;

}

***********************************

Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {

    //创建一个新结点

    return new Node<>(hash, key, value, next);

}

**************************************

final void treeifyBin(Node<K,V>[] tab, int hash) {

    int n, index;

    Node<K,V> e;

    //MIN_TREEIFY_CAPACITY：最小树化容量64

    //如果table是空的，或者  table的长度没有达到64

    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)

        resize();//先扩容

    else if ((e = tab[index = (n - 1) & hash]) != null) {

        //用e记录table[index]的结点的地址

        TreeNode<K,V> hd = null, tl = null;

        /*

do...while，把table[index]链表的Node结点变为TreeNode类型的结点

*/

        do {

            TreeNode<K,V> p = replacementTreeNode(e, null);

            if (tl == null)

                hd = p;//hd记录根结点

            else {

                p.prev = tl;

                tl.next = p;

            }

            tl = p;

        } while ((e = e.next) != null);

        //如果table[index]下面不是空

        if ((tab[index] = hd) != null)

            hd.treeify(tab);//将table[index]下面的链表进行树化

    }

}

```

## 7 LinkedHashMap源码剖析

### 7.1 源码

内部定义的Entry如下：

```java

static class Entry<K,V> extends HashMap.Node<K,V> {

Entry<K,V> before, after;

//体现了双向链表

Entry(int hash, K key, V value, Node<K,V> next) {

super(hash, key, value, next);

}

}

```

LinkedHashMap重写了HashMap中的newNode()方法：

```java

Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {

    LinkedHashMap.Entry<K,V> p =

        new LinkedHashMap.Entry<K,V>(hash, key, value, e);

    linkNodeLast(p);

    return p;

}

```

```java

TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {

    TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);

    linkNodeLast(p);

    return p;

}

```

## 8 Set接口分析

**Set集合与Map集合的关系**

Set的内部实现其实是一个Map，Set中的元素，存储在HashMap的key中。即HashSet的内部实现是一个HashMap，TreeSet的内部实现是一个TreeMap，LinkedHashSet的内部实现是一个LinkedHashMap。