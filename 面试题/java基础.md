# Java面试题

> 因本人最近有面试需求，所以收集整理了部分面试题
>
> 从今天开始不定时路线更新。。

## Java 基础

### 1.JDK 和 JRE 有什么区别？

JDK是Java开发工具包，JRE是Java运行环境。

JDK中包含了Java的开发环境和运行环境，例如开发所使用的编译器等。所以JDK一般为Java开发人员所使用，而JRE只包含运行环境，所以如果只需要运行Java程序，可以使用JRE。

### 2.== 和 equals 的区别是什么？

**== :** 它的作用是判断两个对象的地址是不是相等。基本数据类型（byte，short，char，int，float，double，long，boolean）是以HashSet策略存储，存储在常量池中，常量池中一个常量只会对应一个地址，因此基本数据类型和String常量是可以直接通过==来直接比较的。

基本数据的包装类型（Byte, Short, Character，Integer，Float, Double，Long,  Boolean）除了Float和Double之外，其他的六种都是实现了常量池的，因此对于这些数据类型而言，一般我们也可以直接通过==来判断是否相等

> 注意：Integer 在常量池中的存储范围为[-128,127] 超过127会在堆内存创建对象

**equals**：Object类型的equals方法是直接通过==来比较的，equals是可以重写的，重写后一般比较的是值

### 3.两个对象的 hashCode()相同，则 equals()也一定为 true，对吗？

不对，首先极端的说这两个方法都可以重写，其次hashcode()是内存地址通过算法转换来的。

> 关于hashCode和equal是方法是有一些 常规协定 ：
>
> ​		1、两个对象用equals()比较返回true，那么两个对象的hashCode()方法必须返回相同的结果。
>
> ​		2、两个对象用equals()比较返回false，不要求hashCode()方法也一定返回不同的值，但是最好返回不同值，提搞哈希表性能。
>
> ​		3、重写equals()方法，必须重写hashCode()方法，以保证equals方法相等时两个对象hashcode返回相同的值。

### 4.final 在 java 中有什么作用？

final作为Java中的关键字可以用于三个地方。用于修饰类、类属性和类方法。

特征：凡是引用final关键字的地方皆不可修改！

(1)修饰类：表示该类不能被继承；

(2)修饰方法：表示方法不能被重写；

(3)修饰变量：表示变量只能一次赋值以后值不能被修改（常量）

> 当final修饰的是一个基本数据类型数据时, 这个数据的值在初始化后将不能被改变; 当final修饰的是一个引用类型数据时, 也就是修饰一个对象时, 引用在初始化后将永远指向一个内存地址, 不可修改. 但是该内存地址中保存的对象信息, 是可以进行修改的.
>
> 例如：引用类型变量指向内存中的句柄，所以对象不可以改变（句柄值是不能被修改）但是对象数据可以修改

### 5.java 中的 Math.round(-1.5) 等于多少？

**math类中提供了三个与取整有关的方法：ceil,floor,round,这些方法的作用与它们的英文名称的含义相对应。**
例如：

- ceil的英文意义是天花板，该方法就表示向上取整，math.ceil（11.3）的结果为12，math.ceil(-11.6)的结果为-11；
- floor的英文是地板，该方法就表示向下取整，math.floor(11.6)的结果是11，math.floor(-11.4)的结果-12；
- 最难掌握的是round方法，他表示“四舍五入”，算法为math.floor(x+0.5),即将原来的数字加上0.5后再向下取整，所以，math.round(11.5)的结果是12，math.round(-11.5)的结果为-11.

### 6.String 属于基础的数据类型吗？

*不属于*

1、基本数据类型
基本数据类型只有8种，可按照如下分类
①整数类型：long 64、int 32、short 16、byte 8
②浮点类型：float 32、double 64
③字符类型：char 16
④布尔类型：boolean -

2、引用数据类型
引用数据类型非常多，大致包括：
类、 接口类型、 数组类型、 枚举类型、 注解类型、 字符串型

例如，String类型就是引用类型。
简单来说，所有的非基本数据类型都是引用数据类型。

> 区别
>
> 1.存储位置	
>
> ​	基本类型存储在栈内中	
>
> ​	引用类型内容存储在堆，句柄存储在栈中
>
> 2.传递方式	
>
> ​	基本类型以数值传递会在栈中开辟新的内存空间结束后释放
>
> ​	引用数据类型按引用传递的在栈中开辟新空间指向堆的地址，执行后释放栈中内存

### 7.java 中操作字符串都有哪些类？它们之间有什么区别？

String、StringBuffer、StringBuilder

- String : final修饰，String类的方法都是返回new String。即对String对象的任何改变都不影响到原对象，对字符串的修改操作都会生成新的对象。
- StringBuffer : 对字符串的操作的方法都加了synchronized，保证线程安全。
- StringBuilder : 不保证线程安全，在方法体内需要进行字符串的修改操作，可以new StringBuilder对象，调用StringBuilder对象的append、replace、delete等方法修改字符串。

相同 都是final类,不允许被继承 StringBuffer StringBuilder操作方法类似 append，instert 

区别 

​	速度 **StringBuilder > StringBuffer > String**

​	线程安全 StringBuffer 安全 方法含 synchronized

> String 类型和 StringBuffer 类型的主要性能区别其实在于 String 是不可变的对象, 因此在每次对String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String对象，这样不仅效率低下，而且大量浪费有限的内存空间，所以经常改变内容的字符串最好不要用 String。因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，那速度是一定会相当慢的。

### 8.String str="i"与 String str=new String(“i”)一样吗？

*不一样*

**解释：**

Java 虚拟机会将其分配到常量池中：常量池不会重复创建对象。

