## 一、lambda表达式

### 1、list集合方面

排序

```java
 //排序  sorted((第一个对象，第二个对象)->返回值)
	List<Person> sortedList = list.stream().sorted((o1,o2)->o1.getAge()-o2.getAge()).collect(Collectors.toList());
  //排序ID
        List<User> userList1 = userList.stream().sorted(Comparator.comparingInt(User::getId)).collect(Collectors.toList());
        .sorted(Comparator.comparing(Person::getAge))//依年纪升序排序
        .sorted(Comparator.comparing(Person::getAge).reversed())//依年纪降序排序
```

map(), 提取对象中的某一元素.  用每一项来获得属性（也可以直接用  对象::get属性

```java
//获取所有ID集合
        List<Integer> idList = userList.stream().map(User::getId).collect(Collectors.toList());
 List<String> mapList2 = list.stream().map(item->item.getName()).collect(Collectors.toList());
```

filter为过滤

```
  //获取id大于2的
        List<User> userList2 = userList.stream().filter(user -> user.getId() > 2).collect(Collectors.toList());
```

.findAny()表示将其中任意一个返回；
.orElse(null)表示如果一个都没找到返回null

```java
User user6 =userList.stream().filter(user -> user.getId() > 2).findAny().orElse(null);
```

统计list中的数据

```java
//获取最大
Integer id = userList.stream().map(User::getId).max(Integer::compareTo).get();
//获取最小
Integer id1 = userList.stream().map(User::getId).min(Integer::compareTo).get();
//获取id数量
long count = userList.stream().map(User::getId).count();
//总和
int sum = userList.stream().mapToInt(User::getId).sum();
//获取平均值
double d = userList.stream().mapToInt(User::getId).average().getAsDouble();
```

打印list中的信息

```java
list.forEach((s)-> System.out.print(s+"\t"));
list.forEach(System.out::println);
```

其他方法

```java
  //分组统计
    Map<String, Long> map = userList.stream().collect(Collectors.groupingBy(User::getName, Collectors.counting()));
  //分组 Collectors.groupingBy(属性名)
 Map<Integer, List<Person>> map = list.stream().collect(Collectors.groupingBy(Person::getAge));
    //将名字全转换为大写
    List<String> list = userList.stream().map(User::getName).map(String::toUpperCase).collect(Collectors.toList());
    //获取忽略第一个并取前几条数据
    List<User> list1 = userList.stream().skip(1).limit(2).collect(Collectors.toList());
    //distinct() 去重；collect(Collectors.toList())。封装成集合
    List<User> collect = userList.stream().distinct().collect(Collectors.toList());
```

找出所有 **长度>=5**的字符串，并且**忽略大小写**、**去除重复字符串**，然后**按字母排序**，最后**用“❤”连接成一个字符串**输出！

```
 String result = list.stream().filter(i->!isNum(i))
                .filter(i->i.length()>=5).map(i->i.toLowerCase()).distinct()
                .sorted(Comparator.naturalOrder()).collect(Collectors.joining("❤"));
```



## 二、.Optional接口

### 1、使用方法

`Optional`本质是个容器，你可以将你的变量交由它进行封装，这样我们就不用显式对原变量进行 `null`值检测，防止出现各种空指针异常。举例：

我们想写一个获取学生某个课程考试分数的函数：`getScore()`

```
public Integer getScore(Student student) {
        if (student != null) {
			// 第一层 null判空
            Subject subject = student.getSubject();
            if (subject != null) {
				// 第二层 null判空
                return subject.score;
            }
        }
        return null;
    }
```

这样写倒不是不可以，但我们作为一个**“严谨且良心的”**后端工程师，这么多嵌套的 if 判空多少有点扎眼！

为此我们必须引入 `Optional`：

```
public Integer getScore(Student student) {
        return Optional.ofNullable(student).map(Student::getSubject).map(Subject::getScore).orElse(null);
    }
```

另一个例子：

```
Optional<Student> optional = getStudentByIdFromDB();
        optional.ifPresent(stu -> {
            System.out.println("学生姓名是：" + stu.getName());
        });
    }
```

`Optional<Student>` 作为结果，这样就表明 Student可能存在，也可能不存在，这时候就可以在 Optional 的 `ifPresent()` 方法中使用 Lambda 表达式来直接打印结果。

