# Java面试题

> 因本人最近有面试需求，所以收集整理了部分面试题

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