> 在String str1="i"中，把i值存在常量池，地址赋给str1。假设再写一个String str2=“i”，则会把i的地址赋给str2，但是i对象不会重新创建，他们引用的是同一个地址值，共享同一个i内存。

分到堆内存中：堆内存会创建新的对象。

> 假设再写一个String str3=new String(“i”)，则会创建一个新的i对象，然后将新对象的地址值赋给str3。虽然str3和str1的值相同但是地址值不同。

**拓展：**

> 堆内存用来存放由new创建的对象和数组。 在堆中分配的内存，由Java虚拟机的自动垃圾回收器来管理
>
> 常量池指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。

### 9.如何将字符串反转?

```java
import java.util.Stack;  
  
/** 
 * 8 种字符串反转的方法, 其实可以是9种方法，第9种是使用StringBuffer和StringBuilder中实现的方法 
 * @author Josh Wang(Sheng) 
 *  
 * @email  swang6@ebay.com 
 *  
 */  
public class StringReverse {  
  
    /** 
     * 二分递归地将后面的字符和前面的字符连接起来。 
     *  
     * @param s 
     * @return 
     */  
    public static String reverse1(String s) {  
        int length = s.length();  
        if (length <= 1)  
            return s;  
        String left = s.substring(0, length / 2);  
        String right = s.substring(length / 2, length);  
        return reverse1(right) + reverse1(left);  
    }  
      
    /** 
     * 取得当前字符并和之前的字符append起来 
     * @param s 
     * @return 
     */  
    public static String reverse2(String s) {  
        int length = s.length();  
        String reverse = "";  
        for (int i=0; i<length; i++)  
            reverse = s.charAt(i) + reverse;  
        return reverse;  
    }  
      
    /** 
     * 将字符从后往前的append起来 
     * @param s 
     * @return 
     */  
    public static String reverse3(String s) {  
        char[] array = s.toCharArray();  
        String reverse = "";  
        for (int i = array.length - 1; i >= 0; i--) {  
            reverse += array[i];  
        }  
        return reverse;  
    }  
      
    /** 
     * 和StringBuffer()一样，都用了Java自实现的方法，使用位移来实现 
     * @param s 
     * @return 
     */  
    public static String reverse4(String s) {  
        return new StringBuilder(s).reverse().toString();  
    }  
      
    /** 
     * 和StringBuilder()一样，都用了Java自实现的方法，使用位移来实现 
     * @param s 
     * @return 
     */  
    public static String reverse5(String s) {  
        return new StringBuffer(s).reverse().toString();  
    }  
      
    /** 
     * 二分交换，将后面的字符和前面对应的那个字符交换 
     * @param s 
     * @return 
     */  
    public static String reverse6(String s) {  
        char[] array = s.toCharArray();  
        int end = s.length() - 1;  
        int halfLength = end / 2;  
        for (int i = 0; i <= halfLength; i++) {  
            char temp = array[i];  
            array[i] = array[end-i];  
            array[end-i] = temp;  
        }  
          
        return new String(array);  
    }  
      
    /** 
     * 原理是使用异或交换字符串 
     * a=a^b;  
     * b=b^a;  
         * a=b^a; 
     *  
     * @param s 
     * @return 
     */  
    public static String reverse7(String s) {  
        char[] array = s.toCharArray();  
            
          int begin = 0;  
          int end = s.length() - 1;  
            
          while (begin < end) {  
               array[begin] = (char) (array[begin] ^ array[end]);  
               array[end] = (char) (array[end] ^ array[begin]);  
               array[begin] = (char) (array[end] ^ array[begin]);  
               begin++;  
               end--;  
          }  
            
          return new String(array);  
    }  
      
    /** 
     * 基于栈先进后出的原理 
     *  
     * @param s 
     * @return 
     */  
    public static String reverse8(String s) {  
        char[] array = s.toCharArray();  
        Stack<Character> stack = new Stack<Character>();  
        for (int i = 0; i < array.length; i++)  
            stack.push(array[i]);  
  
        String reverse = "";  
        for (int i = 0; i < array.length; i++)  
            reverse += stack.pop();  
            
        return reverse;  
    }  
      
    public static void main(String[] args) {  
        System.out.println(reverse1("Wang Sheng"));  
        System.out.println(reverse2("Wang Sheng"));  
        System.out.println(reverse3("Wang Sheng"));  
        System.out.println(reverse4("Wang Sheng"));  
        System.out.println(reverse5("Wang Sheng"));  
        System.out.println(reverse6("Wang Sheng"));  
        System.out.println(reverse7("Wang Sheng"));  
        System.out.println(reverse8("Wang Sheng"));  
    }  
}  
```

### 10.String 类的常用方法都有那些？

**1、求字符串长度**

public int length()//返回该字符串的长度

```java
String str = new String("asdfzxc");
int strlength = str.length();//strlength = 7
```

**2、求字符串某一位置字符**

public char charAt(int index)//返回字符串中指定位置的字符；注意字符串中第一个字符索引是0，最后一个是length()-1。

```java
String str = new String("asdfzxc");
char ch = str.charAt(4);//ch = z
```

**3、提取子串**

用String类的substring方法可以提取字符串中的子串，该方法有两种常用参数:

1)public String substring(int beginIndex)//该方法从beginIndex位置起，从当前字符串中取出剩余的字符作为一个新的字符串返回

2)public String substring(int beginIndex, int endIndex)//该方法从beginIndex位置起，从当前字符串中取出到endIndex-1位置的字符作为一个新的字符串返回。

```java
 String str1 = new String("asdfzxc");
 String str2 = str1.substring(2);//str2 = "dfzxc"
 String str3 = str1.substring(2,5);//str3 = "dfz"
```