Optional 之所以可以解决 NPE 的问题，是因为它明确的告诉我们，不需要对它进行判空。它就好像十字路口的路标，明确地告诉你该往哪走。

### 2、创建 Optional 对象

1）可以使用静态方法 `empty()` 创建一个空的 Optional 对象

```
Optional<String> empty = Optional.empty();
System.out.println(empty); // 输出：Optional.empty
```

2）可以使用静态方法 `of()` 创建一个非空的 Optional 对象

```
Optional<String> opt = Optional.of("沉默王二");
System.out.println(opt); // 输出：Optional[沉默王二]
```

当然了，传递给 `of()` 方法的参数必须是非空的，也就是说不能为 null，否则仍然会抛出 NullPointerException。

```
String name = null;
Optional<String> optnull = Optional.of(name);
```

3）可以使用静态方法 `ofNullable()` 创建一个即可空又可非空的 Optional 对象

```
String name = null;
Optional<String> optOrNull = Optional.ofNullable(name);
System.out.println(optOrNull); // 输出：Optional.empty
```

`ofNullable()` 方法内部有一个三元表达式，如果为参数为 null，则返回私有常量 EMPTY；否则使用 new 关键字创建了一个新的 Optional 对象——不会再抛出 NPE 异常了。

### 3、判断值是否存在

可以通过方法 `isPresent()` 判断一个 Optional 对象是否存在，如果存在，该方法返回 true，否则返回 false——取代了 `obj != null` 的判断。

```
Optional<String> opt = Optional.of("沉默王二");
System.out.println(opt.isPresent()); // 输出：true

Optional<String> optOrNull = Optional.ofNullable(null);
System.out.println(opt.isPresent()); // 输出：false
```

### 4、非空表达式

Optional 类有一个非常现代化的方法——`ifPresent()`，允许我们使用函数式编程的方式执行一些代码，因此，我把它称为非空表达式。如果没有该方法的话，我们通常需要先通过 `isPresent()` 方法对 Optional 对象进行判空后再执行相应的代码：

```
Optional<String> optOrNull = Optional.ofNullable(null);
if (optOrNull.isPresent()) {
    System.out.println(optOrNull.get().length());
}
```

有了 `ifPresent()` 之后，情况就完全不同了，可以直接将 Lambda 表达式传递给该方法，代码更加简洁，更加直观。

```
Optional<String> opt = Optional.of("沉默王二");
opt.ifPresent(str -> System.out.println(str.length()));
```

### 5、设置（获取）默认值

有时候，我们在创建（获取） Optional 对象的时候，需要一个默认值，`orElse()`和 `orElseGet()` 方法就派上用场了。

`orElse()` 方法用于返回包裹在 Optional 对象中的值，如果该值不为 null，则返回；否则返回默认值。该方法的参数类型和值得类型一致。

```
String nullName = null;
String name = Optional.ofNullable(nullName).orElse("沉默王二");
System.out.println(name); // 输出：沉默王二
```

`orElseGet()` 方法与 `orElse()` 方法类似，但参数类型不同。如果 Optional 对象中的值为 null，则执行参数中的函数。

```
String nullName = null;
String name = Optional.ofNullable(nullName).orElseGet(()->"沉默王二");
System.out.println(name); // 输出：沉默王二
```

从输出结果以及代码的形式上来看，这两个方法极其相似，这不免引起我们的怀疑，Java 类库的设计者有必要这样做吗？

假设现在有这样一个获取默认值的方法，很传统的方式。

```
public static String getDefaultValue() {
    System.out.println("getDefaultValue");
    return "沉默王二";
}
```

然后，通过 `orElse()` 方法和 `orElseGet()` 方法分别调用 `getDefaultValue()` 方法返回默认值。

```
public static void main(String[] args) {
    String name = null;
    System.out.println("orElse");
    String name2 = Optional.ofNullable(name).orElse(getDefaultValue());

    System.out.println("orElseGet");
    String name3 = Optional.ofNullable(name).orElseGet(OrElseOptionalDemo::getDefaultValue);
}
```

注：`类名 :: 方法名`是 Java 8 引入的语法，方法名后面是没有 `()` 的，表明该方法并不一定会被调用。

输出结果如下所示：

```
orElse
getDefaultValue

orElseGet
getDefaultValue
```

