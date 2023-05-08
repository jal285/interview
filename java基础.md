# java_基础

<!-- more -->

<style>
     img{
        box-shadow:  0px 0px 10px  rgba(0,0,0,.5);
                 /*图片添加阴影*/ 
    }
</style>

## 基础知识

### 内置数据类型

| 类型      | 占用字节 | 占用位数 |
| ------- | ---- | ---- |
| byte    | 1    | 8    |
| short   | 2    | 16   |
| int     | 4    | 32   |
| long    | 8    | 64   |
| float   | 4    | 32   |
| double  | 8    | 64   |
| char    | 2    | 16   |
| boolean | 1    | 8    |

byte、boolean (1字节) - short、char（2字节) - int、flot（4字节）- long 、double（8字节） 

Java语言提供了八种基本类型。六种数字类型（四个整数型（默认是int 型），两个浮点型（默认是double 型）），一种字符类型，还有一种布尔型。

byte：

- byte数据类型是8位、有符号的，以二进制补码表示的整数；（256个数字），占1字节
- 最小值是-128（-2^7）；
- 最大值是127（2^7-1）；
- 默认值是0；
- byte类型用在大型数组中节约空间，主要代替整数，因为byte变量占用的空间只有int类型的四分之一；
- 例子：byte a = 100，byte b = -50。

short：

- short数据类型是16位、有符号的以二进制补码表示的整数，占2字节
- 最小值是-32768（-2^15）；
- 最大值是32767（2^15 - 1）；
- Short数据类型也可以像byte那样节省空间。一个short变量是int型变量所占空间的二分之一；
- 默认值是0；
- 例子：short s = 1000，short r = -20000。

int：

- int数据类型是32位、有符号的以二进制补码表示的整数；占4字节
- 最小值是-2,147,483,648（-2^31）；
- 最大值是2,147,485,647（2^31 - 1）；
- 一般地整型变量默认为int类型；
- 默认值是0；
- 例子：int a = 100000, int b = -200000。

long：

- long数据类型是64位、有符号的以二进制补码表示的整数；占8字节
- 最小值是-9,223,372,036,854,775,808（-2^63）；
- 最大值是9,223,372,036,854,775,807（2^63 -1）；
- ·这种类型主要使用在需要比较大整数的系统上；
- 默认值是0L；
- 例子： long a = 100000L，int b = -200000L。

long a=111111111111111111111111(错误，整数型变量默认是int型)

long a=111111111111111111111111L(正确，强制转换)

float：

- float数据类型是单精度、32位、符合IEEE 754标准的浮点数；占4字节   -3.4*E38- 3.4*E38。。。浮点数是有舍入误差的
- float在储存大型浮点数组的时候可节省内存空间；
- 默认值是0.0f；
- 浮点数不能用来表示精确的值，如货币；
- 例子：float f1 = 234.5f。
- float f=6.26(错误  浮点数默认类型是double类型)
- float f=6.26F（转换正确，强制）
- double d=4.55(正确)

double：

- double数据类型是双精度、64位、符合IEEE 754标准的浮点数；
- 浮点数的默认类型为double类型；
- double类型同样不能表示精确的值，如货币；
- 默认值是0.0d；
- 例子：double d1 = 123.4。

boolean：

- boolean数据类型表示一位的信息；
- 只有两个取值：true和false；
- 这种类型只作为一种标志来记录true/false情况；
- 默认值是false；
- 例子：boolean one = true。

char：

- char类型是一个单一的16位Unicode字符；用 ‘’表示一个字符。。java 内部使用Unicode字符集。。他有一些转义字符  ，2字节
- 最小值是’\u0000’（即为0）；
- 最大值是’\uffff’（即为65,535）；可以当整数来用，它的每一个字符都对应一个数字

### 关键字

```java
关键字: 纯小写; 带颜色; 

三元运算符

a, b , max

max = a>b? a:b   true 时为a ; flase 为 b;

switch 后面的小括号当中只能是下列数据类型:

 基本数据类型:  byte/short/char/int

引用数据类型: String 字符串. enum枚举    

当switch 语句匹配到对应的case时 , 如果当前语句没有break 语句 , 程序会继续向下执行, 知遇到break或整体结束为止 ; 一般加上default 语句

object.toString() 打印当前对象object的地址

当用equals() 方法进行比较时, 对类file , string , Date及包装类(Wrappering Class) 来说, 是比较类型
及内容而不考虑引用的是否是同一个对象
使用 == 时 ,比较这些特殊的类比较的是对象(对象的地址)

原因: 在这些类中, 重写了equals方法
```

![image-20210523172538568](../images/java_learing_temp-writing/image-20210523172538568.png)

<!--字面量创建对象的时候, 旨在常量池创建一个对象 , new的时候 常量池和堆中都有对象-->

<!--'=='比较的是内存地址 -->

#### final

final关键字可以用来修饰类，方法和变量（包括成员变量和局部变量）

当用final修饰一个类时，表明这个类不能被继承。也就是说，如果一个类你永远不会让他被继承，就可以用final进行修饰。final类中的成员变量可以根据需要设为final，但是要注意final类中的所有成员方法都会被隐式地指定为final方法。

在使用final修饰类的时候，要注意谨慎选择，除非这个类真的在以后不会用来继承或者出于安全的考虑，尽量不要将类设计为final类。

final类中的所有成员方法都会被隐式地指为fianl方法

final变量的数值初始化之后便不能更改，如果是引用类型的变量，则对其初始化之后便不能让其再指向另一个对象，final变量一般在定义时或者构造器中进行初始化赋值

##### 修饰方法

　　下面这段话摘自《Java编程思想》第四版第143页：

　　“使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。“

　　因此，如果只有在想明确禁止 该方法在子类中被覆盖的情况下才将方法设置为final的。

#### static

static方法修饰的的成员不再属于某个对象，而是属于它所在的类，只需要通过类名就能访问，不需要消耗资源反复创建对象，在类第一次加载的时候，static就已经在内存当中，知道程序结束后，该内存才释放

如果这个方法是作为一个工具来使用的，就声明为static，不需要new一个对象就可以使用。比如：connect DB就可以声明一个Connection()的static方法，

因为是静态的，说明connection DB不是某个对象所特有的功能，只是作为一种连接到DB的工具。

#### abstract

abstract 关键字用于类和方法 ，定义抽象类和抽象方法， 

抽象类可以同时具有抽象方法和常规方法， 抽象类不可以创建对象

#### native

方法前加native关键字则该方法为原生方法，即原生函数，这个方法是用c/c++语言实现的，并且被编译成了DLL，由java区调用

#### transient

对于不想进行序列化的变量，使用transient进行修饰

瞬态关键字的作用是：阻止实例中那些用此关键字修饰的变量序列化；当对象被反序列化时，被瞬态修饰的变量值不会被持久化和恢复。transient只能修饰变量，不能修饰类和方法。

### 基础方法和类

#### ObjectUtils.isEmpty()

```java
import org.springframework.util.ObjectUtils; 
```

 涵盖了几乎所有的Java对象 

#### getInstance

主函数当中使用此类的getInstance()函数，即可得到系统当前已经实例化的该类对象，若当前系统还没有实例化过这个类的对象，则调用此类的构造函数

对于抽象类，想对其实例化，只能用getInstance()    方法

getInstance()方法即是单例模式，是一种对于方法的引用，相当于c++里面的指针 

getInstance()方法生成的是一个实例对象，也是调用这个方法类的唯一实例

#### StringBuffer 和 StringBuilder

`String`中的对象是不可变的，也就可以理解为常量，线程安全。`AbstractStringBuilder`的英文`StringBuilder`与`StringBuffer`的公共父类，定义了一些字符串的基本操作，如`expandCapacity`，`append`，`insert`，`indexOf`等公共方法。`StringBuffer`对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder`并没有对方法进行加同步锁，所以是非线程安全的。

**对于三者使用的总结：**

1. 操作少量的数据：适用 `String`
2. 单线程操作字符串刑事下操作大量数据：适用 `StringBuilder`
3. 多线程操作字符串司法下操作大量数据：适用 `StringBuffe`

#### "=="于equals

**==**：它的作用是判断两个对象的地址是不是相等即，判断两个对象是不是同一个对象（基本数据类型==比较的是值，引用数据类型==比较的是内存地址） 。

**equals（）**：它的作用也是判断两个对象是否正确。但它一般有两种使用情况：

- 情况1：类没有覆盖equals（）方法。则通过equals（）比较该类的两个对象时，等价于通过“ ==”比较这两个对象。
- 情况2：类覆盖了equals（）方法。一般，我们都覆盖equals（）方法来比较两个对象的内容是否替换；若它们的内容有所不同，则返回true（即，认为这两个对象相似）。

注：String中的qeuals方法是被重写过的，因为obejct的equals方法比较的是对象的内存地址，而String的equals方法比较的是对象的值

#### 从键盘输入

```java
char i = (char)system.out.in.read()  //只能获取单个char类型字符
//可以获取字符串但int float类型仍然要转换
InputStreamReader is = new InputStreamReader(System.in); //new构造InputStreamReader对象 
BufferedReader br = new BufferedReader(is); //拿构造的方法传到BufferedReader中 
try{ //该方法中有个IOExcepiton需要捕获 
      String name = br.readLine(); 
      System.out.println("ReadTest Output:" + name); 
    } 
    catch(IOException e){ 
      e.printStackTrace(); 
    } 