**4、字符串比较**

1)public int compareTo(String anotherString)//该方法是对字符串内容按字典顺序进行大小比较，通过返回的整数值指明当前字符串与参数字符串的大小关系。若当前对象比参数大则返回正整数，反之返回负整数，相等返回0。

2)public int compareToIgnore(String anotherString)//与compareTo方法相似，但忽略大小写。

3)public boolean equals(Object anotherObject)//比较当前字符串和参数字符串，在两个字符串相等的时候返回true，否则返回false。

4)public boolean equalsIgnoreCase(String anotherString)//与equals方法相似，但忽略大小写。

```java
 String str1 = new String("abc");
 String str2 = new String("ABC");
 int a = str1.compareTo(str2);//a>0
 int b = str1.compareToIgnoreCase(str2);//b=0
 boolean c = str1.equals(str2);//c=false
 boolean d = str1.equalsIgnoreCase(str2);//d=true
```

**5、字符串连接**

public String concat(String str)//将参数中的字符串str连接到当前字符串的后面，效果等价于"+"。

```java
String str = "aa".concat("bb").concat("cc");
//相当于String str = "aa"+"bb"+"cc";
```

**6、字符串中单个字符查找**

1)public int indexOf(int ch/String str)//用于查找当前字符串中字符或子串，返回字符或子串在当前字符串中从左边起首次出现的位置，若没有出现则返回-1。

2)public int indexOf(int ch/String str, int fromIndex)//改方法与第一种类似，区别在于该方法从fromIndex位置向后查找。

3)public int lastIndexOf(int ch/String str)//该方法与第一种类似，区别在于该方法从字符串的末尾位置向前查找。

4)public int lastIndexOf(int ch/String str, int fromIndex)//该方法与第二种方法类似，区别于该方法从fromIndex位置向前查找。

```java
 String str = "I am a good student";
 int a = str.indexOf('a');//a = 2
 int b = str.indexOf("good");//b = 7
 int c = str.indexOf("w",2);//c = -1
 int d = str.lastIndexOf("a");//d = 5
 int e = str.lastIndexOf("a",3);//e = 2
```

**7、字符串中字符的大小写转换**

1)public String toLowerCase()//返回将当前字符串中所有字符转换成小写后的新串

2)public String toUpperCase()//返回将当前字符串中所有字符转换成大写后的新串

```java
 String str = new String("asDF");
 String str1 = str.toLowerCase();//str1 = "asdf"
 String str2 = str.toUpperCase();//str2 = "ASDF"
```

**8、字符串中字符的替换**

1)public String replace(char oldChar, char newChar)//用字符newChar替换当前字符串中所有的oldChar字符，并返回一个新的字符串。

2)public String replaceFirst(String regex, String replacement)//该方法用字符replacement的内容替换当前字符串中遇到的第一个和字符串regex相匹配的子串，应将新的字符串返回。

3)public String replaceAll(String regex, String replacement)//该方法用字符replacement的内容替换当前字符串中遇到的所有和字符串regex相匹配的子串，应将新的字符串返回。

```java
 String str = "asdzxcasd";
 String str1 = str.replace('a','g');//str1 = "gsdzxcgsd"
 String str2 = str.replace("asd","fgh");//str2 = "fghzxcfgh"
 String str3 = str.replaceFirst("asd","fgh");//str3 = "fghzxcasd"
 String str4 = str.replaceAll("asd","fgh");//str4 = "fghzxcfgh"
```

**9、其他类方法**

1)String trim()//截去字符串两端的空格，但对于中间的空格不处理。

```java
 String str = " a sd ";
 String str1 = str.trim();
 int a = str.length();//a = 6
 int b = str1.length();//b = 4
```

2)boolean statWith(String prefix)或boolean endWith(String suffix)//用来比较当前字符串的起始字符或子字符串prefix和终止字符或子字符串suffix是否和当前字符串相同，重载方法中同时还可以指定比较的开始位置offset。

```java
 String str = "asdfgh";
 boolean a = str.statWith("as");//a = true
 boolean b = str.endWith("gh");//b = true
```

3)regionMatches(boolean b, int firstStart, String other, int otherStart, int length)//从当前字符串的firstStart位置开始比较，取长度为length的一个子字符串，other字符串从otherStart位置开始，指定另外一个长度为length的字符串，两字符串比较，当b为true时字符串不区分大小写。

4)contains(String str)//判断参数s是否被包含在字符串中，并返回一个布尔类型的值。

```java
 String str = "student";
 str.contains("stu");//true
 str.contains("ok");//false
```

5)String[] split(String str)//将str作为分隔符进行字符串分解，分解后的字字符串在字符串数组中返回。

```java
 String str = "asd!qwe|zxc#";
 String[] str1 = str.split("!|#");//str1[0] = "asd";str1[1] = "qwe";str1[2] = "zxc";
```

**同基本类型转换**

**1、字符串转换为基本类型**
java.lang包中有Byte、Short、Integer、Float、Double类的调用方法：
1)public static byte parseByte(String s)
2)public static short parseShort(String s)
3)public static short parseInt(String s)
4)public static long parseLong(String s)
5)public static float parseFloat(String s)
6)public static double parseDouble(String s)
例如：

```java
 int n = Integer.parseInt("12");
 float f = Float.parseFloat("12.34");
 double d = Double.parseDouble("1.124");
```

**2、基本类型转换为字符串类型**
String类中提供了String valueOf()放法，用作基本类型转换为字符串类型。
1)static String valueOf(char data[])
2)static String valueOf(char data[], int offset, int count)
3)static String valueOf(boolean b)
4)static String valueOf(char c)
5)static String valueOf(int i)
6)static String valueOf(long l)
7)static String valueOf(float f)
8)static String valueOf(double d)
例如：

