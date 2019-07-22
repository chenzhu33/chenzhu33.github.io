# 字节码详述与动态修改实践

## 1.概述

本文将对Java字节码进行详细分析介绍，并介绍两种主流的字节码修改工具：JavaAssist/ASM。

### 目录
1. 深入理解Java字节码格式
2. ASM字节码修改实践
3. JavaAssist修改实践


## 2.深入理解Java字节码（Class）格式

### 2.1.Class文件概述

class文件是一种8位字节的二进制流文件，各个数据项按顺序紧密的从前向后排列，相邻的项之间没有间隙，这样可以使得class文件非常紧凑， 体积轻巧，可以被JVM快速的加载至内存，并且占据较少的内存空间。Java源文件被编译之后，每个类（或者接口）都单独占据一个class文件，并且类中的所有信息都会在class文件中有相应的描述，由于class文件很灵活，它甚至比Java源文件有着更强的描述能力。

Class文件格式如下：

| 类型 | 名称    | Size  | 描述
| --- |:-------:|:-----:|---:|
|u4 | magic  | 1 | 固定为OXCAFEBABE，标志该文件是class文件
|u2 | minor_version | 1 | 主版本号
|u2 | major_version | 1 | 次版本号
|u2 | constant_pool_count | 1 | 常量池Size
|cp_info|	constant_pool | constant_pool_count - 1 | 常量池
|u2 | access_flags	| 1 | 访问标志
|u2 | this_class	| 1 | 当前类常量池索引
|u2 | super_class	| 1 | 父类常量池索引
|u2 | interfaces_count	| 1 | 接口size
|u2 | interfaces	| interfaces_count | 接口
|u2 | fields_count	| 1 | 成员变量size
|field_info	| fields	| fields_count | 成员变量
|u2 | methods_count	| 1 | 方法size
|method_info	| methods	| methods_count | 方法
|u2 | attribute_count	| 1 | 属性size
|attribute_info	| attributes	| attributes_count | 属性

### 2.2.常量池概念

常量池存储了诸如符号常量、final常量值、基本数据类型的字面值等内容。JVM会将每一个常量构成一个常量表，每个常量表都有自己的入口地址。而实际上在JVM会将这些常量表存储在方法区中一块连续的内存空间中，因此class文件会根据常量表在常量池中的位置对其进行索引。比如常量池中的第一个常量表的索引值就是<font color=red>**1**</font>
，第二个就是2。有的时候常量表A需要常量表B的内容，则在常量表A中会存储常量表B的索引值x。而constant_pool_count就记录了有多少个常量表，或则所有多少个索引值。实际上，常量池中没有索引值为0的常量表，但这缺失的索引值也被记录在constant_pool_count中，因此constant_pool_count等于常量表的数量加1。

##### 一个例子
```
public class ClassTest {
	private String itemS = "str";
	private final int itemI = 100;
	public void setItemS(String para){...}
}
```

Java中，除了所熟知的String常量，还有许多东西会存放在常量池中，比如一个类的名字，一个类字段的名字/所属类型，一个类方法的名字/返回类型/参数名与所属类型，一个常量，还有在程序中出现的大量的字面值。上面程序中的`ClassTest`/`String itemS="str"`/`int itemI = 100`/`void setItemS`/`String para`都会存在在常量池中。而这些常量之间又有不同，class文件共有11种常量表，如下所示：

|常量表类型|标志值(占1 byte)|描述
| --- |:-------:|:-----:|
|CONSTANT_Utf8 | 1 | UTF-8编码的Unicode字符串
|CONSTANT_Integer | 3 | int类型的字面值
|CONSTANT_Float | 4|float类型的字面值
|CONSTANT_Long | 5|long类型的字面值
|CONSTANT_Double|6|double类型的字面值
|CONSTANT_Class|7|对一个类或接口的符号引用
|CONSTANT_String|8|String类型字面值的引用
|CONSTANT_Fieldref|9|对一个字段的符号引用
|CONSTANT_Methodref|10|对一个类中方法的符号引用
|CONSTANT_InterfaceMethodref|11|对一个接口中方法的符号引用
|CONSTANT_NameAndType|12|对一个字段或方法的部分符号引用


1. `CONSTANT_Utf8` 用UTF-8编码方式来表示程序中所有的重要常量字符串。这些字符串包括：①类或接口的全限定名，②超类的全限定名，③父接口的全限定名，④类字段名和所属类型名，⑤类方法名和返回类型名、以及参数名和所属类型名。⑥字符串字面值

	表格式：   `tag`(标志1：占1byte)  `length`(字符串所占字节的长度，占2byte)      bytes(字符串字节序列)

2. `CONSTANT_Integer`、`CONSTANT_Float`、 `CONSTANT_Long`、`CONSTANT_Double` 所有基本数据类型的字面值。比如在程序中出现的1用`CONSTANT_Integer`表示。3.1415926F用`CONSTANT_Float`表示。 

	表格式：   `tag`    `bytes`(基本数据类型所需使用的字节序列)

