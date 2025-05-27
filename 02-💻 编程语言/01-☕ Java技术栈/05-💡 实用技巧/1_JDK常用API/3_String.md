---
文章标题: "[[3_String]]" 
文章作者: Dakkk
文章概要: |
  文章系统介绍了Java String类的构造方式、与各类数据转换、核心API方法（查找、截取、替换等）及常见字符串算法实现，通过代码示例提供了Java字符串操作的实用指南。
tags:
- "Java String"
- "字符串操作"
- "字符编码"
- "数据类型转换"
- "String API"
- "字符串算法"
- "Java基础"
相关文章:
- "[[1_基础概念与常识]]"
- "[[1_Java值传递详解]]"
- "[[3_字符串替换]]"
- "[[3_字符集详解]]"
- "[[5_方法]]"
文章分类: "💻 编程语言"
文章路径: "02-💻 编程语言/01-☕ Java技术栈/05-💡 实用技巧/1_JDK常用API/3_String.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-08-11 18:15:12
修改时间: 2025-05-27 21:53:49
---

`String` 类包括的方法可用于检查序列的单个字符、比较字符串、搜索字符串、提取子字符串、创建字符串副本并将所有字符全部转换为大写或小写。

## 1 构造器

* `public String() ` ：初始化新创建的 String对象，以使其表示空字符序列。
* ` String(String original)`： 初始化一个新创建的 `String` 对象，使其表示一个与参数相同的字符序列；换句话说，新创建的字符串是该参数字符串的副本。
* `public String(char[] value) ` ：通过当前参数中的字符数组来构造新的String。
* `public String(char[] value,int offset, int count) ` ：通过字符数组的一部分来构造新的String。
* `public String(byte[] bytes) ` ：通过使用平台的**默认字符集**解码当前参数中的字节数组来构造新的String。
* `public String(byte[] bytes,String charsetName) ` ：通过使用指定的字符集解码当前参数中的字节数组来构造新的String。

举例：

```java
//字面量定义方式：字符串常量对象
String str = "hello";

//构造器定义方式：无参构造
String str1 = new String();

//构造器定义方式：创建"hello"字符串常量的副本
String str2 = new String("hello");

//构造器定义方式：通过字符数组构造
char chars[] = {'a', 'b', 'c','d','e'};     
String str3 = new String(chars);
String str4 = new String(chars,0,3);

//构造器定义方式：通过字节数组构造
byte bytes[] = {97, 98, 99 };     
String str5 = new String(bytes);
String str6 = new String(bytes,"GBK");
```

```java
public static void main(String[] args) {
	char[] data = {'h','e','l','l','o','j','a','v','a'};
	String s1 = String.copyValueOf(data);
	String s2 = String.copyValueOf(data,0,5);
	int num = 123456;
	String s3 = String.valueOf(num);
	
    System.out.println(s1);
	System.out.println(s2);
	System.out.println(s3);
}
```

## 2 与其他结构间的转换

**字符串 --> 基本数据类型、包装类：**

- Integer包装类的public static int parseInt(String s)：可以将由“数字”字符组成的字符串转换为整型。
- 类似地，使用java.lang包中的Byte、Short、Long、Float、Double类调相应的类方法可以将由“数字”字符组成的字符串，转化为相应的基本数据类型。

**基本数据类型、包装类 --> 字符串：**

- 调用String类的public String valueOf(int n)可将int型转换为字符串
- 相应的valueOf(byte b)、valueOf(long l)、valueOf(float f)、valueOf(double d)、valueOf(boolean b)可由参数的相应类型到字符串的转换。

 **字符数组 -->  字符串：**

- String 类的构造器：String(char[]) 和 String(char[]，int offset，int length) 分别用字符数组中的全部字符和部分字符创建字符串对象。 

 **字符串 -->  字符数组：**

- public char[] toCharArray()：将字符串中的全部字符存放在一个字符数组中的方法。

- public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)：提供了将指定索引范围内的字符串存放到数组中的方法。