```java
 String s1 = String.valueOf(12);
 String s1 = String.valueOf(12.34);
```

**3、进制转换**
使用Long类中的方法得到整数之间的各种进制转换的方法：
Long.toBinaryString(long l)
Long.toOctalString(long l)
Long.toHexString(long l)
Long.toString(long l, int p)//p作为任意进制

### 11.抽象类必须要有抽象方法吗？

*不需要*

抽象类不一定有抽象方法；但是包含一个抽象方法的类一定是抽象类。（有抽象方法就是抽象类，是抽象类可以没有抽象方法）

### 12.普通类和抽象类有哪些区别？

- 抽象类不能被实例化
- 抽象类可以有抽象方法，抽象方法只需申明，无需实现
- 含有抽象方法的类必须申明为抽象类
- 抽象的子类必须实现抽象类中所有抽象方法，否则这个子类也是抽象类
- 抽象方法不能被声明为静态
- 抽象方法不能用private修饰
- 抽象方法不能用final修饰

### 13.抽象类能使用 final 修饰吗？

不能，抽象类是被用于继承的，而用final修饰的类，无法被继承

### 14.接口和抽象类有什么区别？

抽象类要被子类继承，接口要被类实现。

接口只能做方法声明，抽象类中可以作方法声明，也可以做方法实现。

接口里定义的变量只能是公共的静态的常量，抽象类中的变量是普通变量。

接口是设计的结果，抽象类是重构的结果。

抽象类和接口都是用来抽象具体对象的，但是接口的抽象级别最高。

抽象类可以有具体的方法和属性，接口只能有抽象方法和不可变常量。

抽象类主要用来抽象类别，接口主要用来抽象功能。

### 15.java 中 IO 流分为几种？

- 按照流的流向分，可以分为输入流和输出流；

  > 注意：这里的输入、输出是针对程序来说的。
  > 输出：把程序(内存)中的内容输出到磁盘、光盘等存储设备中。
  >
  > 输入：读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中。

- 按照操作单元划分，可以划分为字节流和字符流；

  > 字节流：每次读取(写出)一个字节，当传输的资源文件有中文时，就会出现乱码。
  >
  > 字符流：每次读取(写出)两个字节，有中文时，使用该流就可以正确传输显示中文。

- 按照流的角色划分为节点流和处理流。

  > 节点流：从或向一个特定的地方（节点）读写数据。如FileInputStream。
  >
  > 处理流（包装流）：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如BufferedReader。处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。
  >
  > 注意：一个IO流可以既是输入流又是字节流又或是以其他方式分类的流类型，是不冲突的。比如FileInputStream，它既是输入流又是字节流还是文件节点流。

|        | 字节流       | 字符流 |
| ------ | ------------ | ------ |
| 输出流 | OutputStream | Writer |
| 输入流 | InputStream  | Reader |

如何选择

- 首先自己要知道是选择输入流还是输出流。这就要根据自己的情况决定，如果想从程序写东西到别的地方，那么就选择输入流，反之就选输出流；
- 然后考虑你传输数据时，是每次传一个字节还是两个字节，每次传输一个字节就选字节流，如果存在中文，那肯定就要选字符流了。
- 通过前面两步就可以选出一个合适的节点流了，比如字节输入流InputStream，如果要在此基础上增强功能，那么就在处理流中选择一个合适的即可。

**字节输入流 InputStream**

java.io 包下所有的字节输入流都继承自 InputStream，并且实现了其中的方法。InputStream 中提供的主要数据操作方法如下：

- int read()：从输入流中读取一个字节的二进制数据。
- int read(byte[] b)：将多个字节读到数组中，填满整个数组。
- int read(byte[] b, int off, int len)：从输入流中读取长度为 len 的数据，从数组 b 中下标为off 的位置开始放置读入的数据，读完返回读取的字节数。
- void close()：关闭数据流。
- int available()：返回目前可以从数据流中读取的字节数（但实际的读操作所读得的字节数可能大于该返回值）。
- long skip(long l)：跳过数据流中指定数量的字节不读取，返回值表示实际跳过的字节数。

对数据流中字节的读取通常是按从头到尾顺序进行的，如果需要以反方向读取，则需要使用回推（Push Back）操作。在支持回推操作的数据流中经常用到如下几个方法：

- boolean markSupported()：用于测试数据流是否支持回推操作，当一个数据流支持 mark() 和 reset()
  方法时，返回 true，否则返回 false。
- void mark(int readlimit)：用于标记数据流的当前位置，并划出一个缓冲区，其大小至少为指定参数的大小。
- void reset()：将输入流重新定位到对此流最后调用 mark() 方法时的位置。

**字节输入流 InputStream 子类**

- ByteArrayInputStream：字节数组输入流，该类的功能就是从字节数组 byte[]中进行以字节为单位的读取，也就是将资源文件都以字节形式存入到该类中的字节数组中去，我们拿数据也是从这个字节数组中拿。
- PipedInputStream：管道字节输入流，它和 PipedOutputStream 一起使用，能实现多线程间的管道通信。
- FilterInputStream：装饰者模式中充当装饰者的角色，具体的装饰者都要继承它，所以在该类的子类下都是用来装饰别的流的，也就是处理类。
- BufferedInputStream：缓冲流，对处理流进行装饰、增强，内部会有一个缓冲区，用来存放字节，每次都是将缓冲区存满然后发送，而不是一个字节或两个字节这样发送，效率更高。
- DataInputStream：数据输入流，用来装饰其他输入流，它允许通过数据流来读写Java基本类型。
- FileInputStream：文件输入流，通常用于对文件进行读取操作。
- File：对指定目录的文件进行操作。
- ObjectInputStream：对象输入流，用来提供对“基本数据或对象”的持久存储。通俗点讲，就是能直接传输Java对象（序列化、反序列化用）。