3. `CONSTANT_Class` 使用符号引用来表示类或接口。我们知道所有类名都以`CONSTANT_Utf8`表的形式存储。但是我们并不知道`CONSTANT_Utf8`表中哪些字符串是类名，那些是方法名。因此我们必须用一个指向类名字符串的符号引用常量来表明。

	表格式：   `tag`    `name_index`(给出表示类或接口名的`CONSTANT_Utf8`表的索引)

4. `CONSTANT_String` 同`CONSTANT_Class`，指向包含字符串字面值的`CONSTANT_Utf8`表。
	
	表格式：   `tag`    `string_index`(给出表示字符串字面值的`CONSTANT_Utf8`表的索引)

5. `CONSTANT_Fieldref`、`CONSTANT_Methodref`、`CONSTANT_InterfaceMethodref` 指向包含该字段或方法所属类名的`CONSTANT_Utf8`表，以及指向包含该字段或方法的名字和描述符的`CONSTANT_NameAndType`表

	表格式：   `tag`   `class _index`(给出包含所属类名的`CONSTANT_Utf8`表的索引)  `name_and_type_index` (包含字段名或方法名以及描述符的`CONSTANT_NameAndType`表的索引)

 

6. `CONSTANT_NameAndType` 指向包含字段名或方法名以及描述符的`CONSTANT_Utf8`表。

	表格式：   `tag`    `name_index`(给出表示字段名或方法名的`CONSTANT_Utf8`表的索引)  `type_index`(给出表示描述符的`CONSTANT_Utf8`表的索引)

### 2.3.常量池中的特殊字符

特殊字符串是常量池中符号引用的一部分。特殊字符串包括三种：类的全限定名，字段和方法的描述符，特殊方法的方法名。下面我们就分别介绍这三种特殊字符串。

1. 类的全限定名

	在常量池中，一个类型的名字并不是我们在源文件中看到的那样，也不是我们在源文件中使用的包名加类名的形式。**源文件中的全限定名**和**class文件中的全限定名**不是相同的概念。源文件中的全新定名是包名加类名，包名的各个部分之间，包名和类名之间，使用点号分割。如Object类，在源文件中的全限定名是`java.lang.Object`。而class文件中的全限定名是将点号替换成"/"。例如，Object类在class文件中的全限定名是`java/lang/Object`。