public static void ScannerTest(){ 
    Scanner sc = new Scanner(System.in); 
    System.out.println("ScannerTest, Please Enter Name:"); 
    String name = sc.nextLine();  //读取字符串型输入 
    System.out.println("ScannerTest, Please Enter Age:"); 
    int age = sc.nextInt();    //读取整型输入 
    System.out.println("ScannerTest, Please Enter Salary:"); 
    float salary = sc.nextFloat(); //读取float型输入 
    System.out.println("Your Information is as below:"); 
    System.out.println("Name:" + name +"\n" + "Age:"+age + "\n"+"Salary:"+salary); 
    sc.close();
```

通过BufferedReader

```java
BufferedReader input = new BufferedReader(new InputStreamReader(System.in)); 
String s = input.readLine();
```

### string

#### substring

一个参数, 截取 参数到末尾片段

s.substring(n)  --  截取n到末尾的字符 

s.substring(0,n) --截取0到n之间的字符 

### 数组

#### 使用流获得数组最大值或最小值

将数组放进stream ,然后直接调用stream里的min或max函数得到最小值或最大值

```java
 public int getMin(int[] nums) {
        int minNum=Arrays.stream(nums).min().getAsInt();
}
```

#### 使用collection

将数组转化为对象数组, int 转化为Integer

```java
@Test
    public void index3(){
        int ages[] = {18 ,23 ,21 ,19 ,25 ,29 ,17};
        Integer newAges[] = new Integer[ages.length];

        for(int i=0;i<ages.length;i++) {
            newAges[i] = (Integer)ages[i];
        }
        int maxNum = Collections.max(Arrays.asList(newAges));
        System.out.println("最大值为："+ maxNum);
    }
```

### 集合

Collection 接口是接口层次结构中的根节点

| HashMap      | **TreeMap** | **LinkedHashMap** | **HashTable** |
|:------------ | ----------- | ----------------- | ------------- |
| 底层是哈希表：查询速度快 | 底层是红黑树      | 底层是哈希表+链表         | 被HashMap取代    |
| key无序且不重复    | key排序和去重    | key有序             | key无序且不重复     |
| 多线程，不安全，效率高  | 多线程，不安全，效率高 | 多线程，不安全，效率高       | 线程安全，效率低      |

#### List 使用

#### list转为数组 []

例 : List<Integer> 转为 int[]

```java
list.stream().mapToInt(Integer::intValue).toArray();
```

##### size()

获得列表中元素个数 , 但list的下标从 0 开始, 所以list中最后一个元素下标应该为 list.size() -1 

##### ArrayList

容量能动态增长的动态数组, 继承AbstractList , 实现了List, RandomAcess , `Cloneable`, `java.io.Serializable`。

Array的操作线程不安全, 单线程中 才使用ArrayList , 多线程中一般用 Vector 或者copyonwriteArrayList

迭代器遍历(最低)

```java
Iterator<Integer> it=arrayList.iterator();
while(it.hashNext()){
    System.out.println(it.next()+" ");
}
```

索引值 (效率第一)

```java
for(int i=0;i<arrayList.size(); i++){
    System.out.println(arrayList.get(i)+ " ");
}
```

for循环 (第二)

```java
for(Integet:number:arryList){
    System.sout.println(number+" ");
}
```

##### toArray()

有时候，当我们调用ArrayList中的 toArray()，可能遇到过抛出java.lang.ClassCastException异常的情况，这是由于toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(如，将Object[]转换为的Integer[])则会抛出java.lang.ClassCastException异常，因为Java不支持向下转型。
所以一般更常用的是使用另外一种方法进行使用：

<T> T[] toArray(T[] a)

调用toArray(T[] a)返回T[] 可通过以下方式:

```
//第一种(常用)
Integer[] integer = arrayList.toArray(new Integer[0])

//第二种(容易理解)
Integer[] integer1=new Integer[arrayList/size()];
arrayList.toAttay[integer1]

 // 抛出异常，java不支持向下转型
 //Integer[] integer2 = new Integer[arrayList.size()];
 //integer2 = arrayList.toArray();
```

##### Arrays.asList()

将一个数组转换为一个List集合

![阿里巴巴Java开发手-Arrays.asList()方法](https://camo.githubusercontent.com/4734fe14a47bd609efc67807246641c9f1830337d1b45351a32d4a40337b494b/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545392539382542462545392538372538432545352542372542342545352542372542344a6176612545352542432538302545352538462539312545362538392538422d4172726179732e61734c69737428292545362539362542392545362542332539352e706e67)

将数组转为ArrayList<Element> arraylist 时可以用Collection.addAll(arrayList,arry) 转换避免两个使用asList() 的坑, 或者使用guava(goole) 的list.newArrayList(array)

1. 这样做生成的list, 是定长的, 也就是说, 如果你对它做add 或者remove , 都会抛 UnsupportedOperationException
2. 如果修改数组的值, list 中的对应值也会改变! 

#### stream 排序

[参考](https://blog.csdn.net/LLF_1241352445/article/details/81002477)

#### 映射

键值对，A--a ,     B--b

多对一，一对多 

### LinkedList

使用链表形成的线性表 

​    <img src="../images/java_learing_temp-writing/image-20210523153807322.png" alt="image-20210523153807322" style="zoom:80%;" />

<img src="../images/java_learing_temp-writing/image-20210523153822719.png" alt="image-20210523153822719" style="zoom:200%;" />

**以下情况使用 LinkedList :**

- 你需要通过循环迭代来访问列表中的某些元素。
- 需要频繁的在列表开头、中间、末尾等位置进行添加和删除元素操作。

LinkedList 继承了 AbstractSequentialList 类。

LinkedList 实现了 Queue 接口，可作为队列使用。

LinkedList 实现了 List 接口，可进行列表的相关操作。

LinkedList 实现了 Deque 接口，可作为队列使用。

LinkedList 实现了 Cloneable 接口，可实现克隆。

LinkedList 实现了 java.io.Serializable 接口，即可支持序列化，能通过序列化去传输。

### HashMap

**Java**中的**HashMap在**实现**Map**接口的集合类中。它用于存储**键和值**对。每个键都映射到映射中的单个值。

键是唯一的。这意味着我们只能在地图中插入键“ K”一次。不允许重复的密钥。虽然一个值`'V'`可以映射到多个键。

```java
public` `class` `HashMap<K,V> ``extends` `AbstractMap<K,V> ``        ``implements` `Map<K,V>, Cloneable, Serializable 
```

**功能**

- HashMap不能包含重复的键。
- HashMap允许多个`null`值，但只有一个`null`键。
- HashMap是一个**无序集合**。它不保证元素的任何特定顺序。
- HashMap**不是线程安全的**。您必须显式同步对HashMap的并发修改。或者，您可以使用**Collections.synchronizedMap（hashMap）**来获取HashMap的同步版本。
- 只能使用关联的键来检索值。
- HashMap仅存储对象引用。因此，必须将原语与其对应的包装器类一起使用。如`int`
- 将存储为`Integer`。
- HashMap实现了**Cloneable**和**Serializable**接口。

**内部实现**

HashMap遵循哈希原理。在将任何公式/算法应用于其属性之后，散列是一种为任何变量/对象分配唯一代码的方法。Java中的每个对象都有其**哈希码**，使得两个相等的对象必须一致地产生相同的哈希码。

![在这里插入图片描述](../images/java_learing_temp-writing/16bb3603ff3bdae1.png)

#### hashmap

**去重原理** 与HashSet相同

首先比较元素的hashCode()哈希值是否存在，如果不存在，那么将元素存储，如果存在，再通过equals()方法表元素内容是否一致，如果一致不存储，不一致则存储。HashSet底层使用的HashMap，只取了hashMap的key。

**注意**：jdk提供的数据类型String，Integer等类都重写了hashCode和equals方法，所以当这些数据存入HashSet集合中时都会去重，**但是如果存储自定的类型的数据需要去重的话，就必须重写hashCode()和equals()方法**。比如：hashSet集合中要存储Student对象（名字和年龄都相同就是同一个人），且需要去重，那么就需要Student类重写hashCode和equals方法。

#### LinkedHashMap

继承自HashMap  ，底层为哈希表+链表 ，key有序

#### java8中的HashMap改进

作为[JEP 180](https://openjdk.java.net/jeps/180)的工作的一部分，通过使用平衡树而不是链接列表来存储地图条目，HashMap对象的性能得到了提高，其中键中存在很多冲突。主要思想是，**一旦哈希存储桶中的项目数超过某个阈值，该存储桶就会从使用条目的链接列表切换到平衡树。在哈希冲突较高的情况下，这会将最坏情况的性能从提高`O(n)`到`O(log n)`**。

基本上，当存储桶太大时（**当前：TREEIFY_THRESHOLD = 8**），HashMap用树形图的临时实现动态替换它。这样一来，我们不会感到悲观的O（n），而是获得了更好的O（log n）。

可以遍历和使用TreeNodes的bin（元素或节点），但在填充过多时，还支持更快的查找。但是，由于正常使用中的绝大多数垃圾箱都没有人口过多，因此在使用表格方法的过程中，检查是否存在树木垃圾箱可能会延迟。

树箱（即，其元素均为TreeNode的箱）主要通过hashCode进行排序，但在绑定的情况下，如果两个元素的名称相同*`class C implements Comparable<C>`*，则使用其`compareTo()`方法进行排序。

因为TreeNode的大小约为常规节点的两倍，所以我们仅在垃圾箱包含足够的节点时才使用它们。并且当它们变得太小（由于删除或调整大小）时，它们会转换回普通箱（**当前：UNTREEIFY_THRESHOLD = 6**）。在使用具有良好分布的用户hashCode的用法中，很少使用树箱。

### LinkedHashMap

### 反射

反射是框架设计的灵魂

java 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能成为java语言的反射机制

[参考](https://snailclimb.gitee.io/javaguide/#/docs/java/basis/%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6?id=%e5%8f%8d%e5%b0%84%e6%9c%ba%e5%88%b6%e4%bb%8b%e7%bb%8d)

![img](../images/java_learing_temp-writing/20170513133210763.png)

#### 反射中使用的一些方法

##### newInstance

Class 类 和 Consturctor类的newInstance（）方法用于创建该类的新实例

##### invoke（）方法

java反射提供invoke（）方法，在运行时根据业务需要调用相应的方法，在运行时非常常见，主要通过反射获取到方法名之后，就可以调用对应的方法：

```java
Class birdClass = Bird.class;
Constructor constructors1 = birdClass.getConstructor();
Method eatMetchod = birdClass.getMethod("eat", new Class[]{int.class});
System.out.println(eatMetchod.invoke(constructors1.newInstance(), 2));
```

invoke方法有两个参数，第一个参数是要调用方法的对象，上面的代码中就是Bird的对象，第二个参数是调用方法要传入的参数。如果有多个参数，则用数组。

如果调用的是static方法，invoke()方法第一个参数就用null代替

##### 对反射类

getMethods()获取所有public 修饰的方法，返回地为Method[]数组 （Method[] metchods = Class.getMethods();）

getDeclaredMethods()方法到的是所有的成员方法，和修饰符无关。

getName()方法获取的类名包含包信息。

getSimpleName()方法只是获取类名，不包含包信息。

getPackage()获取包名 

getSuperclass()获取父类class对象，返回父亲的class对象

getInterfaces()，获取接口。一个类可以实现多个接口，所以getInterfaces()方法返回的是Class[]数组。 注意：getInterfaces()只返回指定类实现的接口，不会返父类实现的接口。

Constructor()，获取构造函数，一个类会有多个构造函数，getConstructors()返回的是Constructor[]数组，包含了所有声明的用public修饰的构造函数。（1、只能获取到public修饰的构造函数。
2、需要捕获NoSuchMethodException异常。）

getParameterTypes()获取到构造函数的参数。(constructors.getParameterTypes();  )

getFields()方法获取所有public修饰的成员变量，getField()方法需要传入变量名，并且变量必须是public修饰符修饰。 (使用需要创建Field对象，例： Field[] fields1 = birdClass.getFields();)

getDeclaredFields方法获取所有生命的成员变量，不管是public还是private。

get()获取变量值， set()给变量赋值

使用 Class.getDeclaredField(String name)或者Class.getDeclaredFields()才能获取到私有变量。（使用前：必须调用**setAccessible(true)**方法，这是针对私有变量而言，public和protected等都不需要。这个方法是允许通过反射访问类的私有变量。

class.getAnnotations()访问类的所有注解信息

class.getAnnotation(MyAnnotation.class) 访问类特定的注解信息

method.getDeclaredAnnotations(),访问方法注解信息(需要先创建Mehtod 对象)

#### java反射机制提供的主要功能

1、在运行时判断任意一个对象所属的类

2、在运行时构造任意一个类的对象

3、在运行时判断任意一个类所具有的成员变量和方法

4、在运行时调用任意一个对象的方法

5、生成动态代理

##### 动态代理

使用反射可以在运行时创建接口的动态实现，java.lang.reflect.Proxy类提供了创建动态实现的功能。我们把运行时创建接口的动态实现称为动态代理。

动态代理可以用于许多不同的目的，例如数据库连接和事务管理、用于单元测试的动态模拟对象以及其他类似aop的方法拦截等。

创建代理

调用java.lang.reflect.Proxy类的newProxyInstance()方法就可以常见动态代理，newProxyInstance()方法有三个参数： 1、用于“加载”动态代理类的类加载器。
2、要实现的接口数组。 3、将代理上的所有方法调用转发到InvocationHandler的对象。 代码如下：

```java
InvocationHandler handler = new MyInvocationHandler();
MyInterface proxy = (MyInterface) Proxy.newProxyInstance(
                            MyInterface.class.getClassLoader(),
                            new Class[] { MyInterface.class },
                            handler);
```

运行上面代码后，proxy变量包含了MyInterface接口的动态实现。

对代理的所有调用都将由到实现了InvocationHandler接口的handler 对象来处理。

动态代理使用场景：
1、数据库连接和事务管理。例如Spring框架有一个事务代理，可以启动和提交/回滚事务
2、用于单元测试的动态模拟对象
3、类似AOP的方法拦截。

### 循环

```java
label1：

​        外部循环{

​        内部循环{

//... 

break; // 1

//...

continue; //2

//...

cintinue label11;//3

//...

break label1; //4

}

}
```

在条件1 中，break 中断内部循环，并在外部循环结束。在条件2 中，continue 移回内部循环的起始处。但
在条件3 中，continue label1 却同时中断内部循环以及外部循环，并移至label1 处。随后，它实际是继续
循环，但却从外部循环开始。在条件4 中，break label1 也会中断所有循环，并回到label1 处，但并不重
新进入循环。也就是说，它实际是完全中止了两个循环。

### Lambda

#### a single parameter and an expression

parameter -> expression 

#### more than one parameter , wrap them in parentheses

(parameter1,parameter2) -> expression 

```java
import java.util.ArrayList;

public class Main {
  public static void main(String[] args) {
    ArrayList<Integer> numbers = new ArrayList<Integer>();
    numbers.add(5);
    numbers.add(9);
    numbers.add(8);
    numbers.add(1);
    numbers.forEach( (n) -> { System.out.println(n); } );
  }
}
```

  ,Lambda 表达式可以存储在接口 变量中 . lambda表达数应具有相同数量的 参数和该方法相同的返回值类型 

函数式接口 Consumer

```java
import java.util.ArrayList;
import java.util.function.Consumer;

public class Main {
  public static void main(String[] args) {
    ArrayList<Integer> numbers = new ArrayList<Integer>();
    numbers.add(5);
    numbers.add(9);
    numbers.add(8);
    numbers.add(1);
    Consumer<Integer> method = (n) -> { System.out.println(n); };
    numbers.forEach( method );
  }
}
```

上面程序定义了一个 method函数变量, 通过 method 可以调用 lambda表达式中的 System.out.println(n) , 变量为 n

### CGLB动态代理机制

**JDK 动态代理有一个最致命的问题是其只能代理实现了接口的类。**

**为了解决这个问题，我们可以用 CGLIB 动态代理机制来避免。**

[CGLIB](https://github.com/cglib/cglib)(*Code Generation Library*)是一个基于[ASM](http://www.baeldung.com/java-asm)的字节码生成库，它允许我们在运行时对字节码进行修改和动态生成。CGLIB 通过继承方式实现代理。很多知名的开源框架都使用到了[CGLIB](https://github.com/cglib/cglib)， 例如 Spring 中的 AOP 模块中：如果目标对象实现了接口，则默认采用 JDK 动态代理，否则采用 CGLIB 动态代理。

**在 CGLIB 动态代理机制中 `MethodInterceptor` 接口和 `Enhancer` 类是核心。**

你需要自定义 `MethodInterceptor` 并重写 `intercept` 方法，`intercept` 用于拦截增强被代理类的方法。

通过Enhancer类对象设置类加载器和 被代理类,  返回代理类对象  

#### JDK和CGLB动态代理对比

1. **JDK 动态代理只能只能代理实现了接口的类或者直接代理接口，而 CGLIB 可以代理未实现任何接口的类。** 另外， CGLIB 动态代理是通过生成一个被代理类的子类来拦截被代理类的方法调用，因此不能代理声明为 final 类型的类和方法。
2. 就二者的效率来说，大部分情况都是 JDK 动态代理更优秀，随着 JDK 版本的升级，这个优势更加明显。

### 回调

 A类 调用 B类 方法 B类调用A类方法 

 callback 函数  

#### 直接调用与间接调用

- 直接调用
    在函数A的函数体里，通过书写函数B的函数名来调用之，使内存中对应函数B的代码得以执行。
    这里，A称为“主叫函数”（Caller），B称为“被叫函数”（Callee）。
- 间接调用
    在函数A的函数体里并不出现函数B的函数名，而是通过指向函数B的函数指针p，来使内存中属于函数B的代码片断得以执行。 
    Java中的简介调用是使用接口来完成的

通过使用异步计算，在结果计算完成时进行回调即可 

### 对象的深拷贝

 Deep Clone

 重写cone方法里,被拷贝对象里面存在着另一个对象的引用时, 也就是实例变量, 需要将里面的new出一个新的拥有相同数据的对象赋给实例变量, 否则会因为 拷贝者和被拷贝者共享一个实例变量, son1, son2

造成 . son1.GetFather = son2.GetFather (fahter属性为一个实例类型)

clone的拷贝规则如下 

1. **基本类型：如果变量是基本类型，则拷贝其值。比如int、float等**
2. **对 象：如果变量是一个实例对象，则拷贝其地址引用，也就是说此时拷贝出的对象与原有对象共享该实例变量，不受访问权限的控制，这在Java中是很疯狂的，因 为它突破了访问权限的定义：一个private修饰的变量，竟然可以被两个不同的实例对象访问，这让java的访问权限体系情何以堪。**
3. **String字符串：这个比较特殊，拷贝的也是一个地址，是个引用，但是在修改时，它会从字符串池(String pool)中重新生成新的字符串，原有的字符串对象保持不变，在此处我们可以认为String是一个基本类型。**

### 泛型

<T> 和 Class <T>的使用 .泛型类 

ArrayList 就是泛型 , 将类型不同(例 String , Int ), 格式相同的代码形成模板

例 

```java
class Point<T>{// 此处可以随便写标识符号   
    private T x ;        
    private T y ;        
    public void setX(T x){//作为参数  
        this.x = x ;  
    }  
    public void setY(T y){  
        this.y = y ;  
    }  
    public T getX(){//作为返回值  
        return this.x ;  
    }  
    public T getY(){  
        return this.y ;  
    }  
};  
//IntegerPoint使用  
Point<Integer> p = new Point<Integer>() ;   
p.setX(new Integer(100)) ;   
System.out.println(p.getX());    

//FloatPoint使用  
Point<Float> p = new Point<Float>() ;   
p.setX(new Float(100.12f)) ;   
System.out.println(p.getX());    
```

T 定义为模糊类型  , T 可以是任何大写字母, T表示派生自Object 类的任何类(一定 ) 

多泛型变量 Class<T,K,V,E,> 

比较规范的 定义 

- E — Element，常用在java Collection里，如：List<E>,Iterator<E>,Set<E>
- K,V — Key，Value，代表Map的键值对
- N — Number，数字
- T — Type，类型，如String，Integer等等

如果这些还不够用，那就自己随便取吧，反正26个英文

#### 泛型接口

```java
interface Info<T>{        // 在接口上定义泛型    
    public T getVar() ; // 定义抽象方法，抽象方法的返回值就是泛型类型    
    public void setVar(T x);  
}    
```

非泛型类

```java
class InfoImpl implements Info<String>{   // 定义泛型接口的子类  
    private String var ;                // 定义属性  
    public InfoImpl(String var){        // 通过构造方法设置属性内容  
        this.setVar(var) ;  
    }  
    @Override  
    public void setVar(String var){  
        this.var = var ;  
    }  
    @Override  
    public String getVar(){  
        return this.var ;  
    }  
}  

public class GenericsDemo24{  
    public  void main(String arsg[]){  
        InfoImpl i = new InfoImpl("harvic");  
        System.out.println(i.getVar()) ;  
    }  
};  
```

泛型类 

```

```

### 运算符

#### ^

按位异或

#### >>>

无符号右移运算符 ,避免负数的溢出 ,代替  /2 操作, 避免产生负数

'>>>1 =  /2  '>>>2 = /4  ,但避免因数值过大造成的负数溢出

#### 位运算

| 符号  | 描述  | 运算规则                                                            |
|:--- |:--- |:---------------------------------------------------------------:|
| &   | 与   | 两个位都为1时，结果才为1                                                   |
| \|  | 或   | 两个位都为0时，结果才为0                                                   |
| ^   | 异或  | 两个位相同为0，相异为1                                                    |
| ~   | 取反  | 0变1，1变0                                                         |
| <<  | 左移  | 各二进位全部左移若干位，高位丢弃，低位补0                                           |
| >>  | 右移  | 各二进位全部右移若干位，对无符号数，高位补0，有符号数，各编译器处理方法不一样，有的补符号位（算术右移），有的补0（逻辑右移） |

### cast 函数

```
public T[] cast(Object obj)
```

用于将指定的对象强制转换为此类的对象. 该方法以对象形式转换后返回对象

**参数：**此方法接受参数obj，它是要转换的对象

**返回值：**此方法以对象形式转换后返回指定的对象。

**异常：**该方法抛出：

- **ClassCastException：**如果对象不为null，并且不能分配给T类型。

### 关于接口

- 接口为Java中一种提供一种统一的'协议'(相当于为开发者提供一个模板定义), 接口中的属性也属于'协议'中的成员, 他们是公共的,静态的,最终的常量,相当于全局变量,为了保证接口提供的统一这种抽象思想,接口中的属性必然是常量,只能读不能改,这样才能为实现接口的对象提供一个统一的属性
- 抽象类是不"完全"的类,相当于是接口和具体类的一个中间层,即满足接口的抽象, 也满足具体的实现 
- 接口可以继承另一个接口
- 抽象类可以继承接口, 实现多继承

### 正则

一些基础正则表达式

| 正则表达式    | 含义                        |
|:--------:|:-------------------------:|
| `.`      | 匹配任何单个字符。                 |
| `?`      | 一次匹配或根本不匹配前面的元素。          |
| `+`      | 与前面的元素匹配一次或多次。            |
| `*`      | 与前面的元素匹配零次或多次。            |
| `^`      | 匹配字符串中的起始位置。              |
| `$`      | 匹配字符串中的结束位置。              |
| `        | `                         |
| `[abc]`  | 匹配 a 或 b 或 c。             |
| `[a-c]`  | 范围; 匹配 a 或 b 或 c。         |
| `[^abc]` | 否定，匹配除 a 或 b 或 c 之外的所有内容。 |
| `\s`     | 匹配空白字符。                   |
| `\w`     | 匹配单词字符； 等同于`[a-zA-Z_0-9]` |

## 内部类详解

将一个类的定义放在另一个类的定义内部，这就是内部类

 在Java中内部类主要分为成员内部类、局部内部类、匿名内部类、静态内部类。

### 为什么要使用内部类

《Think in java》中有这样一句话：使用内部类最吸引人的原因是

每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

在我们程序设计中有时候会存在一些使用接口很难

的问题，这个时候我们可以利用内部类提供的、可以继承多个具体的或者抽象的类的能力来解决这些程序设计问题。可以这样说，接口只是解决了部分问题，而内部类使得多重继承的解决方案变得更加完整。

```java
public interface Father {

}


public interface Mother {

}

public class Son implements Father, Mother {

}

public class Daughter implements Father{

    class Mother_ implements Mother{

    }
}
```

 其实对于这个实例我们确实是看不出来使用内部类存在何种优点，但是如果Father、Mother不是接口，而是抽象类或者具体类呢？这个时候我们就只能使用内部类才能实现多重继承了。

   其实使用内部类最大的优点就在于它能够非常好的解决多重继承的问题，但是如果我们不需要解决多重继承问题，那么我们自然可以使用其他的编码方式，但是使用内部类还能够为我们带来如下特性（摘自《Think in java》）：

   **1、**内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。

   **2、**在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。

   **3、**创建内部类对象的时刻并不依赖于外围类对象的创建。

   **4、**内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。

   **5、**内部类提供了更好的封装，除了该外围类，其他类都不能访问。

### 成员内部类

 在成员内部类中要注意两点，**第一：**成员内部类中不能存在任何static的变量和方法；**第二：**成员内部类是依附于外围类的，所以只有先创建了外围类才能够创建内部类。

```java
public class OuterClass {
    private String str;

    public void outerDisplay(){
        System.out.println("outerClass...");
    }

    public class InnerClass{
        public void innerDisplay(){
            //使用外围内的属性
            str = "chenssy...";
            System.out.println(str);
            //使用外围内的方法
            outerDisplay();
        }
    }

    /*推荐使用getxxx()来获取成员内部类，尤其是该内部类的构造函数无参数时 */
    public InnerClass getInnerClass(){
        return new InnerClass();
    }

    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        OuterClass.InnerClass inner = outer.getInnerClass();
        inner.innerDisplay();
    }
}
```

#### 

#### **常见面试问题**

准备在下一次面试中面对与CountDownLatch相关的这些问题。

- 解释CountDownLatch的概念吗？

- CountDownLatch和CyclicBarrier之间的区别？

- 举例说明CountDownLatch的用法吗？

- CountDownLatch类有哪些主要方法？

### Synchronized

 3种使用形式 

- Synchronized修饰普通同步方法：锁对象当前实例对象；
- Synchronized修饰静态同步方法：锁对象是当前的类Class对象；
- Synchronized修饰同步代码块：锁对象是Synchronized后面括号里配置的对象，这个对象可以是某个对象（xlock），也可以是某个类（Xlock.class）；

注意 

- 使用synchronized修饰非静态方法或者使用synchronized修饰代码块时制定的为实例对象时，同一个类的不同对象拥有自己的锁，因此不会相互阻塞。
- 使用synchronized修饰类和对象时，由于类对象和实例对象分别拥有自己的监视器锁，因此不会相互阻塞。
    使用使用synchronized修饰实例对象时，如果一个线程正在访问实例对象的一个synchronized方法时，其它线程不仅不能访问该synchronized方法，该对象的其它synchronized方法也不能访问，因为一个对象只有一个监视器锁对象，但是其它线程可以访问该对象的非synchronized方法。
- 线程A访问实例对象的非static synchronized方法时，线程B也可以同时访问实例对象的static synchronized方法，因为前者获取的是实例对象的监视器锁，而后者获取的是类对象的监视器锁，两者不存在互斥关系。

#### 实现原理

[参考](https://blog.csdn.net/tongdanping/article/details/79647337)

1. 对象头 

首先, 对象在内存中的布局 

已知对象是存放在堆内存中的，对象大致可以分为三个部分，分别是对象头、实例变量和填充字节。

- 对象头的zhuyao是由MarkWord和Klass Point(类型指针)组成，其中Klass Point是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据。如果对象是数组对象，那么对象头占用3个字宽（Word），如果对象是非数组对象，那么对象头占用2个字宽。（1word = 2 Byte = 16 bit）
- 实例变量存储的是对象的属性信息，包括父类的属性信息，按照4字节对齐
- 填充字符，因为虚拟机要求对象字节必须是8字节的整数倍，填充字符就是用于凑齐这个整数倍的

![image-20210523230519277](../images/java_learing_temp-writing/image-20210523230519277.png)

通过第一部分可以知道，Synchronized不论是修饰方法还是代码块，都是通过持有修饰对象的锁来实现同步，那么Synchronized锁对象是存在哪里的呢？答案是存在锁对象的对象头的MarkWord中。那么MarkWord在对象头中到底长什么样，也就是它到底存储了什么呢？

在32位的虚拟机中：

![image-20210523230600653](../images/java_learing_temp-writing/image-20210523230600653.png)

在64 位的虚拟机中:

![image-20210523230633942](../images/java_learing_temp-writing/image-20210523230633942.png)

上图中的偏向锁和轻量级锁都是在java6以后对锁机制进行优化时引进的，下文的锁升级部分会具体讲解，Synchronized关键字对应的是重量级锁，接下来对重量级锁在Hotspot JVM中的实现锁讲解。

2. Synchronized 在 JVM 中的实现原理 

重量级锁的标志位是 10 ,存储了指向重量级监视器锁的指针, 在Hotspot 中 ,对象的监视器(monitor) 锁对象由 ObjectMonitor 对象 实现(C++) , 其跟同步相关的数据结构如下: 

```c++
ObjectMonitor() {
    _count        = 0; //用来记录该对象被线程获取锁的次数
    _waiters      = 0;
    _recursions   = 0; //锁的重入次数
    _owner        = NULL; //指向持有ObjectMonitor对象的线程 
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
  }