输出结果是相似的，没什么太大的不同，这是在 Optional 对象的值为 null 的情况下。假如 Optional 对象的值不为 null 呢？

```
public static void main(String[] args) {
    String name = "沉默王三";
    System.out.println("orElse");
    String name2 = Optional.ofNullable(name).orElse(getDefaultValue());

    System.out.println("orElseGet");
    String name3 = Optional.ofNullable(name).orElseGet(OrElseOptionalDemo::getDefaultValue);
}
```

输出结果如下所示：

```
orElse
getDefaultValue
orElseGet
```

咦，`orElseGet()` 没有去调用 `getDefaultValue()`。哪个方法的性能更佳，你明白了吧？

### 6、获取值

直观从语义上来看，`get()` 方法才是最正宗的获取 Optional 对象值的方法，但很遗憾，该方法是有缺陷的，因为假如 Optional 对象的值为 null，该方法会抛出 NoSuchElementException 异常。这完全与我们使用 Optional 类的初衷相悖。

```
public class GetOptionalDemo {
    public static void main(String[] args) {
        String name = null;
        Optional<String> optOrNull = Optional.ofNullable(name);
        System.out.println(optOrNull.get());
    }
}
```

这段程序在运行时会抛出异常：

```
Exception in thread "main" java.util.NoSuchElementException: No value present
    at java.base/java.util.Optional.get(Optional.java:141)
    at com.cmower.dzone.optional.GetOptionalDemo.main(GetOptionalDemo.java:9)
```

尽管抛出的异常是 NoSuchElementException 而不是 NPE，但在我们看来，显然是在“五十步笑百步”。建议 `orElseGet()` 方法获取 Optional 对象的值。

### 7、过滤值

小王通过 Optional 类对之前的代码进行了升级，完成后又兴高采烈地跑去找老马要任务了。老马觉得这小伙子不错，头脑灵活，又干活积极，很值得培养，就又交给了小王一个新的任务：用户注册时对密码的长度进行检查。

小王拿到任务后，乐开了花，因为他刚要学习 Optional 类的 `filter()` 方法，这就派上了用场。

```
public class FilterOptionalDemo {
    public static void main(String[] args) {
        String password = "12345";
        Optional<String> opt = Optional.ofNullable(password);
        System.out.println(opt.filter(pwd -> pwd.length() > 6).isPresent());
    }
}
```

`filter()` 方法的参数类型为 Predicate（Java 8 新增的一个函数式接口），也就是说可以将一个 Lambda 表达式传递给该方法作为条件，如果表达式的结果为 false，则返回一个 EMPTY 的 Optional 对象，否则返回过滤后的 Optional 对象。

在上例中，由于 password 的长度为 5 ，所以程序输出的结果为 false。假设密码的长度要求在 6 到 10 位之间，那么还可以再追加一个条件。来看小王增加难度后的代码。

```
Predicate<String> len6 = pwd -> pwd.length() > 6;
Predicate<String> len10 = pwd -> pwd.length() < 10;

password = "1234567";
opt = Optional.ofNullable(password);
boolean result = opt.filter(len6.and(len10)).isPresent();
System.out.println(result);
```

这次程序输出的结果为 true，因为密码变成了 7 位，在 6 到 10 位之间。想象一下，假如小王使用 if-else 来完成这个任务，代码该有多冗长。

### 8、转换值

小王检查完了密码的长度，仍然觉得不够尽兴，觉得要对密码的强度也进行检查，比如说密码不能是“password”，这样的密码太弱了。于是他又开始研究起了 `map()` 方法，该方法可以按照一定的规则将原有 Optional 对象转换为一个新的 Optional 对象，原有的 Optional 对象不会更改。

先来看小王写的一个简单的例子：

```
public class OptionalMapDemo {
    public static void main(String[] args) {
        String name = "沉默王二";
        Optional<String> nameOptional = Optional.of(name);
        Optional<Integer> intOpt = nameOptional
                .map(String::length);

        System.out.println( intOpt.orElse(0));
    }
}
```

在上面这个例子中，`map()` 方法的参数 `String::length`，意味着要 将原有的字符串类型的 Optional 按照字符串长度重新生成一个新的 Optional 对象，类型为 Integer。

搞清楚了 `map()` 方法的基本用法后，小王决定把 `map()` 方法与 `filter()` 方法结合起来用，前者用于将密码转化为小写，后者用于判断长度以及是否是“password”。