**字节输出流 OutputStream**

与字节输入流类似，java.io 包下所有字节输出流大多是从抽象类 OutputStream 继承而来的。OutputStream 提供的主要数据操作方法：

- void write(int i)：将字节 i 写入到数据流中，它只输出所读入参数的最低 8位，该方法是抽象方法，需要在其输出流子类中加以实现，然后才能使用。
- void write(byte[] b)：将数组 b 中的全部 b.length 个字节写入数据流。
- void write(byte[] b, int off, int len)：将数组 b 中从下标 off 开始的 len个字节写入数据流。元素 b[off] 是此操作写入的第一个字节，b[off + len - 1] 是此操作写入的最后一个字节。
- void close()：关闭输出流。
- void flush()：刷新此输出流并强制写出所有缓冲的输出字节。

为了加快数据传输速度，提高数据输出效率，又是输出数据流会在提交数据之前把所要输出的数据先暂时保存在内存缓冲区中，然后成批进行输出，每次传输过程都以某特定数据长度为单位进行传输，在这种方式下，数据的末尾一般都会有一部分数据由于数量不够一个批次，而存留在缓冲区里，调用 flush() 方法可以将这部分数据强制提交。

**字符输入流 Reader**

子类

- CharReader和SringReader是两种基本的介质流，它们分别将Char数组、String中读取数据。PipedReader
  是从与其它线程共用的管道中读取数据。
- BufferedReader很明显是一个装饰器，它和其他子类负责装饰其他Reader对象。
- FilterReader是所有自定义具体装饰流的父类，其子类PushBackReader对Reader对象进行装饰，会增加一个行号。
- InputStreamReader是其中最重要的一个，用来在字节输入流和字符输入流之间作为中介，可以将字节输入流转换为字符输入流。FileReader可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream 转变为Reader 的方法。

### 16.BIO、NIO、AIO 有什么区别？

- BIO 就是传统的 java.io包，它是基于流模型实现的，交互的方式是同步、阻塞方式，也就是说在读入输入流或者输出流时，在读写动作完成之前，线程会一直阻塞在那里，它们之间的调用时可靠的线性顺序。它的优点就是代码比较简单、直观；缺点就是 IO 的效率和扩展性很低，容易成为应用性能瓶颈。
- NIO 是 Java 1.4 引入的 java.nio 包，提供了 Channel、Selector、Buffer等新的抽象，可以构建多路复用的、同步非阻塞 IO 程序，同时提供了更接近操作系统底层高性能的数据操作方式。

- AIO 是 Java 1.7 之后引入的包，是 NIO 的升级版本，提供了异步非堵塞的 IO 操作方式，所以人们叫它AIO（Asynchronous IO），异步 IO是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

**1. 同步与异步**

**同步与异步**的概念，关注的是 **消息通信机制**

**同步**是指发出一个请求，在没有得到结果之前该请求就不返回结果，请求返回时，也就得到结果了。
比如洗衣服，把衣服放在洗衣机里，没有洗好之前我们一直看着， 直到洗好了才拿出来晾晒。

**异步**是指发出一个请求后，立刻得到了回应，但没有返回结果。这时我们可以再处理别的事情(发送其他请求)，所以这种方式需要我们通过状态主动查看是否有了结果, 或者可以设置一个回调来通知调用者。

比如洗衣服时，把衣服放到洗衣机里，我们就可以去做别的事情，过会儿来看看有没有洗好(通过状态查询)；或者我们设置洗衣机洗完后响铃来通知我们洗好了(回调通知)

**2. 阻塞与非阻塞**

**阻塞与非阻塞**很容易和**同步与异步**混淆，但两者关注点是不一样的。 **阻塞与非阻塞**关注的是 **程序在等待调用结果时的状态**

- **阻塞**是指请求结果返回之前，当前线程会被挂起(被阻塞)，这时线程什么也做不了
- **非阻塞**是指请求结果返回之前，当前线程没有被阻塞，仍然可以做其他事情。

**阻塞**有个明显的特征就是线程通常是处于**BLOCKED**状态(BIO中的**read()操作时，线程阻塞是JVM配合OS完成的，此时Java获取到线程的状态仍是RUNNABLE**但它确实已经被阻塞了)

如果要拿**同步**来做比较的话，同步通信方式中的线程在发送请求之后等待结果这个过程中应该处于**RUNNABLE**状态，同步必须一步一步来完成，就像是代码必须执行完一行才能执行下一行, 所以必须等待这个请求返回之后才可进行下一个请求, 即使等待结果的时间长，也是在执行这个请求的过程中。而**异步**则不用等上一条执行完, 可以先执行别的代码，等请求有了结果再来获取结果。

**3. IO模型**

Java中的IO操作是JVM配合操作系统来完成的。对于一个IO的读操作，数据会先被拷贝到操作系统内核的缓冲区中，然后从操作系统内核的缓冲区拷贝到应用程序的地址空间。所以整个过程可分为两个阶段：

1. 等待I/O数据准备好，这取决于IO目标返回数据的速度, 如网络IO时看网速和数据本身的大小。
2. 数据从内核缓冲区拷贝到进程内。

根据这两个阶段，产生了常见的几种不同的IO模型：**BIO**， **NIO**， **IO多路复用**和**AIO**。

**3.1 BIO**
**BIO**即**Blocking I/O**(阻塞 I/O)，BIO整个过程如下图：
![在这里插入图片描述](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/20200727100928591.png)