```

看一下线程在获取锁的几个状态的转换: 

![img](../images/java_learing_temp-writing/20190403174421871.jpg)

线程的生命周期存在5个状态，start、running、waiting、blocking和dead

对于一个synchronized修饰的方法(代码块)来说：

1. 当多个线程同时访问该方法，那么这些线程会先被放进_EntryList队列，此时线程处于blocking状态_
2. 当一个线程获取到了实例对象的监视器（monitor）锁，那么就可以进入running状态，执行方法，此时，ObjectMonitor对象的_owner指向当前线程，_count加1表示当前对象锁被一个线程获取
3. 当running状态的线程调用wait()方法，那么当前线程释放monitor对象，进入waiting状态，ObjectMonitor对象的_owner变为null，_count减1，同时线程进入_WaitSet队列，直到有线程调用notify()方法唤醒该线程，则该线程重新获取monitor对象进入_Owner区
4. 如果当前线程执行完毕，那么也释放monitor对象，进入waiting状态，ObjectMonitor对象的_owner变为null，_count减1

##### Synchronized 修饰的代码块/方法 获取monitor 对象

在JVM 规范里可以看到,不管是方法同步还是代码块同步都是基于进入和退出monitor对象来实现，然而二者在具体实现上又存在很大的区别。通过javap对class字节码文件反编译可以得到反编译后的代码。

### 并发集合包

JUC 

ConcurrentHashMap 保证了线程安全 , 如果想要安全可以用ConcurrentHashMap 

### AQS

AbstractQueuedSynchronizer  在 java.util.concurrent.locks包下 

AQS 是一个用来构建锁和同步器的框架 使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask`(jdk1.7) 等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器。