2. 描述符

	我们知道在一个类中可以有若干字段和方法，在class文件中，方法和字段的描述符并不会把方法和字段的所有信息全都描述出来，描述符只是一个简单的字符串。在讲解描述符之前，要先明确一个问题：**所有的类型在描述符中都有对应的字符或字符串来对应**。

	**2.1. 基本数据类型与Void**
	
	|基本数据类型和void类型|类型的对应字符|
	| --- |:--|
	|byte|B|
	|char|C|
	|double|D|
	|float|F|
	|int|I|
	|long|J|
	|short|S|
	|boolean|Z|
	|void|V|

	基本上都是以类型的首字符变成大写来对应的，其中long和boolean是特例，long类型在描述符中的对应字符是J，boolean类型在描述符中的对应字符是Z。
	
	**2.2. 引用类型**
	
	引用类型的**对应字符串**格式为：`“L” + 类型的全限定名 + “;” `

	如Object在描述符中的对应字符串是：`Ljava/lang/Object;`，ArrayList在描述符中的对应字符串是：`Ljava/lang/ArrayList;`，自定义类型`com.example.Person`在描述符中的对应字符串是：`Lcom/example/Person;`。
	
	**2.3. 数组类型**
	
	一个数组的元素类型和他的维度决定了他的类型。比如，在`int[] a`声明中， 变量a的类型是int[]，在`int[][] b`声明中， 变量b的类型是int[][]。在class文件的描述符中，数组的类型中每个维度都用一个`[`代表， 数组类型整个类型的对应字符串的格式如下：
	
	`若干个"["  +  数组中元素类型的对应字符串`
	
	因此int[]类型的对应字符串是：`[I`。int[][]类型的对应字符串是：`[[I`。Object[]类型的对应字符串是：`[Ljava/lang/Object;`。Object[][][]类型的对应字符串是：`[[[Ljava/lang/Object;`。

	**2.4. 字段和方法的描述符**
	
	字段的描述符就是字段的类型所对应的字符或字符串。如：int i中， 字段i的描述符就是`I`。Object o中，字段o的描述符就是`Ljava/lang/Object`; 。double[][] d中，字段d的描述符就是`[[D`。
	
	方法的描述符包括所有参数的类型列表和方法返回值。格式为：
	
	`(参数1类型 参数2类型 参数3类型 ...)返回值类型`
	
	其中，不管是参数的类型还是返回值类型，都是使用对应字符和对应字符串来表示的，参数列表使用小括号括起来，各个参数类型之间**没有空格**，参数列表和返回值类型之间也没有空格。例如：
	
	|方法描述符|方法声明|
	| --- |:--|
	|()I|int getSize()|
	|()Ljava/lang/String;|String toString()|
	|([Ljava/lang/String;)V|void main(String[] args)|
	|()V|void wait()|
	|(JI)V|void wait(long timeout, int nanos)|
	|(ZILjava/lang/String;II)Z|boolean regionMatches(boolean ignoreCase, int toOffset, String other, int ooffset, int len)|
	|([BII)I|int read(byte[] b, int off, int len )|
	| ()\[\[Ljava/lang/Object; | Object[][] getObjectArray()|
	
3. 特殊方法的方法名

	特殊方法是指的类的构造方法和类型初始化方法。构造方法就不用多说了，类型初始化方法，对应到源码中就是静态初始化块。也就是说，静态初始化块，在class文件中是以一个方法表述的，这个方法同样有方法描述符和方法名。
	
	类的构造方法的方法名使用字符串`<init>`表示，而静态初始化方法的方法名使用字符串 `<clinit>`表示。除了这两种特殊的方法外，其他普通方法的方法名，和源文件中的方法名相同。
	

### 2.4. class文件中的访问标志信息

#### 2.4.1. access_flags

位于常量池下面的2个字节是`access_flags`。`access_flags`描述的是当前类（或者接口）的访问修饰符，如public，private等，此外，这里面还存在一个标志位，标志当前的这个class描述的是类，还是接口。access_flags的信息比较简单，下面列出了access_flags 中的各个标志位的信息。

|标志名|标志值|标志含义|针对的对像|
| --- |:--|:--|:--|
|ACC_PUBLIC|0x0001|public类型|所有类型
|ACC_FINAL|0x0010|final类型|类
|ACC_SUPER|0x0020|使用新的invokespecial语义|类和接口
|ACC_INTERFACE|0x0200|接口类型|接口
|ACC_ABSTRACT|0x0400|抽象类型|类和接口
|ACC_SYNTHETIC|0x1000|该类不由用户代码生成|所有类型
|ACC_ANNOTATION|0x2000|注解类型|注解
|ACC_ENUM|0x4000|枚举类型|枚举

其中ACC_SUPER的含义是：使用新的invokespecial语义。invokespecial是一个字节码指令，用于调用一个方法，一般情况下，调用构造方法或者使用super关键字显示调用父类的方法时，会使用这条字节码指令。这正是ACC_SUPER这个名字的由来。在java 1.2之前，invokespecial对方法的调用都是静态绑定的，而ACC_SUPER这个标志位在java 1.2的时候加入到class文件中，它为invokespecial这条指令增加了动态绑定的功能。

还有一点需要说明，既然access_flags出现在class文件中的类的层面上，那么它只能描述类型的修饰符，而不能描述字段或方法的修饰符，和后面要介绍的方法表和字段表中的访问修饰符是不一样的。

#### 2.4.2. this_class

`this_class`是对当前类的描述。它的两个字节的数据是对常量池中的一个`CONSTANT_Class_info`数据项的一个索引。`CONSTANT_Class_info`中有一个字段叫做`name_index`，指向一个`CONSTANT_Utf8_info`，在这个`CONSTANT_Utf8_info`中存放着当前类的全限定名。下图给出了`this_class`的一个图例：

![image](http://img.blog.csdn.net/20140324000705203?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmdqZ19ibG9n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 2.4.3. super_class

`super_class`和`this_class`一样是一个指向常量池数据项的索引。它指向一个`CONSTANT_Class_info`，这个`CONSTANT_Class_info`数据项描述的是当前类的超类的信息。`CONSTANT_Class_info`中的`name_index`指向常量池中的一个`CONSTANT_Utf8_info`，`CONSTANT_Utf8_info`中存放的是当前类的超类的全限定名。 如果没有显式的继承一个，也就是说如果当前类是直接继承Object的，那么`super_class`值为0。如果一个索引值为0，那么就说明这个索引不引用任何常量池中的数据项。也就是说，如果一个类的class文件中的`super_class`为0，那么就代表该类直接继承Object类。

#### 2.4.4. interfaces_count和interfaces

`interfaces_count`，表示当前类所实现的接口的数量或者当前接口所继承的超接口的数量。注意，**只有当前类直接实现的接口才会被统计**，如果当前类继承了另一个类，而另一个类又实现了一个接口，那么这个接口不会统计在当前类的`interfaces_count`中。interfaces可以看做是一个数组，其中的每个数组项是一个索引，指向常量池中的一个`CONSTANT_Class_info`，这个`CONSTANT_Class_info`又会引用常量池中的一个`CONSTANT_Utf8_info`，其中存放着有当前类型直接实现或继承的接口的全限定名。当前类型实现或继承了几个接口，在interfaces数组中就会有几个数项与之相对应。

![image](http://img.blog.csdn.net/20140324005003265?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhbmdqZ19ibG9n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 2.4.5. fields_count和fields