程序发送请求给内核，然后由内核去进行通信，在内核准备好数据之前这个线程是被挂起的，所以在两个阶段程序都处于挂起状态。

- BIO的特点就是在IO执行的两个阶段都被block了

**3.2 NIO**
**NIO**即**Non-Blocking I/O**(非阻塞 I/O)， NIO整个过程如下图：
![在这里插入图片描述](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/20200727101055587.png)

与BIO的明显区别是，发起第一次请求后，线程并没有被阻塞，它反复检查数据是否准备好，把原来大块不能用的阻塞时间分成了许多“小阻塞”(检查)，所以进程不断有机会被执行。这个检查有没有准备好数据的过程有点类似于“轮询”。

- NIO的特点就是程序需要不断的主动询问内核数据是否准备好。第一个阶段非阻塞，第二个阶段阻塞

**3.3 IO多路复用**

IO多路复用(**I/O Multiplexing**)有**select**，**poll**，**epoll**等不同方式，它的优点在于单个线程可以同时处理多个网络IO。

**NIO**中轮询操作是用户线程进行的，如果把这个任务交给其他线程，则用户线程就不用这么费劲的查询状态了。**IO多路复用**调用系统级别的**select**或**poll**模型，由系统进行监控IO状态。select轮询可以监控许多socket的IO请求，当有一个socket的数据准备好时就可以返回。

- select： 注册事件由数组管理, 数组是有长度的, 32位机上限1024， 64位机上限2048。轮询查找时需要遍历数组。
- poll：把select的数组采用链表实现，因此没了最大数量的限制
- epoll方式：基于事件回调机制，回调时直接通知进程，无须使用某种方式来查看状态。

多路复用IO过程图：
![在这里插入图片描述](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/2020072710135976.png)

用户线程有一段时间是阻塞的，从上图来看，与**NIO**很像，但与NIO不一样的是，select不是等到所有数据准备好才返回，而是只要有一个准备好就返回，它的优势在于可以同时处理多个连接。若连接不是很多的话，它的效率不一定高，可能还会更差。

**Java 1.4**开始支持**NIO(New IO)**，就是采用了这种方式，在套接字上提供**selector**选择机制，当发起**select()** 时会阻塞等待至少一个事件返回。

- 多路复用IO的特点是用户进程能同时等待多个IO请求，系统来监控IO状态，其中的任意一个进入读就绪状态，select函数就可以返回。

**3.4 AIO**