#### 原理

**AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。

​                                  ![enter image description here](../images/java_learing_temp-writing/CLH.png)

## 五种 I/O模型

unix 下5种可用的 I/O模型 

- 阻塞时I/O
- 非阻塞式I/O
- I/O复用  (select 和 poll)
- 信号驱动式I/O(SIGIO)
- 异步I/O(ＰＯＳＩＸ的ａｉｏ＿系列函数)

一个输入操作通常包含两个不同的阶段：

（１）等待数据准备好

（２）从内核向进程复制数据

### 阻塞式Ｉ／Ｏ模型

最流行的I/O 模型是阻塞式I/O (blocking I/O) 模型 .默认情况下,  所有套接字都是阻塞的. 

以数据报套接字作为例子 

 ![image-20210530233639134](../images/java_learing_temp-writing/image-20210530233639134.png)

使用UDP而不是TCP 作为例子的原因在于就UDP 而言, 数据准备好读取 的概念比较简单 ;

- 整个数据报已经收到, 要么还么有 
  
    对于TCP来说 , 注入套接字低水平位标记 (low-water mark )等额外变量开始起作用, 导致这个概念变得复杂

将recvfrom 函数视为系统调用, 因为我们正在区分应用进程和内核 ,不论它如何实现(在源自Berkeley的内核上视为系统调用,在 System V内核上是作为调用系统调用get'msg函数),一般都会从在应用进程空间中运行切换到内核空间中运行,一段时间后再切换回来 

图6-1, 进程调用 recvfrom, 其系统调用直到数据报到达且被复制到应用进程的缓冲区中或者发生错误才返回. 最常见的错误是系统调用被信号中断 

进程在从调用recvfrom 开始知道它返回地整段时间内是被阻塞的 . recvfrom 返回后, 应用进程开始处理数据 

### 非阻塞式I/O模型(Non-blocking/New I/O)

进程把一个套接字设置成非阻塞是在通知内核: 当所请求的I/O操作非得把本线程投入休眠才能完成时, 不要把本进程投入睡眠, 而是返回一个错误 

![image-20210531103208956](../images/java_learing_temp-writing/image-20210531103208956.png)

前三次调用recvfrom 时没有数据可返回 , 因此内核转而立即返回一个EWOULDBLOCK 错误 ,

第四次调用recvfrom 时已有一个数据报数据准备好 ,它被复制到应用进程缓冲区 于是recvfrom 成功返回 , 继续处理数据 

当一个应用进程像这样对一个非阻塞描述符循环调用recvfrom 时, 我们称之为轮询(polling) , 应用进程持续轮询内核, 以查看某个操作是否就绪, 这么做往往耗费大量CPU时间 , 这种模型偶尔 遇到, 通常是在专门提供某一种功能的系统中才有 

### I/O 复用模型

![image-20210531103813917](../images/java_learing_temp-writing/image-20210531103813917.png)

我们阻塞select调用, 等待数据报套接字变为刻度. 当select 返回套接字可读这一条件时, 我们调用recvfrom 把所读数据报复制到应用进程缓冲区

![image-20210531104823924](../images/java_learing_temp-writing/image-20210531104823924.png)

应用首先发送select 调用, 询问内核数据是否准备就绪, 当内核把数据准备好了之后,用户线程再发起read调用 , read的调用过程(数据从内核空间-> 用户空间)还是阻塞的 

```
目前支持 IO 多路复用的系统调用，有 select，epoll 等等。select 系统调用，是目前几乎在所有的操作系统上都有支持

select 调用 ：内核提供的系统调用，它支持一次查询多个系统调用的可用状态。几乎所有的操作系统都支持。
epoll 调用 ：linux 2.6 内核，属于 select 调用的增强版本，优化了 IO 的执行效率。
```

**IO多路复用模型,通过减少无效的系统调用,减少了对CPU资源的消耗**

Java 中的NIO ,有一个非常重要的选择器(Selector )的概念, 也可以被称为 **多路复用器**. 通过它, 只需要一个线程便可以管理多个客户端连接, 客户端数据到了之后, 才会为其服务 

![image-20210611082324195](../images/java_learing_temp-writing/image-20210611082324195.png)

### 信号驱动式I/O 模型

我们也可以用信号 , 让内核在描述符就绪时发送SIGIO 信号通知我们. 我们称这种模型为信号驱动式I/O(signal-driven I/O)

![image-20210531105057367](../images/java_learing_temp-writing/image-20210531105057367.png)

### 异步 I/O模型(Asynchronous I/O)

AIO

AIO 也就是 NIO 2。Java 7 中引入了 NIO 的改进版 NIO 2,它是异步 IO 模型。(Java)

异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

异步I/O (asychronous I/O) 由POSIX 规范定义. 演变成当前POSIX 规范的各种早期标准所定义的实时函数中存在的差异已经取得一致 

一般的说, 这些函数的工作机制是: 告知内核启动某个操作, 并让内核在整个操作(包括将数据从内核复制到我们自己的缓冲区) 完成后同告知我们 . 这种模型和 信号驱动模型的主要区别在于: 信号驱动式I/O 是由内核通知我们合适可以启动一个I/O 操作, 而异步I/O 模型是由内核通知我们I/O 操作何时完成 

![image-20210531110238304](../images/java_learing_temp-writing/image-20210531110238304.png)

![image-20210531110246381](../images/java_learing_temp-writing/image-20210531110246381.png)

目前来说 AIO 的应用还不是很广泛。Netty 之前也尝试使用过 AIO，不过又放弃了。这是因为，Netty 使用了 AIO 之后，在 Linux 系统上的性能并没有多少提升。

最后，来一张图，简单总结一下 Java 中的 BIO、NIO、AIO。

![image-20210611114815963](../images/java_learing_temp-writing/image-20210611114815963.png)

## 多重继承的实现

### 接口

```java
interface CanFight {
    void fight();
}

interface CanSwim {
    void swim();
}


interface CanFly {
    void fly();
}

public class ActionCharacter {
    public void fight(){

    }
}

public class Hero extends ActionCharacter implements CanFight,CanFly,CanSwim{

    public void fly() {
    }

    public void swim() {
    }

    /**
     * 对于fight()方法，继承父类的，所以不需要显示声明
     */
}
```

```java

```

### 内部类

如果父亲位抽象类或者具体类，就仅能通过内部类来实现多重继承

```
儿子是如何利用多重继承来继承父亲和母亲的优良基因
```

```java
public class Father {
    public int strong(){
        return 9;
    }
}

public class Mother {
    public int kind(){
        return 8;
    }
}
```

```java
public class Son {

    /**
     * 内部类继承Father类
     */
    class Father_1 extends Father{
        public int strong(){
            return super.strong() + 1;
        }
    }

    class Mother_1 extends  Mother{
        public int kind(){
            return super.kind() - 2;
        }
    }

    public int getStrong(){
        return new Father_1().strong();
    }

    public int getKind(){
        return new Mother_1().kind();
    }
}
```

测试程序

```java
public class Test1 {

    public static void main(String[] args) {
        Son son = new Son();
        System.out.println("Son 的Strong：" + son.getStrong());
        System.out.println("Son 的kind：" + son.getKind());
    }

}
----------------------------------------
Output:
Son 的Strong：10
Son 的kind：6
```

儿子继承了父亲，变得比父亲更加强壮，同时也继承了母亲，只不过温柔指数下降了。这里定义了两个内部类，他们分别继承父亲Father类、母亲类Mother类，且都可以非常自然地获取各自父类的行为，这是内部类一个重要的特性：内部类可以继承一个与外部类无关的类，保证了内部类的独立性，正是基于这一点，多重继承才会成为可能。

## 时间类

### 枚举类TimeUnit 线程休眠新方式

 Thread.sleep(7000);  

Da

#### TimeUnit.SECONDS与TimeUnit.MILLISECONDS区别

TimeUnit.SECONDS（5）线程等待五秒

TimeUnit.MILLISECONDS(5000)线程等待五秒.

### datetime

sql datetime 和普通 datetime 

为sql中datetime字段赋值时可以利用 util的datetime , 生成 , year, month, day

## 工具类

utools 

Guava goole 

JUC 集合包中的集合是线程安全的 并发集合

编写工具类时一般使用静态方法或者单例模式

## 序列化 和反序列化

 [参考](https://juejin.cn/post/6844903848167866375)

**序列化 : 将对象写入到IO流中** 

**反序列化: 从Io流中恢复对象** 

**意义：序列化机制允许将实现序列化的Java对象转换位字节序列，这些字节序列可以保存在磁盘上，或通过网络传输，以达到以后恢复成原来的对象。序列化机制使得对象可以脱离程序的运行而独立存在。**

**使用场景: 所有可以在网络上传输的对象都必须是可序列化的,** 比如RMI（remote method invoke,即远程方法调用），传入的参数或返回的对象都是可序列化的，否则会出错；**所有需要保存到磁盘的java对象都必须是可序列化的。通常建议：程序创建的每个JavaBean类都实现Serializeable接口。**

① 想把内存中的对象保存到一个文件中或者数据库中时候；
       ② 想用套接字在网络上传送对象的时候；(序列化远程实现通信)
       ③ 想通过RMI传输对象的时候

一些应用场景，涉及到将对象转化成二进制，序列化保证了能够成功读取到保存的对象。

可以实现数据的持久化, 通过序列化可以把数据永久地保存到硬盘上(通常存放在文件里)

实现进程间的对象传送    

### 几种常见的序列化和反序列化协议

- XML&SOAP

XML 是一种常用的序列化和反序列化协议，具有跨机器，跨语言等优点，SOAP（Simple Object Access protocol） 是一种被广泛应用的，基于 XML 为序列化和反序列化协议的结构化消息传递协议

- JSON（Javascript Object Notation）
- Protobuf

## 注解

### @AllArgsConstructor

生成一个全参数构造器，也就是为类中的属性添加构造器，可用于枚举类中的属性添加构造器，但需要构造器中有参数，也就是需要有类型为final 的字段

### @SPI

SPI全程Service Provider Interface，是一种服务发现机制。当程序运行调用接口时，会根据配置文件或默认规则信息加载对应的实现类。所以在程序中并没有直接指定使用接口的哪个实现，而是在外部进行装配。 要想了解 Dubbo 的设计与实现，其中 Dubbo SPI 加载机制是必须了解的，在 Dubbo 中有大量功能的实现都是基于 Dubbo SPI 实现解耦，同时也使得 Dubbo 获得如此好的可扩展性。

使用场景

  SPI （`Service Provider Interface）`是`调用方`来制定接口规范，提供给外部来实现，`调用方在调用时则`选择自己需要的外部实现。 从使用人员上来说，SPI 被框架扩展人员使用。

## java集合框架(Collection)

### 集合概述

![img](../images/java_learing_temp-writing/java-collection-hierarchy.png)

#### List, Set, Map 三者区别

- `List`(对付顺序的好帮手)： 存储的元素是有序的、可重复的。
- `Set`(注重独一无二的性质): 存储的元素是无序的、不可重复的。
- `Map`(用 Key 来搜索的专家): 使用键值对（key-value）存储，类似于数学上的函数 y=f(x)，“x”代表 key，"y"代表 value，Key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。

### 集合框架底层数据结构总结1

Collection 接口下集合 

#### List

存储元素有序 , 可重复的

Arraylist: Object[] 数组 

Vector : Object[] 数组 

LinkedList : 双向链表(JDK1.6之前为循环链表, JDK1.7 取消了循环)

#### Set

(注重独一无二), 存储元素无序, 不可重复的  可以存储key 

#### Map

用key搜索, 使用键值对存储, 类似于数学上的函数y=f(x),  “x”代表 key，"y"代表 value，Key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。

### Map 类

#### putIfAbsent

key存在,value为空,填充value,并返回value

源码

```java
  default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }
```

## hashMap 底层数据结构分析

[参考](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90)

HashMap 主要用来存放键值对, 它基于哈希表的Map结构实现, 是常用的Java集合之一 

JDK1.8 之前 HashMap 由 数组+链表 组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。

JDK1.8 之后 HashMap 的组成多了红黑树，在满足下面两个条件之后，会执行链表转红黑树操作，以此来加快搜索速度。

- 链表长度大于阈值（默认为 8）
- HashMap 数组长度超过 64

### hash方法源码

 JDK1.8

```
  static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

JDK 1.7 

```
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

'拉链法': 将链表和数组相结合. 即创建一个链表数组, 数组中每一格就是一个链表, 若遇到哈希冲突, 将哈希冲突的值加进链表即可

![jdk1.8之前的内部结构](../images/java_learing_temp-writing/jdk1.8%E4%B9%8B%E5%89%8D%E7%9A%84%E5%86%85%E9%83%A8%E7%BB%93%E6%9E%84.png)

JDK1.8 之后 

JDK1.8 以后在解决哈希冲突时有了较大的变化 

当链表长度大于阈值（默认为 8）时，会首先调用 `treeifyBin()`方法。这个方法会根据 HashMap 数组来决定是否转换为红黑树。只有当数组长度大于或者等于 64 的情况下，才会执行转换红黑树操作，以减少搜索时间。否则，就是只是执行 `resize()` 方法对数组扩容。重点关注 `treeifyBin()`方法即可！

![img](../images/java_learing_temp-writing/up-bba283228693dae74e78da1ef7a9a04c684.png)

类的属性:

```
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V> ,cloneable, Serizable{

  // 序列号
    private static final long serialVersionUID = 362498820763181265L;
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table;
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 加载因子
    final float loadFactor;

} 
```

- **loadFactor 加载因子**
  
    loadFactor 加载因子是控制数组存放数据的疏密程度，loadFactor 越趋近于 1，那么 数组中存放的数据(entry)也就越多，也就越密，也就是会让链表的长度增加，loadFactor 越小，也就是趋近于 0，数组中存放的数据(entry)也就越少，也就越稀疏。
  
    **loadFactor 太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。loadFactor 的默认值为 0.75f 是官方给出的一个比较好的临界值**。
  
    给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。

- **threshold**
  
    **threshold = capacity \* loadFactor**，**当 Size>=threshold**的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 **衡量数组是否需要扩增的一个标准**。

Node节点类源码![image-20210520140645127](../images/java_learing_temp-writing/image-20210520140645127.png)

```
// 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
       final K key;//键
       V value;//值
       // 指向下一个节点
       Node<K,V> next;
       Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // 重写hashCode()方法
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        // 重写 equals() 方法
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
}
```

树节点类源码:![image-20210520140653234](../images/java_learing_temp-writing/image-20210520140653234.png)

```
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // 父
        TreeNode<K,V> left;    // 左
        TreeNode<K,V> right;   // 右
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;           // 判断颜色
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
        // 返回根节点
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
       }
```

### 源码分析

hashMap中有四个构造方法 , 如下 

```
 // 默认构造函数。
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
     }

     // 包含另一个“Map”的构造函数
     public HashMap(Map<? extends K, ? extends V> m) {
         this.loadFactor = DEFAULT_LOAD_FACTOR;
         putMapEntries(m, false);//下面会分析到这个方法
     }

     // 指定“容量大小”的构造函数
     public HashMap(int initialCapacity) {
         this(initialCapacity, DEFAULT_LOAD_FACTOR);
     }

     // 指定“容量大小”和“加载因子”的构造函数
     public HashMap(int initialCapacity, float loadFactor) {
         if (initialCapacity < 0)
             throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
         if (initialCapacity > MAXIMUM_CAPACITY)
             initialCapacity = MAXIMUM_CAPACITY;
         if (loadFactor <= 0 || Float.isNaN(loadFactor))
             throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
         this.loadFactor = loadFactor;
         this.threshold = tableSizeFor(initialCapacity);
     }
```

putMapEntries 方法:

```
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { // pre-size
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

#### put方法

HashMap 只提供了用 put用于添加元素, putVal 方法只是给put方法调用的一个方法, 并没有共给用户使用 

**对 putVal 方法添加元素的分析如下：**

1. 如果定位到的数组位置没有元素 就直接插入。
2. 如果定位到的数组位置有元素就和要插入的 key 比较，如果 key 相同就直接覆盖，如果 key 不相同，就判断 p 是否是一个树节点，如果是就调用`e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)`将元素添加进入. 如果不是就遍历链表插入(链表尾部)

![ ](../images/java_learing_temp-writing/put%E6%96%B9%E6%B3%95.png)

说明:上图有两个小问题：

- 直接覆盖之后应该就会 return，不会有后续操作。参考 JDK8 HashMap.java 658 行（[issue#608](https://github.com/Snailclimb/JavaGuide/issues/608)）。
- 当链表长度大于阈值（默认为 8）并且 HashMap 数组长度超过 64 的时候才会执行链表转红黑树的操作，否则就只是对数组扩容。参考 HashMap 的 `treeifyBin()` 方法（[issue#1087](https://github.com/Snailclimb/JavaGuide/issues/1087)）。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值(默认为 8 )，执行 treeifyBin 方法
                    // 这个方法会根据 HashMap 数组来决定是否转换为红黑树。
                    // 只有当数组长度大于或者等于 64 的情况下，才会执行转换红黑树操作，以减少搜索时间。否则，就是只是对数组扩容。
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) {
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

#### get 方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### resize方法

进行扩容, 会伴随着一次重新hash分配, 并且会遍历 hash 表中所有的元素, 是非常耗时的, 编写程序中, 尽量避免resize

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
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
```

## ConcurrentHashMap

线程安全的hashMap 

### 1.7

![Java 7 ConcurrentHashMap 存储结构](../images/java_learing_temp-writing/image-20200405151029416.png)

Java 7 中 ConcurrentHashMap 的存储结构如上图，ConcurrnetHashMap 由很多个 Segment 组合，而每一个 Segment 是一个类似于 HashMap 的结构，所以每一个 HashMap 的内部可以进行扩容。但是 Segment 的个数一旦**初始化就不能改变**，默认 Segment 的个数是 16 个，你也可以认为 ConcurrentHashMap 默认支持最多 16 个线程并发。

不做太多了解 ,直接学习1.8

### 1.8

#### 存储结构

![Java8 ConcurrentHashMap 存储结构（图片来自 javadoop）](../images/java_learing_temp-writing/java8_concurrenthashmap.png)

Java8 的ConcurrentHashMap 相对于 java7 来说变化较大 , 不再是之前的Segment 数组 + HashEntry 数组 + 链表 , 而是 Node 数组 + 链表/ 红黑树, 当冲突链表达到一定长度时, 链表会转换成红黑树 

#### 初始化 initTale

```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        ／／　如果 sizeCtl < 0 ,说明另外的线程执行CAS 成功，正在进行初始化。
        if ((sc = sizeCtl) < 0)
            // 让出 CPU 使用权
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

从源码中可以发现 ConcurrentHashMap 的初始化是通过自旋和CAS操作完成的 , 里面需要注意的是变量sczeCtl  ,它的值决定着当前的初始状态 

1. -1 说明正在初始化1
2. -N 说明有N-1 个线程正在进行扩容 
3. 表示table 初始化大小, 如果 table没有初始化 
4. 表示table 容量, 如果 table 已经 初始化

Java8 中的 ConcurrentHashMap 使用的 Synchronized 锁加 CAS 的机制。结构也由 Java7 中的 **Segment 数组 + HashEntry 数组 + 链表** 进化成了 **Node 数组 + 链表 / 红黑树**，Node 是类似于一个 HashEntry 的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表。

## 设计模式

### 工厂模式

![image-20210621160856499](../images/java_learing_temp-writing/image-20210621160856499.png)

工厂有时里面是不接收参数的，直接从工厂里面返回一个结果，当然如果工厂里面要接收参数且返回结果那Function函数式接口就可以派上用场了

#### 使用supplire接口

使用supplire 生成Student 对象

 例如

![image-20210621161114345](../images/java_learing_temp-writing/image-20210621161114345.png)

![image-20210621161147168](../images/java_learing_temp-writing/image-20210621161147168.png)

### 单例模式

实现对象的单一化, 节省内存,提高性能,  

不适用于变化频繁的对象, 避免将数据连接池对象涉及为单例类, 可能会导致共享连接池对象的惩处过多而出现连接池溢出

 工具类

#### 懒汉单例

```java
public class Singleton{

    private static Singleton instance = null;


     /**
     * 2、适用于多线程环境，但效率不高（不推荐）
     * 线程调用getInstance 都会被synchronized 锁住, 造成线程阻塞
     */
    public static synchronized Singleton getInstance1() {
        if (instance == null) {
            instance = new Singleton1();
        }
        return instance;
    }

    /**
    *双重检查加锁,不影响程序性能, 实例未被创建时再加锁
    */
    public static Singleton getInstance2(){
        if(instance == null){
             synchronized  (Singleton.class){
                instance = new Sincleton();
            }
        }
        return instance;
    }
}
```

但会节约资源 

#### 饿汉单例

##### 饿汉式

类初始化时已经自行实例化 , 采取直接 实例化 instance 的方式不会产生线程不安全 

```java
public class Singleton2(){
    private static final Singleton2 instance = new Singleton2();
    private Singleton2(){}
    public  static Singleton2 getInstance(){
        return instance;
    } 
}
```

#### 双重校验锁- 线程安全

instance 只需要被实例化一次 ,之后就可以直接使用了. 加锁操作只需要对实例化那部分的代码进行,  只有当instance没有被实例化时, 才需要加锁 

双重校验锁先判断instance 是否已经被实例化, 没有被实例化, 那么才对实例化语句进行加锁

```
public class  Singleton{
    private volatile static Singleton instance ;

    private Singleton(){
    }

    public statoc static Singleton getInstance (){
        if(instance == null){
            synchromized (Singleton.class){
                if(instance==null){
                    instance = new Singleton();        
                }
            }
            return instance;
        }

}
```

考虑下面的代码, 当只用了一个if语句时, 在 instance  == null 的情况下,如果两个线程都执行了 if语句, 那么两个线程都会进入if语句块内 , 虽然在 if 语句块内有加锁操作 ,但是两个线程都会执行 instance = new Singleton();   , 只是先后问题,那么就会进行两次实例化. 因此必须使用双重校验锁, 也就是需要使用两个if语句 : 第一个语句用来避免 instance 已经实例化之后的加锁操作, 而第二个if语句进行了加锁, 所以只能有一个线程进入 , 就不会出现 instance == null 时两个线程同时进行实例化操作 

```
 if (uniqueInstance == null) {
    synchronized (Singleton.class) {
        uniqueInstance = new Singleton();
    }
}
```

instance 采用 volatile 关键字修饰也是很有必要的 , instance = new Singleton(); 这段代码其实是分三步: 

1. 为instance 分配内存空间 
2. 初始化 instance 
3. 将 instance 指向分配的内存地址 

但是由于 JVM 具有指令重排的, 执行顺序有可能变成1>3>2 . 指令重排在单线程环境下不会出问题, 但是在多线程环境下回导致一个线程获得还没有初始化的实例 . 例如, 线程 T1 执行了 1 和 3 , 此时T2 调用 getInstance() 后发现 instance 不为空, 因此返回 instance , 但此时instance 还未被 初始化 . 

使用volatile 可以禁止 JVM 的指令重排 ,保证在多线程环境下也能正常运行 

#### 静态内部类

加载一个类时，其内部类不会同时被加载。一个类被加载，当且仅当其某个静态成员（静态域、构造器、静态方法等）被调用时发生。 由于在调用 StaticSingleton.getInstance() 的时候，才会对单例进行初始化，而且通过反射，是不能从外部类获取内部类的属性的；由于静态内部类的特性，只有在其被第一次引用的时候才会被加载，所以可以保证其线程安全性。
总结：
优势：兼顾了懒汉模式的内存优化（使用时才初始化）以及饿汉模式的安全性（不会被反射入侵）。
劣势：需要两个类去做到这一点，虽然不会创建静态内部类的对象，但是其 Class 对象还是会被创建，而且是属于永久带的对象

这种方式不仅具有延迟初始化的好处，而且由 JVM 提供了对线程安全的支持。。

```java
/**
*静态内部类方式 
*/
public class StaticSingleton(){
    /*
    * 私有构造方法,
    */
    private StaticSingleton(){   
    }

    public static StaticSingleton getInstance(){
       return StaticSingletonHolder.instance;
    }
    private static class StaticSingletonHolder(){
        private static final StaticSingleton instance = new StaticSingleton(); 
    }
    public void method(){}

   public static void main(){
       StaticSingleton.getInstance().method();
   }
}
```

```
public class Singleton{
    private Singleton(){}
    private static class SingletonHolder{
        private static final Singleton INSTANCE = new Singleton();
    }
    pulbic statoc Somgleton getInstance(){
        return SingletonHolder.INSTANCE 
    }
}
```

#### 枚举

> 创建枚举默认就是线程安全的，所以不需要担心double checked locking，而且还能防止反序列化导致重新创建新的对象。保证只有一个实例（即使使用反射机制也无法多次实例化一个枚举量）。

```java
public class Singleton(){
    public static void main(String[] args){
        Single single = Single.SINGLE;
        single.print();
    }

    enum Single(){
        SINGLE;
        private Single(){
        }
        public void print(){
            System.out.println("hello World");
        }
    }
}
```

一般情况下直接使用饿汉式就好了，如果明确要求要懒加载（lazy initialization）会倾向于使用静态内部类，如果涉及到反序列化创建对象时会试着使用枚举方式来实现单例。

### 策略模式

### 模板方法

## jvm

[参考](https://www.huaweicloud.com/articles/877c28be67566a35de9146038afeb59e.html)

### jvm 栈

它是Java方法执行的内存模型。里面会对局部变量，动态链表，方法出口，栈的操作（入栈和出栈）进行存储，且线程独享。同时如果我们听到局部变量表，那也是在说虚拟机栈

如果线程请求的栈的深度大于虚拟机栈的最大深度，就会报StackOverflowError（这种错误常出现在递归中）Java虚拟机也可以动态扩展，但随着扩展会不断地申请内存，当无法申请足够内存时就会报错 **OutOfMemoryError**。

#### 栈生命周期

对于栈来说，只要程序运行结束，栈空间自然会释放，栈的生命周期和所处的线程是一致的。

8种基本类型的变量+对象的引用变量+实例方法都是在栈里面分配内存。

#### jvm栈的执行

JVM中中栈帧，在Java中就是方法，存放在栈中

栈中的数据都是以栈帧的格式存在，它是一个关于方法和运行期数据的数据集。比如我们执行一个方法a，就会对应产生一个栈帧A1，然后A1会被压入栈中。同理方法b会有一个B1，方法c会有一个C1，等到这个线程执行完毕后，栈会先弹出C1，后B1,A1。它是一个先进后出，后进先出原则。

#### 局部变量的复用

局部变量表用于存放方法参数和方法内部所定义的局部变量。它的容量是以Slot为最小单位，一个slot可以存放32位以内的数据类型。

虚拟机通过索引定位的方式使用局部变量表，范围为[0,局部变量表的slot的数量]。方法中的参数就会按一定顺序排列在这个局部变量表中，至于怎么排的我们可以先不关心。而为了节省栈帧空间，这些slot是可以复用的，当方法执行位置超过了某个变量，那么这个变量的slot可以被其它变量复用。当然如果需要复用，那我们的垃圾回收自然就不会去动这些内存。

### 堆

堆内存中划分年轻代和老年代，

年轻代分为Eden和Survivor区，Survivor区又分为FromPlace和ToPlace ,toPlace的survivor区域是空的。Eden，FromPlace和ToPlace的默认占比为 **8:1:1**。当然这个东西其实也可以通过一个 -XX:+UsePSAdaptiveSurvivorSizePolicy 参数来根据生成对象的速率动态调整

#### Eden年轻代

当我们new一个对象后，会先放到Eden划分出来的一块作为存储空间的内存，但是我们知道对堆内存是线程共享的，所以有可能会出现两个对象共用一个内存的情况。这里JVM的处理是每个线程都会预先申请好一块连续的内存空间并规定了对象存放的位置，而如果空间不足会再申请多块内存空间。这个操作我们会称作TLAB

当Eden空间满了之后 ，会触发一个叫做Minor Gc（就是一个发生在年轻代的Gc）操作，存活下来的对象移动到Survivor0区。Survivor0区满后触发 Minor GC，就会将存活对象移动到Survivor1区，此时还会把from和to两个指针交换，这样保证了一段时间内总有一个survivor区为空且to所指向的survivor区为空。经过多次的 Minor GC后仍然存活的对象（**这里的存活判断是15次，对应到虚拟机参数为 -XX:MaxTenuringThreshold 。为什么是15，因为HotSpot会在对象投中的标记字段里记录年龄，分配到的空间仅有4位，所以最多只能记录到15**）会移动到老年代。老年代是存储长期存活的对象的，占满时就会触发我们最常听说的Full GC，期间会停止所有线程等待GC的完成。所以对于响应要求高的应用应该尽量去减少发生Full GC从而避免响应超时的问题。

而且当老年区执行了full gc之后仍然无法进行对象保存的操作，就会产生OOM，这时候就是虚拟机中的堆内存不足，原因可能会是堆内存设置的大小过小，这个可以通过参数-Xms、-Xmx来调整。也可能是代码中创建的对象大且多，而且它们一直在被引用从而长时间垃圾收集无法收集它们。

![image-20210401205811824](../images/java_learing_temp-writing/image-20210401205811824.png)

补充说明：关于-XX:TargetSurvivorRatio参数的问题。其实也不一定是要满足-XX:MaxTenuringThreshold才移动到老年代。可以举个例子：如对象年龄5的占30%，年龄6的占36%，年龄7的占34%，加入某个年龄段（如例子中的年龄6）后，总占用超过Survivor空间*TargetSurvivorRatio的时候，从该年龄段开始及大于的年龄对象就要进入老年代（即例子中的年龄6对象，就是年龄6和年龄7晋升到老年代），这时候无需等到MaxTenuringThreshold中要求的15

##### 如何判断一个对象需要被干掉

​            ![img](https://camo.githubusercontent.com/0a581ddfeae6d7077cd8d3683ca20595eb4892547eaa0b6ae92a73a96e6ca669/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d31312f31633164383562356662386234373233396166326135633034333665623264372d6e65772d696d61676530636431303832372d326639362d343333632d396231362d3933643466653439316438382e706e67)

## 学习中的一些问题

### 在使用构造器为类成员变量传递参数时出现问题

竟然不知道private成员赋值要加上this.  

凸显了抄程序的坏处，也说明最基础的东西过了遍数多就以为自己记住了一些最基础的东西

使用构造函数为私有类成员变量赋值，不论方法或者变量加上static关键字就成为类共有内容

调用时都需要使用类名.static_fangfa_or_variable

### 写的java程序启动时有几个线程

```java
public class TestOne {
    public static void main(String[] args) {
        ThreadMXBean mxBean = ManagementFactory.getThreadMXBean();
        ThreadInfo[] allThreads = mxBean.dumpAllThreads(false, false);
        for (ThreadInfo threadInfo : allThreads) {
            System.out.println(threadInfo.getThreadId()+"==="+threadInfo.getThreadName());

        }
    }
    }
```

![image-20210327223617509](../images/java_learing_temp-writing/image-20210327223617509.png)

win10 家庭版  idea 2020.1.3 版本 jdk1.8 版本 下 测得

### 类加载机制

[参考](https://juejin.cn/post/6844903564804882445)

#### 在类中的一些进行过程

类中 ，成员变量赋值第一个进行，其次时静态构造函数（静态代码块，静态关键字定义的），再次时构造函数

#### 类加载过程

![类加载过程](https://camo.githubusercontent.com/7078336ec98b1f79e04e9fc60f514915f723c042c99e20cccb5d8bc18d45a3c6/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f2545372542312542422545352538412541302545382542442542442545382542462538372545372541382538422e706e67)

Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的加载机制。* Class文件由类装载器装载后，在JVM中将形成一份描述Class结构的元信息对象，通过该元信息对象可以获知Class的结构信息：如构造函数，属性和方法等，Java允许用户借由这个Class相关的元信息对象间接调用Class对象的功能,这里就是我们经常能见到的Class类。

一个非数组类的加载阶段（加载阶段获取类的二进制字节流的动作）是可空性最强的阶段，这一步我们去完成还可以自定义加载器去控制字节流的获取方式（重写一个类加载器的loadclass()方法）数组类型不通过类加载器创建，它由Java虚拟机直接创建

##### **类加载器**

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`：

1. **BootstrapClassLoader(启动类加载器)** ：最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者或被 `-Xbootclasspath`参数指定的路径中的所有类。

2. **ExtensionClassLoader(扩展类加载器)** ：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。

3. **AppClassLoader(应用程序类加载器)** ：面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。
   
    **双亲委派模型**
   
    系统中的ClassLoader在协同 工作时会默认使用**双亲委派模型**，即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。加载的时候，首先会把该请求委派该父类加载器的 `loadClass()` 处理，因此所有的请求最终都应该传送到顶层的启动类加载器 `BootstrapClassLoader` 中。当父类加载器无法处理时，才由自己来处理。当父类加载器为null时，会使用启动类加载器 `BootstrapClassLoader` 作为父类加载器。

当一个类收到了加载请求时，它是不会自己尝试加载的，而是委派给父亲去完成。

比如new一个自定义类Person ，如果我们要加载它，就会先委派App ClassLoader ，只有当父类加载器都反馈自己无法完成这个请求（也就是父类加载器都没有找到加载所需的Class）时，子类加载器才会自行尝试加载

这样做的好处是，加载位于rt.jar包中的类时不管是哪个加载器加载，最终都会委托到BootStrap ClassLoader进行加载，这样保证了使用不同的类加载器得到的都是同一个结果。

其实这个也是一个隔离的作用，避免了我们的代码影响了JDK的代码

##### 加载

.class文件二进制数据读入内存中，放在运行时数据区的方法区内，在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。 类加载的最终产品是位于堆区中的Class对象，Class对象封装了装在方法取的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口

类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

加载.class文件的方式有：

1). 从本地系统中直接加载 2). 通过网络下载.class文件 3). 从zip，jar等归档文件中加载.class文件 4). 从专有数据库中提取.class文件 5). 将Java源文件动态编译为.class文件

