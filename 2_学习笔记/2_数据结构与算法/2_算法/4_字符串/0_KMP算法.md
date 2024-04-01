## 目录

- [1 KMP](#1%20KMP)
	- [1.1 概念](#1.1%20%E6%A6%82%E5%BF%B5)
	- [1.2 作用](#1.2%20%E4%BD%9C%E7%94%A8)
- [2 前缀表](#2%20%E5%89%8D%E7%BC%80%E8%A1%A8)
	- [2.1 概念](#2.1%20%E6%A6%82%E5%BF%B5)
	- [2.2 必要性](#2.2%20%E5%BF%85%E8%A6%81%E6%80%A7)
	- [2.3 计算前缀表](#2.3%20%E8%AE%A1%E7%AE%97%E5%89%8D%E7%BC%80%E8%A1%A8)
- [3 3前缀表与next数组](#3%203%E5%89%8D%E7%BC%80%E8%A1%A8%E4%B8%8Enext%E6%95%B0%E7%BB%84)
	- [3.1 使用next数组来匹配](#3.1%20%E4%BD%BF%E7%94%A8next%E6%95%B0%E7%BB%84%E6%9D%A5%E5%8C%B9%E9%85%8D)
	- [3.2 时间复杂度分析](#3.2%20%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6%E5%88%86%E6%9E%90)
	- [3.3 构造next数组](#3.3%20%E6%9E%84%E9%80%A0next%E6%95%B0%E7%BB%84)
		- [3.3.1 初始化：](#3.3.1%20%E5%88%9D%E5%A7%8B%E5%8C%96%EF%BC%9A)
		- [3.3.2 处理前后缀不相同的情况](#3.3.2%20%E5%A4%84%E7%90%86%E5%89%8D%E5%90%8E%E7%BC%80%E4%B8%8D%E7%9B%B8%E5%90%8C%E7%9A%84%E6%83%85%E5%86%B5)
		- [3.3.3 处理前后缀相同的情况](#3.3.3%20%E5%A4%84%E7%90%86%E5%89%8D%E5%90%8E%E7%BC%80%E7%9B%B8%E5%90%8C%E7%9A%84%E6%83%85%E5%86%B5)
	- [3.4 使用next数组来匹配](#3.4%20%E4%BD%BF%E7%94%A8next%E6%95%B0%E7%BB%84%E6%9D%A5%E5%8C%B9%E9%85%8D)
- [4 代码实现](#4%20%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0)
	- [4.1 前缀表统一减一](#4.1%20%E5%89%8D%E7%BC%80%E8%A1%A8%E7%BB%9F%E4%B8%80%E5%87%8F%E4%B8%80)
	- [4.2 前缀表（不减一）](#4.2%20%E5%89%8D%E7%BC%80%E8%A1%A8%EF%BC%88%E4%B8%8D%E5%87%8F%E4%B8%80%EF%BC%89)

## 1 KMP

### 1.1 概念

KMP的经典思想就是：**当出现字符串不匹配时，可以记录一部分之前已经匹配的文本内容，利用这些信息避免从头再去做匹配。**

说到KMP，先说一下KMP这个名字是怎么来的，为什么叫做KMP呢。

因为是由这三位学者发明的：Knuth，Morris和Pratt，所以取了三位学者名字的首字母。所以叫做KMP
### 1.2 作用

`KMP主要应用在字符串匹配上`。

KMP的主要思想是**当出现字符串不匹配时，可以知道一部分之前已经匹配的文本内容，可以利用这些信息避免从头再去做匹配了。**

所以如何记录已经匹配的文本内容，是KMP的重点，也是next数组肩负的重任。

其实KMP的代码不好理解，一些同学甚至直接把KMP代码的模板背下来。

没有彻底搞懂，懵懵懂懂就把代码背下来太容易忘了。

不仅面试的时候可能写不出来，如果面试官问：**next数组里的数字表示的是什么，为什么这么表示？**

估计大多数候选人都是懵逼的。
## 2 前缀表

### 2.1 概念

写过KMP的同学，一定都写过next数组，那么这个next数组究竟是个啥呢？

next数组就是一个前缀表（prefix table）。

前缀表有什么作用呢？

**前缀表是用来回退的，它记录了模式串与主串(文本串)不匹配的时候，模式串应该从哪里开始重新匹配。**

为了清楚地了解前缀表的来历，我们来举一个例子：

- 要在文本串：aabaabaafa 中查找是否出现过一个模式串：aabaaf。
- 请记住文本串和模式串的作用，对于理解下文很重要，要不然容易看懵。
- 如动画所示：![KMP精讲1.gif|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/018c8e5756939dc350c66a425850f38a.gif)

	- 动画中，特意把子串`aa`标记上了，这是有原因的，先注意一下，后面会说到；
	- 可以看出，文本串第6个字符b 和 模式串第6个字符f，不匹配了。如果暴力匹配，发现不匹配，此时就要插头开始匹配了
	- 但如果使用前缀表，就不会从头匹配，而是从上次已经匹配的内容开始匹配，找到了模式串第3个字符b 继续开始匹配

此时就要问`前缀表是如何记录的呢？`
- 首先，要知道前缀表的任务是`当前位置匹配失败，找到之前已经匹配上的位置，再重新匹配`
- 这就，意味着在某个字符失配时，前缀表会告诉你`下一步匹配中，模式串应该跳到那个位置`
	- 示例中，下标i之前的字符串为aabaa
	- 所以最长前缀为aa，最长后缀为aa，且前缀 == 后缀
	- 那么我们找到最长前缀后一位字符，就是模式串应该跳的位置

所以，前缀表就是`记录下标i之前（包括i）的字符串中，有多大长度的相同前缀后缀`

`注：正确理解什么是前缀和后缀很重要！`
- 前缀：指不包含最后一个字符的所有以第一个字符开头的连续子串
	- 示例中：a、aa、aab、aaba、aabaa都是前缀
- 后缀：指不包含第一个字符的所有以最后一个字符结尾的连续子串
	- 示例中：f、af、aaf、baaf、abaaf都是后缀
### 2.2 必要性

回顾一下，刚刚匹配的过程在下标5的地方遇到不匹配，模式串是指向f，如图：![image.png|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/208777c288b806a8cbf19a656ebf3134.png)
然后就找到了下标2，指向b，继续匹配：如图：![image.png|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/32f605cac9023a24710ee6dc61605bec.png)
以下这句话，对于理解为什么使用前缀表可以告诉我们匹配失败之后跳到哪里重新匹配 非常重要！

下标5之前这部分的字符串（也就是字符串aabaa）的最长相等的前缀 和 后缀字符串是 子字符串aa

因为找到了最长相等的前缀和后缀，`匹配失败的位置是后缀子串的后面，那么我们找到与其相同的前缀的后面重新匹配就可以了`。

所以前缀表具有告诉我们当前位置匹配失败，跳到之前已经匹配过的地方的能力。
### 2.3 计算前缀表

接下来就要说一说怎么计算前缀表。

如图：![image.png|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/36f4d7c4d6d3588c3415c51f9fff1739.png)
长度为前1个字符的子串`a`，最长相同前后缀的长度为0。

长度为前2个字符的子串`aa`，最长相同前后缀的长度为1。![image.png|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/853077afe023385f87243f18531b2388.png)
长度为前3个字符的子串`aab`，最长相同前后缀的长度为0。![image.png|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d1cff105c75291fcf992838baf2ddaee.png)

以此类推： 
- 长度为前4个字符的子串`aaba`，最长相同前后缀的长度为1。 
- 长度为前5个字符的子串`aabaa`，最长相同前后缀的长度为2。 
- 长度为前6个字符的子串`aabaaf`，最长相同前后缀的长度为0。

那么把求得的最长相同前后缀的长度就是对应前缀表的元素，如图：![image.png|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/664b4f443d5ef302c65f6f3baeb8b5dd.png)
可以看出模式串与前缀表对应位置的数字表示的就是：**下标i之前（包括i）的字符串中，有多大长度的相同前缀后缀。**

再来看一下如何利用 前缀表找到 当字符不匹配的时候应该指针应该移动的位置。如动画所示：![KMP精讲2.gif|200|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c4a6953c87145ced8c9b403c2c73640c.gif)

找到的不匹配的位置， 那么此时我们要看它的前一个字符的前缀表的数值是多少。

为什么要前一个字符的前缀表的数值呢，因为要`找前面字符串的最长相同的前缀和后缀`。

所以要看前一位的前缀表的数值。

前一个字符的前缀表的数值是2， 所以把下标移动到下标2的位置继续比配。 可以再反复看一下上面的动画。

最后就在文本串中找到了和模式串匹配的子串了。

## 3 前缀表与next数组

很多KMP算法的实现都是使用next数组来做回退操作，那么next数组与前缀表有什么关系呢？

next数组就可以是前缀表，但是很多实现都是把前缀表统一减一（右移一位，初始位置为-1）之后作为next数组。

为什么这么做呢，其实也是很多文章视频没有解释清楚的地方。

其实**这并不涉及到KMP的原理，而是具体实现，next数组既可以就是前缀表，也可以是前缀表统一减一（右移一位，初始位置为-1）。**

后面我会提供两种不同的实现代码，大家就明白了。
### 3.1 使用next数组来匹配

**以下我们以前缀表统一减一之后的next数组来做演示**。

有了next数组，就可以根据next数组来 匹配文本串s，和模式串t了。

`注意next数组是新前缀表（旧前缀表统一减一了）`。

匹配过程动画如下：![KMP精讲4.gif|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/26dde3bfb2dc6fe5c8026c9dcfe4fca9.gif)
### 3.2 时间复杂度分析

其中n为文本串长度，m为模式串长度，因为在匹配的过程中，根据前缀表不断调整匹配的位置，可以看出匹配的过程是O(n)，之前还要单独生成next数组，时间复杂度是O(m)。所以整个KMP算法的时间复杂度是O(n+m)的。

暴力的解法显而易见是O(n × m)，所以**KMP在字符串匹配中极大地提高了搜索的效率。**

为了和力扣题目28.实现strStr保持一致，方便大家理解，以下文章统称haystack为文本串, needle为模式串。

都知道使用KMP算法，一定要构造next数组。
### 3.3 构造next数组

我们定义一个函数getNext来构建next数组，函数参数为next数组，和一个字符串。 代码如下：

```java
void getNext(int[] next, string s)
```

**构造next数组其实就是计算模式串s，前缀表的过程。** 主要有如下三步：
1. 初始化
2. 处理前后缀不相同的情况
3. 处理前后缀相同的情况

接下来我们详解一下。

#### 3.3.1 初始化：

`定义两个指针i和j，j指向前缀末尾位置，i指向后缀末尾位置`。

然后还要对next数组进行初始化赋值，如下：
```java
int j = -1;
next[0] = j;
```

j 为什么要初始化为 -1呢，因为之前说过 前缀表要统一减一的操作仅仅是其中的一种实现，我们这里选择j初始化为-1，下文我还会给出j不初始化为-1的实现代码。

`next[i] 表示 i（包括i）之前最长相等的前后缀长度（其实就是j）`![image.png|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6f503a0d70a95cb6f4cb280422aac3d5.png)


所以初始化next[0] = j 。
#### 3.3.2 处理前后缀不相同的情况

因为j初始化为-1，那么i就从1开始，进行s[i] 与 s[j+1]的比较。

所以遍历模式串s的循环下标i 要从 1开始，代码如下：

```java
for (int i = 1; i < s.length(); i++) {
```


如果` s[i] 与 s[j+1]不相同`，也就是遇到 `前后缀末尾不相同的情况，就要向前回退`。

怎么回退呢？

next[j]就是记录着j（包括j）之前的子串的相同前后缀的长度。

那么 s[i] 与 s[j+1] 不相同，就要找 j+1前一个元素在next数组里的值（就是next[j]）。

所以，处理前后缀不相同的情况代码如下：

```java
while(j >= 0 && s.charAt(i) != s.charAt(j+1)){
	//循环不变量
	//不断遍历前一个子串的状态
	//即上一个前缀的 最长相等前后缀子串
	j=next[j];
}
```

#### 3.3.3 处理前后缀相同的情况

如果 s[i] 与 s[j + 1] 相同，那么就同时向后移动i 和j 说明找到了相同的前后缀，同时还要将j（前缀的长度）赋给next[i], 因为next[i]要记录相同前后缀的长度。

代码如下：

```java
if(s.charAt(i) == s.charAt(j+1)){
	j++;
}
next[i] = j;
```
最后整体构建next数组的函数代码如下：

```java
public void getNext(int[] next, String s){
	int j = -1;
	next[0] = j;
	for (int i = 1; i < s.length(); i++){
		while(j >= 0 && s.charAt(i) != s.charAt(j+1)){
			j=next[j];
		}

		if(s.charAt(i) == s.charAt(j+1)){
			j++;
		}
		next[i] = j;
	}
}
```

代码构造next数组的逻辑流程动画如下：![KMP精讲3.gif|300|380](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/f818be1946b45387c6b256e31fd71d97.gif)



得到了next数组之后，就要用这个来做匹配了。
### 3.4 使用next数组来匹配

在文本串s里 找是否出现过模式串t。

定义两个下标 `j 指向模式串起始位置`，`i指向文本串起始位置`。

那么j初始值依然为-1，为什么呢？ **依然因为next数组里记录的起始位置为-1。**

i就从0开始，遍历文本串，代码如下：

```java
for (int i = 0; i < s.length(); i++) 
```

接下来就是 s[i] 与 t[j + 1] （因为j从-1开始的） 进行比较。

如果 s[i] 与 t[j + 1] 不相同，j就要从next数组里寻找下一个匹配的位置。

代码如下：

```java
while(j >= 0 && s.charAt(i) != s.charAt(j+1)){
    j = next[j];
}
```


如果 s[i] 与 t[j + 1] 相同，那么i 和 j 同时向后移动， 代码如下：

```java
if (s.charAt(i) == s.charAt(j+1)) {
    j++; // i的增加在for循环里
}
```
 

如何判断在文本串s里出现了模式串t呢，如果j指向了模式串t的末尾，那么就说明模式串t完全匹配文本串s里的某个子串了。

本题要在文本串字符串中找出模式串出现的第一个位置 (从0开始)，所以返回当前在文本串匹配模式串的位置i 减去 模式串的长度，就是文本串字符串中出现模式串的第一个位置。

代码如下：

```java
if (j == (t.length() - 1) ) {
    return (i - t.length() + 1);
}
```

那么使用next数组，用模式串匹配文本串的整体代码如下：

```java
// 因为next数组里记录的起始位置为-1
int j = -1; 
// 注意i就从0开始
for (int i = 0; i < s.length(); i++) { 
    // 不匹配
    while(j >= 0 && s.charAt(i) != s.charAt(j+1)) { 
        // j 寻找之前匹配的位置
        j = next[j]; 
    }
    // 匹配，j和i同时向后移动
    if (s.charAt(i) == s.charAt(j+1)) { 
        j++;
        // i的增加在for循环里 
    }
    // 文本串s里出现了模式串t
    if (j == (t.length() - 1) ) { 
        return (i - t.length() + 1);
    }
}
```

## 4 代码实现

此时所有逻辑的代码都已经写出来了，力扣 28.实现strStr 题目的整体代码如下：
### 4.1 前缀表统一减一

```java
// 方法一
class Solution {
    public void getNext(int[] next, String s){
        int j = -1;
        next[0] = j;
        for (int i = 1; i < s.length(); i++){
            while(j >= 0 && s.charAt(i) != s.charAt(j+1)){
                j=next[j];
            }

            if(s.charAt(i) == s.charAt(j+1)){
                j++;
            }
            next[i] = j;
        }
    }
    public int strStr(String haystack, String needle) {
        if(needle.length()==0){
            return 0;
        }

        int[] next = new int[needle.length()];
        getNext(next, needle);
        int j = -1;
        for(int i = 0; i < haystack.length(); i++){
            while(j>=0 && haystack.charAt(i) != needle.charAt(j+1)){
                j = next[j];
            }
            if(haystack.charAt(i) == needle.charAt(j+1)){
                j++;
            }
            if(j == needle.length()-1){
                return (i-needle.length()+1);
            }
        }

        return -1;
    }
}
```
### 4.2 前缀表（不减一）

```java
//KMP算法
class Solution {
    public int strStr(String haystack, String needle) {
        //1.获取needle的next数组
        int[] next = new int[needle.length()];
        getNext(next, needle);

        //2.遍历haystack字符串
        int j = 0;
        for(int i=0;i<haystack.length();i++){
            //2.1 两个字符串首元素不相等
            // i回退，值为前缀数组中j的前一位元素
            while(j>0&&needle.charAt(j)!=haystack.charAt(i)){
                j = next[j-1];
            }
            //2.2 逐个字符判断
            if(needle.charAt(j)==haystack.charAt(i)) j++;
            //2.3 到needle的末尾，表示存在
            if(j==needle.length()) return i-needle.length()+1;
        }
        //3.没有找到
        return -1;
    }

    public void getNext(int[] next,String s){
        //1.初始化(不偏移next数组)
        int j = 0;
        next[0] = j;
        //2.给next数组赋值（i代表后缀末尾，j代表前缀末尾）
        for(int i=1;i<s.length();i++){
            //2.1 前后缀末尾不相等
            //注意j是大于0的，避免回退的时候，越界了
            while(j>0&&s.charAt(j)!=s.charAt(i)){
                //这一步是关键：回退j
                j = next[j-1];
            }
            //2.2 前后缀末尾相等
            if(s.charAt(j)==s.charAt(i)) j++;
            //2.3 next[i]的取值
            next[i] = j;
        }
    }
}
```