**字符串 --> 字节数组：（编码）**

- public byte[] getBytes() ：使用平台的默认字符集将此 String 编码为 byte 序列，并将结果存储到一个新的 byte 数组中。
- public byte[] getBytes(String charsetName) ：使用指定的字符集将此 String 编码到 byte 序列，并将结果存储到新的 byte 数组。

 **字节数组 --> 字符串：（解码）**

- String(byte[])：通过使用平台的默认字符集解码指定的 byte 数组，构造一个新的 String。
- String(byte[]，int offset，int length) ：用指定的字节数组的一部分，即从数组起始位置offset开始取length个字节构造一个字符串对象。
- String(byte[], String charsetName ) 或 new String(byte[], int, int,String charsetName )：解码，按照指定的编码方式进行解码。

代码示例：

```java
@Test
public void test01() throws Exception {
    String str = "中国";
    System.out.println(str.getBytes("ISO8859-1").length);// 2
    // ISO8859-1把所有的字符都当做一个byte处理，处理不了多个字节
    System.out.println(str.getBytes("GBK").length);// 4 每一个中文都是对应2个字节
    System.out.println(str.getBytes("UTF-8").length);// 6 常规的中文都是3个字节

    /*
     * 不乱码：（1）保证编码与解码的字符集名称一样（2）不缺字节
     */
    System.out.println(new String(str.getBytes("ISO8859-1"), "ISO8859-1"));// 乱码
    System.out.println(new String(str.getBytes("GBK"), "GBK"));// 中国
    System.out.println(new String(str.getBytes("UTF-8"), "UTF-8"));// 中国
}

## 2、与其他结构转换


## 1、常用方法

（1）boolean isEmpty()：字符串是否为空 
（2）int length()：返回字符串的长度 
（3）String concat(xx)：拼接 
（4）boolean equals(Object obj)：比较字符串是否相等，区分大小写 
（5）boolean equalsIgnoreCase(Object obj)：比较字符串是否相等，不区分大小写 （6）int compareTo(String other)：比较字符串大小，区分大小写，按照Unicode编码值比较大小 
（7）int compareToIgnoreCase(String other)：比较字符串大小，不区分大小写 
（8）String toLowerCase()：将字符串中大写字母转为小写 
（9）String toUpperCase()：将字符串中小写字母转为大写 
（10）String trim()：去掉字符串前后空白符 
（11）public String intern()：结果在常量池中共享

```java
@Test  
public void test01(){  
	//将用户输入的单词全部转为小写，如果用户没有输入单词，重新输入  
	Scanner input = new Scanner(System.in);  
	String word;  
	while(true){  
		System.out.print("请输入单词：");  
		word = input.nextLine();  
		if(word.trim().length()!=0){  
			word = word.toLowerCase();  
			break;  
		}  
	}  
	System.out.println(word);  
}  
```
​  
```java
@Test  
    public void test02(){  
        //随机生成验证码，验证码由0-9，A-Z,a-z的字符组成  
        char[] array = new char[26*2+10];  
        for (int i = 0; i < 10; i++) {  
            array[i] = (char)('0' + i);  
        }  
        for (int i = 10,j=0; i < 10+26; i++,j++) {  
            array[i] = (char)('A' + j);  
        }  
        for (int i = 10+26,j=0; i < array.length; i++,j++) {  
            array[i] = (char)('a' + j);  
        }  
        String code = "";  
        Random rand = new Random();  
        for (int i = 0; i < 4; i++) {  
            code += array[rand.nextInt(array.length)];  
        }  
        System.out.println("验证码：" + code);  
        //将用户输入的单词全部转为小写，如果用户没有输入单词，重新输入  
        Scanner input = new Scanner(System.in);  
        System.out.print("请输入验证码：");  
        String inputCode = input.nextLine();  
          
        if(!code.equalsIgnoreCase(inputCode)){  
            System.out.println("验证码输入不正确");  
        }  
    }