jvm虚拟机在类加载阶段需要完成三件事情：

1. 通过一个类的全限定名称来获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法取的运行时数据结构
3. 在Java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口

加载阶段是开发期相对来说可控性较强的阶段

##### 验证

为了确保Class文件中的字节流中信息符合当前虚拟机的要求，而且不会危害虚拟机自身的安全，不同的虚拟机对类验证的实现可能会有所不同，但大致都会完成一下四个阶段的验证：文件格式的验证、元数据的验证、字节码验证和符号引用验证。

1）文件格式的验证：验证字节流是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理，该验证的主要目的是保证输入的字节流能正确地解析并存储于方法区之内。经过该阶段的验证后，字节流才会进入内存的方法区中进行存储，后面的三个验证都是基于方法区的存储结构进行的。

2）元数据验证：对类的元数据信息进行语义校验（其实就是对类中的各数据类型进行语法校验），保证不存在不符合Java语法规范的元数据信息。

3）字节码验证：该阶段验证的主要工作是进行数据流和控制流分析，对类的方法体进行校验分析，以保证被校验的类的方法在运行时不会做出危害虚拟机安全的行为。

4）符号引用验证：这是最后一个阶段的验证，它发生在虚拟机将符号引用转化为直接引用的时候（解析阶段中发生该转化，后面会有讲解），主要是对类自身以外的信息（常量池中的各种符号引用）进行匹配性的校验。

