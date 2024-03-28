
函数就是定义在类中具有特定功能的一段独立的小程序。[也称为方法]  

函数的格式：  

```java
修饰符 返回值类型 函数名 (参数类型 形式参数1,参数类型 形式参数2,......)  {  
	执行语句;
	return 返回值;
}
```

```java
public class Example20 {//调用函数举例
    public static void main(String[] args) {
        //调用myprintf方法
        myprintf(3,5);
        myprintf(2,4);
        myprintf(6,10);
    }
    //接受两个参数
    public static void myprintf(int height,int width){
        //for循环打印*
        for(int i=0;i<height;i++){
            for(int j=0;j<width;j++){
                System.out.print("#");
            }
            System.out.print("\n");
        }
        System.out.println();
    }
}
```

`返回值类型`：函数运行后的结果数据类型  
`参数类型` ：形式参数的数据类型  
`形式参数` ：是一个变量，用于存储调用函数时传递给函数的实际参数  
`实际参数` ：传递给形参的具体数值  
`return` 　　 ：结束函数  
`返回值` 　 ：函数的返回结果，该结果会返回给调用者

特殊情况：功能没有具体的返回值类型时，返回值类型用关键字 void 表示，若函数的返回值void，函数中return可省略不写。

```csharp
public class FunctionDemo {//函数的简单应用
    public static void main(String[] args) {
        /*int sum=add(2,4);//定义两数
        System.out.println("sum="+sum);*/
        myprintf();//调用myprintf()方法
    }
    
     //public static 修饰符
/*    public static int add(int a,int b){
        return a+b;//返回值
    }*/
    
    //功能没有具体返回值时，返回值类型用关键字void表示
    public static void myprintf(){
        System.out.println("hello world!");
        return;//函数的返回值void，函数中return可省略
    }
}
```

函数的特点：  
1. 函数将功能进行封装  
2. 被调用才会被执行  
3. 提高代码的复用性  
4. 对于函数没有具体返回值类型时，返回类型可以用void表示，函数中return可不写

注意细节：函数只能调用函数，不可在函数内部定义函数

```csharp
//标准的错误格式
public class FunctionDemo2 {
    public static void main(String[] args) {
        System.out.println("hello world!");
    
    }
    public static void myprintf(){
        myprintf();    
    }
}
```

定义函数时，函数的结果应该返回给调用者

简单例题：

```csharp
public class FunctionDemo1 {//打印正方形
    public static void main(String[] args) {
        //调用
        getXH(3,3);
        getXH(5,6);
    }
    //定义方法
    public static void getXH(int row,int col){
        for (int x = 1; x <=row; x++) {
            for (int j = 1; j <=col; j++) {
                System.out.print("#");
            }
            System.out.print("\n");
        }
        System.out.println();
    }
}
```

```csharp
public class FunctionDemo2 {//打印乘法表
    public static void main(String[] args) {
    //    printfCFB();
        printf(4);
    }
    //指定输出（int num）
    public static void printf(int num){
        for (int i = 1; i <=num; i++) {
            for (int j = 1; j <=i; j++) {
                System.out.print(i+"*"+j+"="+i*j+"\t");
            }
            System.out.println();
        }
    }
    //标准输出
    /*public static void printfCFB(){
        for (int i = 1; i <=9; i++) {
            for (int j = 1; j <=i; j++) {
                System.out.print(i+"*"+j+"="+i*j+"\t");
            }
            System.out.println();
        }
    }*/
}
```

方法的重载：Java允许一个程序中定义多个名称相同的方法，但是`参数的类型或者个数必须不同，这就是重载  `

代码领悟：

```csharp
public class Example23{//方法的重载
    public static void main(String[] args) {
    /*
     *     //调用，方法名字各不同，getnum01、getnum02、getnum03、
    int num1=getnum01(1,3);
    int num2=getnum02(1,3,5);
    double num3=getnum03(1.2,3.8);
    //打印
    System.out.println("num1="+num1);
    System.out.println("num2="+num2);
    System.out.println("num3="+num3);
    }
    //三个不同的方法
    public static int getnum01(int x,int y){
        return x+y;
    } 
    public static int getnum02(int x,int y,int z){
        return x+y+z;
    }
    public static double getnum03(double x,double y){
        return x+y; 
    }
     */
        
/*重载代码
        求和方法调用 同个方法名getnum*/
    int num1=getnum(1,3);
    int num2=getnum(1,3,5);
    double num3=getnum(1.2,3.8);
    //打印
    System.out.println("num1="+num1);
    System.out.println("num2="+num2);
    System.out.println("num3="+num3);
    }
    //三个不同的方法
    public static int getnum(int x,int y){
        return x+y;
    } 
    public static int getnum(int x,int y,int z){
        return x+y+z;
    }
    public static double getnum(double x,double y){
        return x+y; 
    }
}
//Java允许一个程序中定义多个名称相同的方法，但是参数的类型或者个数必须不同，这就是重载
```


  