```

## 3 查找

（11）boolean contains(xx)：是否包含xx 
（12）int indexOf(xx)：从前往后找当前字符串中xx，即如果有返回第一次出现的下标，要是没有返回-1 
（13）int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
（14）int lastIndexOf(xx)：从后往前找当前字符串中xx，即如果有返回最后一次出现的下标，要是没有返回-1
（15）int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索。

```java
    @Test  
    public void test01(){  
        String str = "尚硅谷是一家靠谱的培训机构，尚硅谷可以说是IT培训的小清华，JavaEE是尚硅谷的当家学科，尚硅谷的大数据培训是行业独角兽。尚硅谷的前端和UI专业一样独领风骚。";  
        System.out.println("是否包含清华：" + str.contains("清华"));  
        System.out.println("培训出现的第一次下标：" + str.indexOf("培训"));  
        System.out.println("培训出现的最后一次下标：" + str.lastIndexOf("培训"));  
    }
```

## 4 字符串截取

（16）String substring(int beginIndex) ：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。 
（17）String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。

```java
@Test  
public void test01(){  
    String str = "helloworldjavaatguigu";  
    String sub1 = str.substring(5);  
    String sub2 = str.substring(5,10);  
    System.out.println(sub1);  
    System.out.println(sub2);  
}  
​  
@Test  
public void test02(){  
    String fileName = "快速学习Java的秘诀.dat";  
    //截取文件名  
    System.out.println("文件名：" + fileName.substring(0,fileName.lastIndexOf(".")));  
    //截取后缀名  
    System.out.println("后缀名：" + fileName.substring(fileName.lastIndexOf(".")));  
}
```

## 5 和字符/字符数组相关

（18）char charAt(index)：返回[index]位置的字符 
（19）char[] toCharArray()： 将此字符串转换为一个新的字符数组返回 
（20）static String valueOf(char[] data) ：返回指定数组中表示该字符序列的 String （21）static String valueOf(char[] data, int offset, int count) ： 返回指定数组中表示该字符序列的 String 
（22）static String copyValueOf(char[] data)： 返回指定数组中表示该字符序列的 String 
（23）static String copyValueOf(char[] data, int offset, int count)：返回指定数组中表示该字符序列的 String

```java
   @Test  
    public void test01(){  
        //将字符串中的字符按照大小顺序排列  
        String str = "helloworldjavaatguigu";  
        char[] array = str.toCharArray();  
        Arrays.sort(array);  
        str = new String(array);  
        System.out.println(str);  
    }  
      
    @Test  
    public void test02(){  
        //将首字母转为大写  
        String str = "jack";  
        str = Character.toUpperCase(str.charAt(0))+str.substring(1);  
        System.out.println(str);  
    }  
    @Test  
    public void test03(){  
        char[] data = {'h','e','l','l','o','j','a','v','a'};  
        String s1 = String.copyValueOf(data);  
        String s2 = String.copyValueOf(data,0,5);  
        int num = 123456;  
        String s3 = String.valueOf(num);  
      
        System.out.println(s1);  
        System.out.println(s2);  
        System.out.println(s3);  
    }

```
 
## 6 开头与结尾

（24）boolean startsWith(xx)：测试此字符串是否以指定的前缀开始 
（25）boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始 
（26）boolean endsWith(xx)：测试此字符串是否以指定的后缀结束

```java
    @Test  
    public void test1(){  
        String name = "张三";  
        System.out.println(name.startsWith("张"));  
    }  
      
    @Test  
    public void test2(){  
        String file = "Hello.txt";  
        if(file.endsWith(".java")){  
            System.out.println("Java源文件");  
        }else if(file.endsWith(".class")){  
            System.out.println("Java字节码文件");  
        }else{  
            System.out.println("其他文件");  
        }  
    }