##### 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中进行分配

注：

1）这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。

2）这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、null、false等），而不是被在Java代码中被显式地赋予的值。

##### 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

符号引用（Symbolic Reference）：符号引用以一组符号来描述所引用的目标，符号引用可以是任何形式的字面量，符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经在内存中。

直接引用（Direct Reference）：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现的内存布局相关的，同一个符号引用在不同的虚拟机实例上翻译出来的直接引用一般都不相同，如果有了直接引用，那引用的目标必定已经在内存中存在。

1)、类或接口的解析：判断所要转化成的直接引用是对数组类型，还是普通的对象类型的引用，从而进行不同的解析。 2)、字段解析：对字段进行解析时，会先在本类中查找是否包含有简单名称和字段描述符都与目标相匹配的字段，如果有，则查找结束；如果没有，则会按照继承关系从上往下递归搜索该类所实现的各个接口和它们的父接口，还没有，则按照继承关系从上往下递归搜索其父类，直至查找结束。

3)、类方法解析：对类方法的解析与对字段解析的搜索步骤差不多，只是多了判断该方法所处的是类还是接口的步骤，而且对类方法的匹配搜索，是先搜索父类，再搜索接口。

4)、接口方法解析：与类方法解析步骤类似，只是接口不会有父类，因此，只递归向上搜索父接口就行了。

