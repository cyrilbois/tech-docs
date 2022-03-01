---
title:  Java编码约定
...

代码约定提高了软件的可读性，使工程师能够更快、更彻底地理解彼此的代码。 
本文参考Google Java 代码规范 和 Oracle 的 Code Conventions 结合自己项目中的思考合并而来。


## 1 源文件基础

### 1.1 编码格式：UTF-8
源文件采用UTF-8编码

### 1.2 特殊字符

#### 1.2.1 转义字符
对于任何有特殊意义的转义序列应当用(\b,\t,\n,\f,\r,",'和\),而不是用八进制转义符(e.g.\012)或Unicode转义符(e.g.\u000a)
#### 1.2.2 非ASCII字符
对于非ASCII字符，采用Unicode字符(e.g.∞)或Unicode转义符(e.g.\u221e)。这取决于那种方式更易于理解，建议不要才有Unicode转义符，即使有注释。

示例
```
Example	Discussion
String unitAbbrev = "μs";	最优解：清晰明了甚至不需要注释
String unitAbbrev = "\u03bcs"; // "μs"	允许：但是没必要这么做
String unitAbbrev = "\u03bcs"; // 希腊字母mu，"s"	允许：但是笨拙且易出错
String unitAbbrev = "\u03bcs";	不建议：不易理解
return '\ufeff' + content; // 字节顺序标记	良好：对于不会打印出的字符在需要的时候提供了注释
```
不要担心一些程序不能处理非ASCII字符而使得你的程序不可读，如果发生这种情况，程序将会被破坏，会有人修复的。

















## 2 源文件结构
### 2.1 头部内容
Java源文件头部按顺序包括：

* 许可或版权信息（如果有的话）
* 包声明 （不换行）
* 导入声明 （不换行，禁用通配符的方式导入，不导入内部类）
* 类说明

### 2.2  类中内容的顺序
1. 禁止添加方法的时候直接添加在类的最后。
2. 相似方法、类似业务处理代码放在一起。 
3. 重载：永远放在一起。 当一个类有多个构造器或多个同名方法时，它们应该依次放在一起，中间没有其他代码，甚至是私有成员。













## 3 代码格式
术语小贴士：块状构造是指类，方法或构造函数的主体。注意，根据第4.8.3.1节关于数组初始化程序的介绍，可以选择将任何数组初始化程序视为类似于块的构造。

> 格式尽量沿用主流IDE IntelliJ的默认格式，避免在这上面花费过多精力。

### 3.1 大括号


#### 3.1.1 大括号用于可选的地方
大括号和if,else,for,do还有while语句一起使用，即使代码块儿为空或只有一行。

#### 3.1.2 非空块儿：K & R风格
在非空块儿和块状构造中，大括号准守Kernighan和Ritchie风格（“埃及风格”）：

* 左括号前不换行
* 左括号后换行
* 右括号前换行
* 仅在大括号终止语句或终止方法，构造函数或命名类的主体时，才在右括号后换行。例如，如果大括号后跟着else或逗号，则没有换行符。

> 采用这种方式也是因为这个是大部分IDE (如: IntelliJ)默认格式。

```
return () -> {
    while (condition()) {
        method();
    }
};
return new MyClass() {
    @Override public void method() {
        if (condition()) {
            try {
                something();
            } catch (ProblemException e) {
                recover();
            }
        } else if (otherCondition()) {
            somethingElse();
        } else {
            lastThing();
        }
        {
            int x = foo();
            frob(x);
        }
    }
};
```

#### 3.1.3 空块儿：可以简介一些
一个空的块或构造块儿可以按K&R风格，也可以把左右括号紧挨着，中间没有字符或空行，除非这时多个块中的一部分（一个直观的例子就是代码块儿：if/else或try/catch/finally）。

例如：

```
// 这时被允许的
void doNothing() {}

// 这也是被允许的
void doNothingElse() {
}

// 这是不允许的：在多块儿代码中不是简洁代码
try {
    doSomething();
} catch (Exception e) {}
```

### 3.2 块儿缩进：四个空格
每个新块儿或块状构造中，都要缩进四个空格。当块结束时，缩进返回到原来的状态。缩进适用于代码和注释。（参见3.1.2，非空代码块儿：K&R风格）






### 3.3 每句代码占一行
每句代码后面跟一个换行符。


### 3.4 每行字数限制：100
Java代码限制每行最多100个字符。一个字符就是任意的一个Unicode单元，除非另有说明，任何超过限制的代码都应该换行，就像3.5换行解释的那样。每个Unicode单位算一个字符，不管其展示如何。例如，在全角模式下，你可能比半角模式更早的换行。

例外情况：

* 不是每一行都必须遵守字数限制（例如，Javadoc中的长URL或长JSNI方法引用）
* 包声明和导入声明（详见，3.2包声明和3.3导入声明）
* 换行会导致不在注释区的注释



### 3.5 换行原则


1.  在非运算符处换行，具体细节可以参考Google的说明
2.  换行比原行缩进至少4个空格；IntelliJ默认8个
3.  当有多行换行时，通常，两条连续在当它们在句法上平行时使用相同的缩进级别。










### 3.6 空行

空行根据逻辑来添加空行， 禁止连续两个空行

### 3.7 横向空格
除了语言或其他样式规则的要求之外，除了文字，注释和Javadoc外，单个ASCII空间也仅出现在以下位置：

1. 将保留字例如if,for或catch与后面的左括号（(）分开。如： `if (condition)`
2. 将保留字例如else或catch与前面的右括号（}）分开
3. 在所有左大括号前（{），有两个例外： `SomeAnnotation({a, b})`   和  `String[][] x = {{"foo"}};`
4.  在二元或三元操作符两边: `if (userInfoStr != null) `

> 一般保持IntelliJ默认即可


### 3.8. 变量声明
* 每次声明一个变量
* 在需要的地方声明变量；尽量在靠近它第一次被使用的地方（也可能一组业务相关的变量，可提前放在一起生命）。变量声明要么附带赋值，要么马上赋值。

### 3.9  Switch语句

* 在switch块内，，每个语句必须通过break,continue,return或抛出异常）
* 提供default条件； 每个switch语句要包含一个default语句。即使后面没有代码。例外：如果包含了所有的枚举情况，枚举的switch语句可以省略default。IDE或者静态检查工具可以分析出是否漏掉了某些情况。

### 3.10 注释
本节主要讲普通注释。Javadoc在后面单独讲。
任何换行符前都可以加任意的空格然后添加普通注释，这中注释使这行成为非空白行。

#### 3.10.1 块儿注释风格
块儿注释和它所注释的代码保持相同的缩进。他们可能是/* ... */或// ...。对于多行/* ... */注释后续行必须以*开头并且与上一行的*对齐。

```
/*
 * This is          
 * okay.            
 */

 // And so          
 // is this.          

  /* Or you can
   * even do this. */
 ```
注释不包含在有型号或其他字符的框中。

> 当写多行注释时，如果你想在注释自动换行时。使用/* ... */。大多数格式化工具对// ...没有处理。

### 3.11 修饰符
如果有类和成员的修饰符，按Java语言的推荐顺序来写：

public protected private abstract default static final transient volatile synchronized native strictfp
### 3.12 数字
long类型的值用大写字母L，不要用小写l（避免和数字1混淆）,例如，使用3000000000L而不是3000000000l

## 4 命名
### 4.1 所有标识的通用规则
标识使用ASCII字母或数字并且仅在少量的情况下使用下划线。

### 4.2 标识类型的规则
##### 4.2.1 包名
包名采用全小写，通过连续的简单单词组合在一起（没有下划线），例如采用com.example.deepspace，而不是com.example.deepSpace或者com.exaple.deep_space。

#### 4.2.2 类名
* 类名采用大写驼峰UpperCamelCase 。
* 类名通常是名称或名词短语。例如，Charater或ImmutabelList。
* 接口名字也可能是名词或名词短语（例如，List），但有时也可能是形容词或形容词短语。
* 注解的命名没有特殊的或者已经完备的规则。
* 测试类命名以被测试类名开头，以Test结尾。例如,HashTest或者HashIntegrationTast。

#### 4.2.3 方法名
* 方法名采用小写小写驼峰lowerCamelCase 。
* 方法名通常是动词或动词短语。例如，sendMessage或stop 。
* 下划线在JUnit测试方法中划分不同的逻辑部分，每个部分用小写的驼峰来写。通常采用<methodUnderTest>_<state>，例如pop_emptyStack。对于测试方法没唯一的正确名命方法。

#### 4.2.4 常量名
常量名使用CONSTANT_CASE：全大写字母，单词之间使用下划线分割。什么是常量呢？
常量是静态的final字段，其值确定并且方法不能改变。包括，基本类型，字符串，不可变类型，以及不可变类型的集合。如果有一个实例可以改变，那这个实例不是常量。一般的变量即使不打算改变其值也不是常量。例如：

```
// 常量
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final ImmutableMap<String, Integer> AGES = ImmutableMap.of("Ed", 35, "Ann", 32);
static final Joiner COMMA_JOINER = Joiner.on(','); // because Joiner is immutable
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }
```
```
// 非常量
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final ImmutableMap<String, SomeMutableType> mutableValues =
    ImmutableMap.of("Ed", mutableInstance, "Ann", mutableInstance2);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```
他们的名字通常都是名词或名词短语

//全局常量即：跨类使用的常量，采用接口的方式分组定义

```
/**
 * @author : Joe
 * @date : 2021/1/14
 */
public interface Constants {

    Long DISTRIBUTOR_ID_WX = 931L;//小程序
    interface Log{
        String TRACE_ID = "X-TraceId";
        String BIZTYPE = "X-BizType";
        String KEYID = "X-KeyId";
    }

    interface UserName{
        String ADMIN_USER = "adminUsername";
        String ANNON_USER = "annon";
    }
}
```

> 注意考虑面向对象的封装性，单个类使用的常量直接定义在其类中即可。

#### 4.2.5 非常量名 参数名 局部变量
采用小驼峰。

#### 4.2.6 泛型名
泛型遵守以下规则之一：

* 单个大写字母，后面可以跟单个数字（例如E, T, X, T2）
* 以类的形式命名，后面跟上大写字母T(例如：RequestT,FooBarT)。

### 4.3 驼峰：定义

![CamelCase.png](http://tech.jiu-shu.com/Java-Technologies/CamelCase.png)


## 5 编码实践
6.1 @Override:必须添加
方法在可以添加@Override的时候添加上他。这包括重写父类方法，实现接口方法。一个接口方法重新指定父接口方法。
例外：当父方法被标记为@Deprecated时@Override可以被省略。

6.2 捕获异常：不可忽略
除以下情况外，不处理异常是不正确的。（典型的是打印日志，如果认为这种情况不可能出现，重新以AssertionError的形式抛出。）
当在捕获块中确实应当什么都不做的话，原因应该写在注释里面。

try {
  int i = Integer.parseInt(response);
  return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  // 这不是数字，正常，继续执行
}
return handleTextResponse(response);
例外：在测试中，如果捕获的异常为预期异常或者以预期开通可以在没有注释的情况下省略。下面是确保测试代码抛出预期异常，所以注释在这里不是必要的。

try {
  emptyStack.pop();
  fail();
} catch (NoSuchElementException expected) {
}
6.3 静态成员：通过类引用
当使用一个静态的类成员，应当使用该类的名词而不是该类对象的引用。

Foo aFoo = ...;
Foo.aStaticMethod(); // 好
aFoo.aStaticMethod(); // 坏
somethingThatYieldsAFoo().aStaticMethod(); // 非常坏
6.4 回收器：不要用
极少重写Object.finalize。

提示：不要这么做，如果你必须这么做请先仔细阅读《Effective Java Item 7》,"Avoid Finalizers"。然后放弃这么做。

7 Javadoc
7.1 格式化
7.1.1 一般格式
在此示例中可以看到一般的JavaDoc块的格式：

/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String p1) { ... }
或者单行的例子

/** An especially short bit of Javadoc. */
基本格式总是可以接受的。当JavaDoc可以在一行内放下时可以采用单行格式代替。注意，这仅适用于没有@return之类的标签的时候。

7.1.2 段落
段落之间以一个空行--仅在这行开头有一个星号*，隔开。除第一个单词外，每个段落的第一个单词前面加上<p>后面没有空格。

7.1.3 块标签
所有的“块标签”按顺序出现@param,@return,@throws,@desprecate。这四种标签都不会出现空描述，当一行放不下的时候，下一行缩进到@后面四个或者更多空格的地方。

7.2 摘要碎片
每个JavaDoc都会以一个简短的摘要碎片开始。这个碎片非常重要：它是上下文的一部分文本，例如类和方法的索引。
这是一个碎片，是一个名词或动词，而不是完整的句子，它不以A {@code Foo} is a...或者This method returns...开头，也不会形成完整的命令式句子Save the record.，但是该碎片被大写并且被标点，就像他是完整句子一样。

	提示：一个常见的错误是简单JavaDoc被写成了这种格式/** @return the customer ID */。这是不正确的，应当写成/** Returns the customer ID. */。

7.3 哪里需要Javadoc
至少，在public的类中，每个public或protect成员需要JavaDoc，下面有少数特殊情况。
如第7.3.4节“不需要的Javadoc”中所述，还可能存在其他Javadoc内容。

7.3.1 例外：不言自明的方法
Javadoc在一个简单明显的方法像getFoo。如果真的没有其他话，除了“返回foo”之外，没有什么好说的。

7.3.2 例外：重写
对于重写的方法，不必总是写Javadoc

7.3.3 不需要的Javadoc
当其他类或成员有需要的Javadoc
当注释被用来描述类或方法的目的或行为时，注释应当写成Javadoc的形式（使用/**）
7.1.2和7.1.3中的注释中的非必要Javadoc可以省略，既是是建议的。