```

## 7 替换

（27）String replace(char oldChar, char newChar)：返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所有 oldChar 得到的。 不支持正则。 
（28）String replace(CharSequence target, CharSequence replacement)：使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串。 
（29）String replaceAll(String regex, String replacement)：使用给定的 replacement 替换此字符串所有匹配给定的正则表达式的子字符串。 
（30）String replaceFirst(String regex, String replacement)：使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串。

```java
@Test  
public void test1(){  
    String str1 = "hello244world.java;887";  
    //把其中的非字母去掉  
    str1 = str1.replaceAll("[^a-zA-Z]", "");  
    System.out.println(str1);  
​  
    String str2 = "12hello34world5java7891mysql456";  
    //把字符串中的数字替换成,，如果结果中开头和结尾有，的话去掉  
    String string = str2.replaceAll("\\d+", ",").replaceAll("^,|,$", "");  
    System.out.println(string);  
​  
}
```

## 8 常见算法题目

**题目1：** 模拟一个trim方法，去除字符串两端的空格。

```java
    public String myTrim(String str) {  
        if (str != null) {  
            int start = 0;// 用于记录从前往后首次索引位置不是空格的位置的索引  
            int end = str.length() - 1;// 用于记录从后往前首次索引位置不是空格的位置的索引  
​  
            while (start < end && str.charAt(start) == ' ') {  
                start++;  
            }  
​  
            while (start < end && str.charAt(end) == ' ') {  
                end--;  
            }  
            if (str.charAt(start) == ' ') {  
                return "";  
            }  
​  
            return str.substring(start, end + 1);  
        }  
        return null;  
    }  
​  
    @Test  
    public void testMyTrim() {  
        String str = "   a   ";  
        // str = " ";  
        String newStr = myTrim(str);  
        System.out.println("---" + newStr + "---");  
    }
```

**题目2：** 将一个字符串进行反转。将字符串中指定部分进行反转。比如“ab`cdef`g”反转为”ab`fedc`g”

```java
    // 方式一：  
    public String reverse1(String str, int start, int end) {// start:2,end:5  
        if (str != null) {  
            // 1.  
            char[] charArray = str.toCharArray();  
            // 2.  
            for (int i = start, j = end; i < j; i++, j--) {  
                char temp = charArray[i];  
                charArray[i] = charArray[j];  
                charArray[j] = temp;  
            }  
            // 3.  
            return new String(charArray);  
​  
        }  
        return null;  
​  
    }  
​  
    // 方式二：  
    public String reverse2(String str, int start, int end) {  
        // 1.  
        String newStr = str.substring(0, start);// ab  
        // 2.  
        for (int i = end; i >= start; i--) {  
            newStr += str.charAt(i);  
        } // abfedc  
            // 3.  
        newStr += str.substring(end + 1);  
        return newStr;  
    }  
​  
    // 方式三：推荐 （相较于方式二做的改进）  
    public String reverse3(String str, int start, int end) {// ArrayList list = new ArrayList(80);  
        // 1.  
        StringBuffer s = new StringBuffer(str.length());  
        // 2.  
        s.append(str.substring(0, start));// ab  
        // 3.  
        for (int i = end; i >= start; i--) {  
            s.append(str.charAt(i));  
        }  
​  
        // 4.  
        s.append(str.substring(end + 1));  
​  
        // 5.  
        return s.toString();  
​  
    }  
​  
    @Test  
    public void testReverse() {  
        String str = "abcdefg";  
        String str1 = reverse3(str, 2, 5);  
        System.out.println(str1);// abfedcg  
​  
    }
```

**题目3：** 获取一个字符串在另一个字符串中出现的次数。 比如：获取“ ab”在 “abkkcadkabkebfkabkskab” 中出现的次数

```java
 // 第3题  
    // 判断str2在str1中出现的次数  
    public int getCount(String mainStr, String subStr) {  
        if (mainStr.length() >= subStr.length()) {  
            int count = 0;  
            int index = 0;  
            // while((index = mainStr.indexOf(subStr)) != -1){  
            // count++;  
            // mainStr = mainStr.substring(index + subStr.length());  
            // }  
            // 改进：  
            while ((index = mainStr.indexOf(subStr, index)) != -1) {  
                index += subStr.length();  
                count++;  
            }  
​  
            return count;  
        } else {  
            return 0;  
        }  
​  
    }  