##### 初始化

类初始化时类加载过程的最后一步，前面的类加载过程中，除了加载（Loading）阶段用户应该用程序

可以通过自定义类加载器参与之外，其余动作完全由虚拟机主导和控制。到了初始化阶段，才真正开始执行类中定义的Java程序代码。初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。在Java对类变量进行初始值设定有两种方式；

1. 声明类变量时指定初始值
2. 使用静态代码块为变量指定初始值‘

JVM初始化步骤

1. 假如这个类还没有被加载和连接，则程序先加载并连接该类
2. 假如该类直接父类还没有被初始化，则先初始化其直接父类
3. 加入类中有初始化语句，则系统依次执行这些初始化语句

初始化阶段时执行类构造器方法的过程

1）类构造器方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序所决定。

2）类构造器方法与类的构造函数不同，它不需要显式地调用父类构造器，虚拟机会保证在子类的类构造器方法执行之前，父类的类构造器方法已经执行完毕，因此在虚拟机中第一个执行的类构造器方法的类一定是java.lang.Object。

3）由于父类的类构造器方法方法先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作。

4）类构造器方法对于类或者接口来说并不是必需的，如果一个类中没有静态语句块也没有对变量的赋值操作，那么编译器可以不为这个类生成类构造器方法。

5）接口中可能会有变量赋值操作，因此接口也会生成类构造器方法。但是接口与类不同，执行接口的类构造器方法不需要先执行父接口的类构造器方法。只有当父接口中定义的变量被使用时，父接口才会被初始化。另外，接口的实现类在初始化时也不会执行接口的类构造器方法。