**AIO**即**Asynchronous I/O**(异步 I/O)，这是**Java 1.7**引入的**NIO 2.0**中用到的。整个过程中，用户线程发起一个系统调用之后无须等待，可以处理别的事情。由操作系统等待接收内容，接收后把数据拷贝到用户进程中，最后通知用户程序已经可以使用数据了，两个阶段都是非阻塞的。AIO整个过程如下图：
![在这里插入图片描述](https://gitee.com/kongyin/picture_bed/raw/master/wx_picture/20200727101800945.png)

**AIO**属于异步模型， 用户线程可以同时处理别的事情，我们怎么进一步加工处理结果呢? Java在这个模型中提供了两种方法：

- 一种是基于”回调”，我们可以实现**CompletionHandler**接口，在调用时把回调函数传递给对应的API即可
- 另一种是返回一个**Future**。处理完别的事情，可以通过**isDone()** 可查看是否已经准备好数据，通过get()方法等待返回数据。

**3.5 小结**

上面这几种模式，**BIO**整个过程都等待返回，**NIO**和**IO多路复用**在第二个阶段等待返回，因此从整个过程来看，这三个模式都属于同步方式。 **AIO**在整个过程中没有等待返回，属于异步方式。

### 17.Files的常用方法都有哪些？

- isExecutable：文件是否可以执行
- isSameFile：是否同一个文件或目录
- isReadable：是否可读
- isDirectory：是否为目录
- isHidden：是否隐藏
- isWritable：是否可写
- isRegularFile：是否为普通文件
- getPosixFilePermissions：获取POSIX文件权限，windows系统调用此方法会报错
- setPosixFilePermissions：设置POSIX文件权限
- getOwner：获取文件所属人
- setOwner：设置文件所属人
- createFile：创建文件
- newInputStream：打开新的输入流
- newOutputStream：打开新的输出流
- createDirectory：创建目录，当父目录不存在会报错
- createDirectories：创建目录，当父目录不存在会自动创建
- createTempFile：创建临时文件
- newBufferedReader：打开或创建一个带缓存的字符输入流
- probeContentType：探测文件的内容类型
- list：目录中的文件、文件夹列表
- find：查找文件
- size：文件字节数
- copy：文件复制
- lines：读出文件中的所有行
- move：移动文件位置
- exists：文件是否存在
- walk：遍历所有目录和文件
- write：向一个文件写入字节
- delete：删除文件
- getFileStore：返回文件存储区
- newByteChannel：打开或创建文件，返回一个字节通道来访问文件
- readAllLines：从一个文件读取所有行字符串
- setAttribute：设置文件属性的值
- getAttribute：获取文件属性的值
- newBufferedWriter：打开或创建一个带缓存的字符输出流
- readAllBytes：从一个文件中读取所有字节
- createTempDirectory：在特殊的目录中创建临时目录
- deleteIfExists：如果文件存在删除文件
- notExists：判断文件不存在
- getLastModifiedTime：获取文件最后修改时间属性
- setLastModifiedTime：更新文件最后修改时间属性
- newDirectoryStream：打开目录，返回可迭代该目录下的目录流
- walkFileTree：遍历文件树，可用来递归删除文件等操作

**详细介绍**

java NIO Files类(java.nio.file.Files) 提供了操作文件的相关方法。本篇文章将会覆盖大多数常用的方法。

Java7中文件IO发生了很大的变化，专门引入了很多新的类：

import java.nio.file.DirectoryStream;
import java.nio.file.FileSystem;
import java.nio.file.FileSystems;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.attribute.FileAttribute;
import java.nio.file.attribute.PosixFilePermission;
import java.nio.file.attribute.PosixFilePermissions;

…等等，来取代原来的基于java.io.File的文件IO操作方式.

**1. Path就是取代File的**

A Path represents a path that is hierarchical and composed of a sequence of directory and file name elements separated by a special separator or delimiter.

**Path用于来表示文件路径和文件**。可以有多种方法来构造一个Path对象来表示一个文件路径，或者一个文件：

1）首先是final类Paths的两个static方法，如何从一个路径字符串来构造Path对象：

```java
Path path = Paths.get("C:/", "Xmp");
Path path2 = Paths.get("C:/Xmp");
URI u = URI.create("file:///C:/Xmp/dd");        
Path p = Paths.get(u);
```

2）FileSystems构造：

```java
Path path3 = FileSystems.getDefault().getPath("C:/", "access.log");
```

3）File和Path之间的转换，File和URI之间的转换：

```java
File file = new File("C:/my.ini");
Path p1 = file.toPath();
p1.toFile();
file.toURI();
```

4）创建一个文件：

```java
Path target2 = Paths.get("C:\\mystuff.txt");
//      Set<PosixFilePermission> perms = PosixFilePermissions.fromString("rw-rw-rw-");
//      FileAttribute<Set<PosixFilePermission>> attrs = PosixFilePermissions.asFileAttribute(perms);
        try {
            if(!Files.exists(target2))
                Files.createFile(target2);
        } catch (IOException e) {
            e.printStackTrace();
        }
```

windows下不支持PosixFilePermission来指定rwx权限。

5）Files.newBufferedReader读取文件：

```java
try {
//            Charset.forName("GBK")
            BufferedReader reader = Files.newBufferedReader(Paths.get("C:\\my.ini"), StandardCharsets.UTF_8);
            String str = null;
            while((str = reader.readLine()) != null){
                System.out.println(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
```

可以看到使用 Files.newBufferedReader 远比原来的FileInputStream，然后BufferedReader包装，等操作简单的多了。

这里如果指定的字符编码不对，可能会抛出异常 MalformedInputException ，或者读取到了乱码：

```java
java.nio.charset.MalformedInputException: Input length = 1
    at java.nio.charset.CoderResult.throwException(CoderResult.java:281)
    at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:339)
    at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
    at java.io.InputStreamReader.read(InputStreamReader.java:184)
    at java.io.BufferedReader.fill(BufferedReader.java:161)
    at java.io.BufferedReader.readLine(BufferedReader.java:324)
    at java.io.BufferedReader.readLine(BufferedReader.java:389)
    at com.coin.Test.main(Test.java:79)
```

6）文件写操作：

```java
try {
            BufferedWriter writer = Files.newBufferedWriter(Paths.get("C:\\my2.ini"), StandardCharsets.UTF_8);
            writer.write("测试文件写操作");
            writer.flush();
            writer.close();
        } catch (IOException e1) {
            e1.printStackTrace();
        }
```

7）遍历一个文件夹：

```java
Path dir = Paths.get("D:\\webworkspace");
        try(DirectoryStream<Path> stream = Files.newDirectoryStream(dir)){
            for(Path e : stream){
                System.out.println(e.getFileName());
            }
        }catch(IOException e){
            
        }
try (Stream<Path> stream = Files.list(Paths.get("C:/"))){
            Iterator<Path> ite = stream.iterator();
            while(ite.hasNext()){
                Path pp = ite.next();
                System.out.println(pp.getFileName());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
```

上面是遍历单个目录，它不会遍历整个目录。遍历整个目录需要使用：Files.walkFileTree

8）遍历整个文件目录：

```java
public static void main(String[] args) throws IOException{
        Path startingDir = Paths.get("C:\\apache-tomcat-8.0.21");
        List<Path> result = new LinkedList<Path>();
        Files.walkFileTree(startingDir, new FindJavaVisitor(result));
        System.out.println("result.size()=" + result.size());        
    }
    
    private static class FindJavaVisitor extends SimpleFileVisitor<Path>{
        private List<Path> result;
        public FindJavaVisitor(List<Path> result){
            this.result = result;
        }
        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs){
            if(file.toString().endsWith(".java")){
                result.add(file.getFileName());
            }
            return FileVisitResult.CONTINUE;
        }
    }
```

来一个实际例子：

```java
public static void main(String[] args) throws IOException {
        Path startingDir = Paths.get("F:\\upload\\images");    // F:\\upload\\images\\2\\20141206
        List<Path> result = new LinkedList<Path>();
        Files.walkFileTree(startingDir, new FindJavaVisitor(result));
        System.out.println("result.size()=" + result.size()); 
        
        System.out.println("done.");
    }
    
    private static class FindJavaVisitor extends SimpleFileVisitor<Path>{
        private List<Path> result;
        public FindJavaVisitor(List<Path> result){
            this.result = result;
        }
        
        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs){
            String filePath = file.toFile().getAbsolutePath();       
            if(filePath.matches(".*_[1|2]{1}\\.(?i)(jpg|jpeg|gif|bmp|png)")){
                try {
                    Files.deleteIfExists(file);
                } catch (IOException e) {
                    e.printStackTrace();
                }
              result.add(file.getFileName());
            } return FileVisitResult.CONTINUE;
        }
    }
```

将目录下面所有符合条件的图片删除掉：filePath.matches(".*_[1|2]{1}\.(?i)(jpg|jpeg|gif|bmp|png)")

```java
public static void main(String[] args) throws IOException {
        Path startingDir = Paths.get("F:\\111111\\upload\\images");    // F:\111111\\upload\\images\\2\\20141206
        List<Path> result = new LinkedList<Path>();
        Files.walkFileTree(startingDir, new FindJavaVisitor(result));
        System.out.println("result.size()=" + result.size()); 
        
        System.out.println("done.");
    }
    
    private static class FindJavaVisitor extends SimpleFileVisitor<Path>{
        private List<Path> result;
        public FindJavaVisitor(List<Path> result){
            this.result = result;
        }
        
        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs){
            String filePath = file.toFile().getAbsolutePath();
            int width = 224;
            int height = 300;
            StringUtils.substringBeforeLast(filePath, ".");
            String newPath = StringUtils.substringBeforeLast(filePath, ".") + "_1." 
                                            + StringUtils.substringAfterLast(filePath, ".");
            try {
                ImageUtil.zoomImage(filePath, newPath, width, height);
            } catch (IOException e) {
                e.printStackTrace();
                return FileVisitResult.CONTINUE;
            }
            result.add(file.getFileName());
            return FileVisitResult.CONTINUE;
        }
    }
```

为目录下的所有图片生成指定大小的缩略图。a.jpg 则生成 a_1.jpg

**2. 强大的java.nio.file.Files**

1）创建目录和文件：

```java
try {
            Files.createDirectories(Paths.get("C://TEST"));
            if(!Files.exists(Paths.get("C://TEST")))
                    Files.createFile(Paths.get("C://TEST/test.txt"));
//            Files.createDirectories(Paths.get("C://TEST/test2.txt"));
        } catch (IOException e) {
            e.printStackTrace();
        }
```

注意创建目录和文件Files.createDirectories 和 Files.createFile不能混用，必须先有目录，才能在目录中创建文件。

2）文件复制:

从文件复制到文件：Files.copy(Path source, Path target, CopyOption options);

从输入流复制到文件：Files.copy(InputStream in, Path target, CopyOption options);

从文件复制到输出流：Files.copy(Path source, OutputStream out);

```java
try {
            Files.createDirectories(Paths.get("C://TEST"));
            if(!Files.exists(Paths.get("C://TEST")))
                    Files.createFile(Paths.get("C://TEST/test.txt"));
//          Files.createDirectories(Paths.get("C://TEST/test2.txt"));
            Files.copy(Paths.get("C://my.ini"), System.out);
            Files.copy(Paths.get("C://my.ini"), Paths.get("C://my2.ini"), StandardCopyOption.REPLACE_EXISTING);
            Files.copy(System.in, Paths.get("C://my3.ini"), StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            e.printStackTrace();
        }
```

3）遍历一个目录和文件夹上面已经介绍了：Files.newDirectoryStream ， Files.walkFileTree

4）读取文件属性：

```java
Path zip = Paths.get(uri);
System.out.println(Files.getLastModifiedTime(zip));
System.out.println(Files.size(zip));
System.out.println(Files.isSymbolicLink(zip));
System.out.println(Files.isDirectory(zip));
System.out.println(Files.readAttributes(zip, "*"));
```

5）读取和设置文件权限：

```java
Path profile = Paths.get("/home/digdeep/.profile");
            PosixFileAttributes attrs = Files.readAttributes(profile, PosixFileAttributes.class);// 读取文件的权限
            Set<PosixFilePermission> posixPermissions = attrs.permissions();
            posixPermissions.clear();
            String owner = attrs.owner().getName();
            String perms = PosixFilePermissions.toString(posixPermissions);
            System.out.format("%s %s%n", owner, perms);
            
            posixPermissions.add(PosixFilePermission.OWNER_READ);
            posixPermissions.add(PosixFilePermission.GROUP_READ);
            posixPermissions.add(PosixFilePermission.OTHERS_READ);
            posixPermissions.add(PosixFilePermission.OWNER_WRITE);
            
            Files.setPosixFilePermissions(profile, posixPermissions);    // 设置文件的权限
```

Files类简直强大的一塌糊涂，几乎所有文件和目录的相关属性，操作都有想要的api来支持。这里懒得再继续介绍了，详细参见 jdk8 的文档。

一个实际例子：

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class StringTools {
    public static void main(String[] args) {
        try {
            BufferedReader reader = Files.newBufferedReader(Paths.get("C:\\Members.sql"), StandardCharsets.UTF_8);
            BufferedWriter writer = Files.newBufferedWriter(Paths.get("C:\\Members3.txt"), StandardCharsets.UTF_8);

            String str = null;
            while ((str = reader.readLine()) != null) {
                if (str != null && str.indexOf(", CAST(0x") != -1 && str.indexOf("AS DateTime)") != -1) {
                    String newStr = str.substring(0, str.indexOf(", CAST(0x")) + ")";
                    writer.write(newStr);
                    writer.newLine();
                }
            }
            writer.flush();
            writer.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

场景是，sql server导出数据时，会将 datatime 导成16进制的binary格式，形如：, CAST(0x0000A2A500FC2E4F AS DateTime))

所以上面的程序是将最后一个 datatime 字段导出的 , CAST(0x0000A2A500FC2E4F AS DateTime) 删除掉，生成新的不含有datetime字段值的sql 脚本。用来导入到mysql中。

做到半途，其实有更好的方法，使用sql yog可以很灵活的将sql server中的表以及数据导入到mysql中。使用sql server自带的导出数据的功能，反而不好处理。