​  
    @Test  
    public void testGetCount() {  
        String str1 = "cdabkkcadkabkebfkabkskab";  
        String str2 = "ab";  
        int count = getCount(str1, str2);  
        System.out.println(count);  
    }
```

**题目4：** 获取两个字符串中最大相同子串。比如： str1 = "abcwerthelloyuiodef“;str2 = "cvhellobnm" 提示：将短的那个串进行长度依次递减的子串与较长的串比较。

```java
// 第4题  
    // 如果只存在一个最大长度的相同子串  
    public String getMaxSameSubString(String str1, String str2) {  
        if (str1 != null && str2 != null) {  
            String maxStr = (str1.length() > str2.length()) ? str1 : str2;  
            String minStr = (str1.length() > str2.length()) ? str2 : str1;  
​  
            int len = minStr.length();  
​  
            for (int i = 0; i < len; i++) {// 0 1 2 3 4 此层循环决定要去几个字符  
​  
                for (int x = 0, y = len - i; y <= len; x++, y++) {  
​  
                    if (maxStr.contains(minStr.substring(x, y))) {  
​  
                        return minStr.substring(x, y);  
                    }  
​  
                }  
​  
            }  
        }  
        return null;  
    }  
​  
    // 如果存在多个长度相同的最大相同子串  
    // 此时先返回String[]，后面可以用集合中的ArrayList替换，较方便  
    public String[] getMaxSameSubString1(String str1, String str2) {  
        if (str1 != null && str2 != null) {  
            StringBuffer sBuffer = new StringBuffer();  
            String maxString = (str1.length() > str2.length()) ? str1 : str2;  
            String minString = (str1.length() > str2.length()) ? str2 : str1;  
​  
            int len = minString.length();  
            for (int i = 0; i < len; i++) {  
                for (int x = 0, y = len - i; y <= len; x++, y++) {  
                    String subString = minString.substring(x, y);  
                    if (maxString.contains(subString)) {  
                        sBuffer.append(subString + ",");  
                    }  
                }  
                System.out.println(sBuffer);  
                if (sBuffer.length() != 0) {  
                    break;  
                }  
            }  
            String[] split = sBuffer.toString().replaceAll(",$", "").split("\\,");  
            return split;  
        }  
​  
        return null;  
    }  
   // 如果存在多个长度相同的最大相同子串：使用ArrayList
//	public List<String> getMaxSameSubString1(String str1, String str2) {
//		if (str1 != null && str2 != null) {
//			List<String> list = new ArrayList<String>();
//			String maxString = (str1.length() > str2.length()) ? str1 : str2;
//			String minString = (str1.length() > str2.length()) ? str2 : str1;
//
//			int len = minString.length();
//			for (int i = 0; i < len; i++) {
//				for (int x = 0, y = len - i; y <= len; x++, y++) {
//					String subString = minString.substring(x, y);
//					if (maxString.contains(subString)) {
//						list.add(subString);
//					}
//				}
//				if (list.size() != 0) {
//					break;
//				}
//			}
//			return list;
//		}
//
//		return null;
//	} 
​  
    @Test  
    public void testGetMaxSameSubString() {  
        String str1 = "abcwerthelloyuiodef";  
        String str2 = "cvhellobnmiodef";  
        String[] strs = getMaxSameSubString1(str1, str2);  
        System.out.println(Arrays.toString(strs));  
    }
```
    

**题目5：** 对字符串中字符进行自然顺序排序。 提示： 1）字符串变成字符数组。 2）对数组排序，选择，冒泡，Arrays.sort(); 3）将排序后的数组变成字符串。

```java
    // 第5题  
    @Test  
    public void testSort() {  
        String str = "abcwerthelloyuiodef";  
        char[] arr = str.toCharArray();  
        Arrays.sort(arr);  
​  
        String newStr = new String(arr);  
        System.out.println(newStr);  
    }
```