6）虚拟机会保证一个类的类构造器方法在多线程环境中被正确地加锁和同步。如果有多个线程去同时初始化一个类，那么只会有一个线程去执行这个类的类构造器方法，其它线程都需要阻塞等待，直到活动线程执行类构造器方法完毕。如果在一个类的类构造器方法中有耗时很长的操作，那么就可能造成多个进程阻塞。

##### 结束生命周期

以下情况时，Java虚拟机会结束生命周期

1. 执行了System.exit()方法 
2. 程序执行结束
3. 程序在执行过程中遇到了异常或错误而异常终止
4. 由于操作系统出现错误而导致Java虚拟机进程结束

#### 类的初始化

类加载过程第一个阶段"加载"。虚拟机规范中没有强行约束 ，这点交给虚拟机的具体实现自由把握，但是对于初始化阶段虚拟机规范严格如下情况，如果类未初始化会对类进行初始化

1、创建类的实例

2、访问类的静态变量(除常量【被final修辞的静态变量】原因:常量一种特殊的变量，因为编译器把他们当作值(value)而不是域(field)来对待。如果你的代码中用到了常变量(constant variable)，编译器并不会生成字节码来从对象中载入域的值，而是直接把这个值插入到字节码中。这是一种很有用的优化，但是如果你需要改变final域的值那么每一块用到那个域的代码都需要重新编译。

3、访问类的静态方法

4、反射如(Class.forName("my.xyz.Test"))

5、当初始化一个类时，发现其父类还未初始化，则先出发父类的初始化

6、虚拟机启动时，定义了main()方法的那个类先初始化

以上情况称为称对一个类进行“**主动引用**”，除此种情况之外，均不会触发类的初始化，称为“被动引用” 接口的加载过程与类的加载过程稍有不同。接口中不能使用static{}块。当一个接口在初始化时，并不要求其父接口全部都完成了初始化，只有真正在使用到父接口时（例如引用接口中定义的常量）才会初始化。

**被动引用例子**

1、子类调用父类的静态变量，子类不会被初始化。只有父类被初始化。。对于静态字段，只有直接定义这个字段的类才会被初始化.

2、通过数组定义来引用类，不会触发类的初始化

3、 访问类的常量，不会初始化类

```java
class Super class{
    static{
        System.out.println("superclass init");
    }
    public static int value = 123
}
class SubClass extends SuperClass {  
    static {  
        System.out.println("subclass init");  
    }  
}  

public class Test {  
    public static void main(String[] args) {  
        System.out.println(SubClass.value);// 被动应用1  
        SubClass[] sca = new SubClass[10];// 被动引用2  
    }  
}  

//运行结果 输出 superclass init 123 
```

```java
class ConstClass {  
    static {  
        System.out.println("ConstClass init");  
    }  
    public static final String HELLOWORLD = "hello world";  
}  

public class Test {  
    public static void main(String[] args) {  
        System.out.println(ConstClass.HELLOWORLD);// 调用类常量  
    }  
}  

//运行结果 输出 hello world
```

##### 类初始化顺序

触发类的初始化规则卸载Java语言规范中，但了解清楚 域（fields，静态的还是非静态的）、块（block静态的还是非静态的）、不同类（子类和超类）和不同的接口（子接口，实现类和超接口）的初始化顺序也很重要类。

```txt
1.类从顶至底的顺序初始化，所以声明在顶部的字段的早于底部的字段初始化
2.超类早于子类和衍生类的初始化
3.如果类的初始化是由于访问静态域而触发，那么只有声明静态域的类才被初始化，而不会触发超类的初始化或者子类的4.初始化即使静态域被子类或子接口或者它的实现类所引用。
5.接口初始化不会导致父接口的初始化。
6.静态域的初始化是在类的静态初始化期间，非静态域的初始化时在类的实例创建期间。这意味这静态域初始化在非静态域之前。
7.非静态域通过构造器初始化，子类在做任何初始化之前构造器会隐含地调用父类的构造器，他保证了非静态或实例变量（父类）初始化早于子类
```

### Calendar创建多个getinstance 实例

在学习Calender的使用时，存在着  

```java
 Calendar c1 = Calendar.getInstance();
 Calendar c2 = Calendar.getInstance();
```

这样的用法， 而getinstance()方法创建的是单例，也就是一个类有着一个共同的对象，其它调用都是对这个对象的引用

     Calendar本身是个抽象类，是多个日期实现类的父类，提供了Calendar.getInstance()给我们方便调用

```java
GregorianCalendar c1 = new GregorianCalendar();
```

也是可以创建日期类的实例，但破坏了封装性和多态性，所以应该根据不同的环境使用不同的实现类

### add()  需要list 对象 初始化

![image-20210413174008081](../images/java_learing_temp-writing/image-20210413174008081.png)

在这里  commodities 利用 ArrayList 实现了空间的分配 ,所以下面才得以使用 add() 函数进行添加commodity对象 

![image-20210413174115248](../images/java_learing_temp-writing/image-20210413174115248.png)

下面将其修改为 null 

![image-20210413174221553](../images/java_learing_temp-writing/image-20210413174221553.png)

空指针 错误 ,  空对象 , 需要先将 list 实例化

![image-20210413174346851](../images/java_learing_temp-writing/image-20210413174346851.png)

这里 getAllCommodity()方法的返回值类型为 List<commodity>   ![image-20210413174513349](../images/java_learing_temp-writing/image-20210413174513349.png)

list对象 ,所以直接 " = " 获得其对象引用地址 