```
public class OptionalMapFilterDemo {
    public static void main(String[] args) {
        String password = "password";
        Optional<String>  opt = Optional.ofNullable(password);

        Predicate<String> len6 = pwd -> pwd.length() > 6;
        Predicate<String> len10 = pwd -> pwd.length() < 10;
        Predicate<String> eq = pwd -> pwd.equals("password");

        boolean result = opt.map(String::toLowerCase).filter(len6.and(len10 ).and(eq)).isPresent();
        System.out.println(result);
    }
}
```

## 三、.LocalDateTime

自 `Java8`开始， `JDK`中其实就增加了一系列表示日期和时间的新类，最典型的就是 `LocalDateTime`。直言不讳，这玩意的出现就是为了干掉之前 `JDK`版本中的 `Date`老哥！

### 1、获取当前此刻的时间

```
LocalDateTime rightNow = LocalDateTime.now();
System.out.println( "当前时刻：" + rightNow );
System.out.println( "当前年份：" + rightNow.getYear() );
System.out.println( "当前月份：" + rightNow.getMonth() );
System.out.println( "当前日份：" + rightNow.getDayOfMonth() );
System.out.println( "当前时：" + rightNow.getHour() );
System.out.println( "当前分：" + rightNow.getMinute() );
System.out.println( "当前秒：" + rightNow.getSecond() );
// 输出结果：
当前时刻：2019-12-13T22:05:26.779
当前年份：2019
当前月份：DECEMBER
当前日份：13
当前时：22
当前分：5
当前秒：26
```

### 2、构造一个**指定年、月、日**的时间：

比如，想构造：`2019年10月12月12日9时21分32秒`

```
LocalDateTime beforeDate = LocalDateTime.of(2019, Month.DECEMBER, 12, 9, 21, 32);
```

### 3、修改日期

```
LocalDateTime rightNow = LocalDateTime.now(); 
rightNow = rightNow.minusYears( 2 );  // 减少 2 年
rightNow = rightNow.plusMonths( 3 );  // 增加 3 个月
rightNow = rightNow.withYear( 2008 ); // 直接修改年份到2008年
rightNow = rightNow.withHour( 13 );   // 直接修改小时到13时
```

### 4、格式化日期

```java
LocalDateTime rightNow = LocalDateTime.now();
String result1 = rightNow.format( DateTimeFormatter.ISO_DATE );
String result2 = rightNow.format( DateTimeFormatter.BASIC_ISO_DATE );
String result3 = rightNow.format( DateTimeFormatter.ofPattern("yyyy/MM/dd") );
System.out.println("格式化后的日期(基本样式一举例)：" + result1);
System.out.println("格式化后的日期(基本样式二举例)：" + result2);
System.out.println("格式化后的日期(自定义样式举例)：" + result3);
// 输出结果：
	格式化后的日期(基本样式一举例)：2019-12-13
    格式化后的日期(基本样式二举例)：20191213
    格式化后的日期(自定义样式举例)：2019/12/13
```

### 5、时间反解析

给你一个陌生的字符串，你可以按照你需要的格式把时间给反解出来

```
LocalDateTime time = LocalDateTime.parse("2002--01--02 11:21",DateTimeFormatter.ofPattern("yyyy--MM--dd HH:mm"));
System.out.println("字符串反解析后的时间为：" + time);
// 输出：字符串反解析后的时间为：2002-01-02T11:21
```

 ### 6、线程安全性问题！

其实上面讲来讲去只讲了在用法上的，这其实倒还好，并不致命，可是接下来要讨论的**线程安全性问题**才是致命的！

其实以前我们惯用的 `Date`时间类是可变类，这就意味着在多线程环境下对共享 `Date`变量进行操作时，必须**由程序员自己来保证线程安全**！否则极有可能翻车。

而自 `Java8`开始推出的 `LocalDateTime`却是线程安全的，开发人员不用再考虑并发问题，这点我们从 `LocalDateTime`的官方源码中即可看出：

不说别的，就光一句：

```
This class is immutable and thread-safe.  （不可变、线程安全！）
```

除了惯用 `Date`来表示时间之外，还有一个用于和 `Date`连用的 `SimpleDateFormat` 时间格式化类大家可能也戒不掉了!

`SimpleDateFormat`最主要的致命问题也是在于它本身**并不线程安全**





