---
title: 程序员随想
date: 2019-02-01 23:37:00
tags: 
---



## 程序员随想

***
 - 2016.5.30杂谈
    今天先记录几个关键词吧(Android学习方面)，也很迟了zzz: 多线程、反编译（dex2jar和jdgui）、okhttp、retrofit（对okhttp进行封装）、学习网站：CSDN、Github、appxcode、博客园（cnblogs）、极客学院、CSDN学院、Android官网、Segmentfault、cnbeta（资讯类）。

    <!--more-->
------

- 2016.5.31随想

  今天还是主要在学习之前下好安装好的IntelliJ IDEA的使用，上手也不算难，也试了一个web项目和一个用了SSH框架的web项目，很爽 \(≧▽≦)/和Eclipse、Myeclipse相比较只能用四个字形容感受：相见恨晚！╮(╯▽╰)╭ 1.8的JDK环境加Tomcat8的本地服务器，不知道是IDE优化了还是各种加成，重新部署项目很快,点一下Run汤姆猫6,7秒就Redeploy好了，这可真是比用Myeclipse+JDK1.6+Tomcat7快的多了。还有一个必须提的就是代码提示很强大，快键键也很多（因为和Android Studio是同平台产品的所以快捷键完全一样）。说说遇到的问题吧，最令我印象深刻的是，这Spring的applicationContext.xml里面配的数据库连接改了之后还是原来那个数据库，很怪，用IDEA自带的Database管理工具改变了连接的数据库参数好像也是没有什么变化，后台的Hibernate的DAO还是会到原来那个数据库那里去访问数据，甚至我把applicationContext.xml数据库的连接信息的		

```
	 <property name="url"value="jdbc:mysql://localhost:3306/xyb"></property>
```

去掉最后的数据库名字（xyb）变成 "jdbc:mysql://localhost:3306/" 这样居然也能对数据访问和操作，而且还是最初的那个数据库（xyb），这真是日了鬼了，确实是没怎么想明白——6月2日补充：是可以在hbm.xml中更改连接的数据库名字的，疏忽了

```
//Account.hbm.xml
<hibernate-mapping>
	<class name="com.po.Account" table="account" catalog="xyb">
	……
</hibernate-mapping>
```

，对了，IDEA里用SQL语句也有代码提示，，挺方便的吧。还有Android Studio里菜单栏下面有一排工具栏，怎么IDEA里一个都没了，嗯，原来是要手动显示菜单栏中的View->Toolbar，打上勾就出来了。
![默认不显示工具栏Toolbar](http://img.blog.csdn.net/20160601014020690)
![打开了Toolbar](http://img.blog.csdn.net/20160601014103362)
	然后今天还更熟悉了foreach遍历集合，发现（对于我算是发现了）当集合中没有元素时（不为NULL）用foreach去遍历并且在遍历中使用遍历的对象并不会报出异常，这就很明显了，当遇到不为NULL的空集合时foreach一次也不会循环，这真是奈斯。
	

------

- 2016.6.1记录

```
	IDEA关闭重复代码检测：Settings->搜索Duplicated code，把勾取消。
	 
SSH框架下的用JUnit和Spring-test进行单元测试（直接在单元测试类中操作和验证数据库中的数据）。参看文章《SSH框架下用JUnit+spring-test单元测试》。
今天还熟悉了Map用entrySet遍历的方法，并且将Map对象作为List集合的元素。entry就相当于Map中的一个实体。通过entry.getKey( )和entry.getValue( )方法可以很方便地得到这个实体的键和值。上代码了。
```

```
       List<Map<String,String>> list1=new ArrayList<Map<String,String>>();
        Map<String,String> map1=new HashMap<String, String>();
        map1.put("heihei", "heihei");
        map1.put("haha", "haha");
        Map<String,String> map2=new HashMap<String, String>();
        map2.put("lel", "lele");
        map2.put("nunu", "nunu");

        list1.add(map1);
        list1.add(map2);
        String ke;
        String va;
        for(Map.Entry entry2:map2.entrySet()){
            ke=entry2.getKey().toString();
            va=entry2.getValue().toString();
            System.out.println("key:"+ke+", "+"value:"+va);
        }
```

输出：
key:lel, value:lele
key:nunu, value:nunu

遍历以Map为集合元素的List：

```
        for(Map<String, String> map3:list1){
            Iterator iterator2=map3.entrySet().iterator();
            while(iterator2.hasNext()){
                Map.Entry entry=(Map.Entry)iterator2.next();
                ke=entry.getKey().toString();
                va=entry.getValue().toString();
                System.out.println("key:"+ke+", "+"value:"+va);
                System.out.println("");
            }
```

输出：
key:haha, value:haha

key:heihei, value:heihei

key:lel, value:lele

key:nunu, value:nunu

------

- 2016.6.4记录
  Java判断字符串内容是否为数字方法之一：利用正则表达式

```
String str="64738";
boolean result=str.matches("[0-9]+");//result为true或false
//matches方法判断是否匹配所给的正则表达式
```

判断是否为浮点数：

```
  public static boolean isDouble(String str) {    
    Pattern pattern = Pattern.compile("^[-\\+]?[.\\d]*$");    
    return pattern.matcher(str).matches();    
  }  
```

final关键字修饰**类**时该类**不能被继承**，
final关键字修饰**方法**时该方法**不能被其子类方法覆盖**(覆盖表示子类方法中参数类型、或参数个数、或参数顺序不同)
final关键字修饰**变量**时该变量的**值不能改变**，该变量不能再去指向别的对象，一旦初始化后无法改变。但其指向的对象的值可以改变，如：

```
static String b="222";
final static String a=b;
System.out.println(a);//输出"222"
b="3333";  
System.out.println(b); //输出"3333"
System.out.println(a);//还是输出"222"，因为前面已经初始化赋值了b的值"222"，值不再更改
```

------

- 2016.6.9随想
  Java中一个方法中的匿名内部类调用该方法中的局部变量时，该变量必须在方法中声明为final，如

```
        final TextView tvtopkymn=(TextView)popupView.findViewById(R.id.tvtopktmn);
        tvtopkymn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                tvtopkymn.setText("我是调用final变量");
            }
        });

```

这是为了克服局部变量和内部类之间生命周期的不一致性，内部类的生命周期比方法（等同于局部变量的生命周期）要长。设想一下：方法f被调用,从而在它的调用栈中生成了变量i,此时产生了一个局部内部类对象 inner_object,它访问了该局部变量i .当方法f()运行结束后,局部变量i就已死亡了,不存在了.但:局部内部类对象 inner_object还可能一直存在(只能没有人再引用该对象时,它才会死亡),它不会随着方法f()运行结束死亡.这时:出现了一个"荒唐"结果:局部内部类对象 inner_object要访问一个已不存在的局部变量i!	
	

------

- 2016.6.16记录
  java.lang.String中的**split方法**，实现方法为
   **`public String[] split(String regex)`**根据给定正则表达式的匹配拆分此字符串，将所有符合条件的都拆分完全，相当于下面一种的limit参数为0。
   或者
   **`public String[] split(String regex,int limit)`**根据匹配给定的正则表达式来拆分此字符串，limit参数限制目标数组中的元素个数，即该模式最多应用（limit-1）次就不再继续往下拆分；如果limit为0，则拆完为止，和不带这个limit参数结果一样。
   举个栗子

```
public class Test{
   public static void main(String args[]){
      String Str = new String("Welcome-to-cheetahlou-cheetah.com");

      for (String retval: Str.split("-", 2)){
         System.out.println(retval);
      }
      System.out.println("");

      for (String retval: Str.split("-", 3)){
         System.out.println(retval);
      }
      System.out.println("");

//limit参数为0
      for (String retval: Str.split("-", 0)){
         System.out.println(retval);
      }
      System.out.println("");
//无limit参数，其实就是limit参数为0
      for (String retval: Str.split("-")){
         System.out.println(retval);
      }
   }
}

```

输出结果将是：

Welcome
to-cheetahlou-cheetah.com

Welcome
to
cheetahlou-cheetah.com

Welcome
to
cheetahlou
cheetah.com

Welcome
to
cheetahlou
cheetah.com

另外，
1、如果用“.”作为分隔的话，必须是如下写法：`String.split("\\.")`,这样才能正确的分隔开，不能用`String.split(".")`;
2、如果用“|”作为分隔的话，必须是如下写法：`String.split("\\|")`,这样才能正确的分隔开，不能用`String.split("|")`;
“.”和“|”都是转义字符，必须得加"\\";
3、如果在一个字符串中有多个分隔符，可以用“|”作为连字符，比如：“a=1 and b =2 or c=3”,把三个都分隔出来，可以用`String.split("and|or")`; 再比如：`String str="Java string-split#test"`,可以用`Str.split(" |-|#")`把每个字符串分开。

------

- 2016.6.19记录
  CSS中margin和padding的区别用图表示为
   ![这里写图片描述](http://img.blog.csdn.net/20160619194914629)

------

 -2016.6.24感想
 没什么具体的代码方面的想说，只是想到今天考完了可能是大学最后一门考试，也是有些感慨，感慨里夹杂着一丝丝害怕，一丝丝莫名的害怕，也许是我的内心不够强大，还是会有对未知未来的不确定，有时候还真是会想未来啊你来的慢一点，But，the only thing we can do is KEEP GOING! 

------

- 2016.7.6记录
  用shiro框架验证登录：根据url过滤用户，设计用户组和权限，密码加密，记住密码登录

------

- 2016.8.6记录 
  对于 Android 深入的点会问一些自定义组件、性能优化，屏幕适配，IPC、Binder的理解，插件化，常用开源库以及原理等等。

------

- 2016.10.6记录
  回调方法，另见摘录的博文《Java中的回调（摘录）》。

------

- 2016.11.7记录
  jQuery是一个类数组对象，而DOM对象就是一个单独的DOM元素。

------

- 2017.1.2记录
  好的命名相当于给代码加了一段有用的注释

------

- 2017.2.13记录
  **jQuery的常用功能总结**
  - 在 DOM 树中进行查询
  - 修改 DOM 树及 DOM 相关操作
  - 事件处理
  - Ajax
  - Deferred 和 Promise
  - 对象和数组处理
  - 还有一个一直在用却很难在列清单时想到的——跨浏览器

------

- 2017.3.5记录
  **Java中String的拆箱装箱**
   先看个例子：

```
	String str1 ="abc";
	String str2 ="abc";
	System.out.println(str2==str1);  //输出为 true 
	System.out.println(str2.equals(str1));  //输出为 true 
            
	String str3 =new String("abc");
	String str4 =new String("abc"); 
	System.out.println(str3==str4);  //输出为 false 
	System.out.println(str3.equals(str4));  //输出为 true

```

再看个例子：

```
	String d ="2"; 
	String e ="23";
	e = e.substring(0, 1);
	System.out.println(e.equals(d)); //输出为 true 
	System.out.println(e==d); //输出为 false

```

第二个例子中，e的初始值与d并不同，因此e与d是各自创建了个对象，（e==d）为false 。
同理可知，第一个例子中的str3与str4也是各自new了个对象，而str1与str2却是引用了同一个对象。
**补充**：obj1==obj2 判断是obj1,obj2这两个引用变量是否相等，即它们所指向的对象是否为同一个对象。言外之意就是要求两个变量所指内存地址相等的时候，才能返回true，每个对象都有自己的一块内存，因此必须指向同一个对象才返回ture。

------

- 2017.3.10记录
  **Java跳出多重循环**
  在Java中，如何跳出当前的多重嵌套循环？
  答：在最外层循环前加一个标记如A，然后用break A;可以跳出多重循环。（Java中支持带标签的break和continue语句，作用有点类似于C和C++中的goto语句，但是就像要避免使用goto一样，应该避免使用带标签的break和continue，因为它不会让你的程序变得更优雅，很多时候甚至有相反的作用，所以这种语法其实不知道更好）

------

- 2017.3.19记录
  **类加载器工作机制**

  1.装载：将Java二进制代码导入jvm中，生成Class文件。

  2.连接：a）校验：检查载入Class文件数据的正确性 b）准备：给类的静态变量分配存储空间 c）解析：将符号引用转成直接引用

  3：初始化：对类的静态变量，静态方法和静态代码块执行初始化工作。

  双亲委派模型：类加载器收到类加载请求，首先将请求委派给父类加载器完成

  用户自定义加载器->应用程序加载器->扩展类加载器->启动类加载器。

------

2017.4.20记录
**Android Studio 项目push到GitHub报错**

```
Can't finish GitHub sharing process
		Successfully created project 'CareU' on GitHub, but initial push failed:
		unable to access 'https://github.com/cheetahlou/CareU.git/': error setting certificate verify locations:

```

找了很久没有发现原因，后来发现是CA证书的问题，电脑重装过一次Git没有重新安装过，CA证书没有导入，在Git安装目录中安装证书（有好几个，我装了其中的一两个，举例如下路径），然后再在Android Studiopush一次，会跳出windows证书提示你输入github的账户密码，success后就可以正常使用了。

```
	"G:\Android\Git\mingw64\ssl\certs\ca-bundle.crt"

```

------

- 2017.5.4一点记录
  **GroupId、ArtifactId**

  ```
  Maven项目的GroupId相当于公司和部门名，ArtifactId相当于项目名
  
  ```

------

- 2017.6.17记录
  **Socket连接建立流程**

1. 服务端实例化ServerSocket
2. 服务端执行accept()监听指定端口
3. 客户端实例化Socket尝试连接服务器（IP,port）
4. TCP三次握手成功，服务端的accept()返回Socket实例，同时客户端的Socket实例创建成功。

------

- 2017.6.20记录
  **不同变量默认值**
  · 对基本数据类型来说，对于类变量（static）和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值，而对于局部变量来说，在使用前必须显式地为其赋值，否则编译时不通过。
  · 对于同时被static和final修饰的常量，必须在声明的时候就为其显式地赋值，否则编译时不通过；而只被final修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值。
  · 对于引用数据类型reference来说，如数组引用、对象引用等，如果没有对其进行显式地赋值而直接使用，系统都会为其赋予默认的零值，即null。
  · 如果在数组初始化时没有对数组中的各元素赋值，那么其中的元素将根据对应的数据类型而被赋予默认的零值。

------

- 2017.6.30记录
  **Java取模运算**
  Java中取模运算%的结果的正负符号和被除数一致。
  如：
  	

  ```
  17 % 10 = 7；
  -17 % 10 = -7；
  -17 % (-10) = -7；
  
  ```

------

- 2017.8.4记录
  **count(1),count(*)和count(column)**
  <code>如果你的数据表没有主键，那么count(1)比count(*)快 
  如果有主键的话，那主键（联合主键）作为count的条件也比count(*)要快 
  如果你的表只有一个字段的话那count(*)就是最快的啦 
  count(*) count(1) 两者比较。主要还是要count(1)所相对应的数据字段。 
  如果count(1)是聚索引,id,那肯定是count(1)快。但是差的很小的。 
  因为count(*),自动会优化指定到那一个字段。所以没必要去count(1)，用count(*),sql会帮你完成优化的 
  因此：count(1)和count(*)基本没有差别！
  count(*)和count(column)比较时，和选取的column有关，越靠后越慢，column有索引时，如果索引可以为null则count(column)较快，如果索引要求不为null则两者一样快。
      综合来看，大多数场景可以大胆地使用count(*)。
      <code>

------

- 2017.8.7记录
  Mybatis中如果使用**ResultType**而不是**ResultMap**的话,以实体类为接收对象举例，默认实体类中的属性名要和数据库中的字段名相同才能得到数据，也可以和别名相同。如属性名是userName，则select t.user_name as userName,也能接收成功。

  SpringMVC中如果要接收多个参数可以用(HttpServletRequest request)获取，然后用request.getParameter("参数名")获取参数值。

  **@RestController注解**

> A convenience annotation that is itself annotated with @Controller and @ResponseBody. Types that carry this annotation are treated as controllers where @RequestMapping methods assume @ResponseBody semantics by default.  
> 注解本身使用@Controller和@ResponseBody注解。使用了这个注解的类会被看作一个controller-使用@RequestMapping的方法有一个默认的@ResponseBody注解。  
> @ResponseBody – As of version 4.0 this annotation can also be added on the type level in which case is inherited and does not need to be added on the method level.  
> @ResponseBody也可以加到类一级，通过继承方法一级不需要添加。 

------

- 2017.8.10记录
  **sql中用union all或in代替or**

应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行**全表扫描**。而改用union之后，性能就大大提高了。
使用"union all"的性能比"union"更高一些。因为当SQL 语句需要UNION两个查询结果集合时，这两个结果集合会以UNION-ALL的方式被合并， 然后在输出最终结果前进行排序。 如果用UNION ALL替代UNION， 这样排序就不是必要了，效率就会因此得到提高。

```sql
select * from Member where MemberName = '张三' 
union all
select * from Member where MemberName = '李四'
或
select * from Member where MemberName in ('张三','李四')

```

来代替or
~~select * from Member where MemberName = '张三' or MemberName = '李四'~~
相关链接：[避免全表扫描，提高sql查询效率](http://panyi5202.iteye.com/blog/612414)

- **MySQl的全表扫描情况**

```
 在以下几种条件下，MySQL就会做全表扫描： 
1)  数据表是在太小了，做一次全表扫描比做索引键的查找来得快多了。当表的记录总数小于10且记录长度比较短时通常这么做。 
2)  没有合适用于 ON 或 WHERE 分句的索引字段。 
3)  让索引字段和常量值比较，MySQL已经计算（基于索引树）到常量覆盖了数据表的很大部分，因此做全表扫描应该会来得更快。 
4)  通过其他字段使用了一个基数很小（很多记录匹配索引键值）的索引键。这种情况下，MySQL认为使用索引键需要大量查找，还不如全表扫描来得更快。

 对于小表来说，全表扫描通常更合适。但是对大表来说，尝试使用以下技术来避免让优化程序错误地选择全表扫描： 
 1)  执行 ANALYZE TABLE tbl_name 更新要扫描的表的索引键分布。 
 2)  使用 FORCE INDEX 告诉MySQL，做全表扫描的话会比利用给定的索引更浪费资源。 
 SELECT * FROM t1, t2 FORCE INDEX (index_for_column) 
 WHERE t1.col_name=t2.col_name; 
 3)  启动 mysqld 时使用参数 --max-seeks-for-key=1000 或者执行 SET max_seeks_for_key=1000 来告诉优化程序，所有的索引都不会导致超过1000次的索引搜索

```

------

- 2017.8.11记录
  **ajax跨域处理**

1.在response的头文件添加

```java
httpServletResponse.setHeader("Access-Control-Allow-Origin","*");

```

2.SpringMVC解决跨域问题的注解

```java
@CrossOrigin("http://test.com")
@CrossOrigin(origins="http://test.com",maxAge=3600)

```

 **Ajax请求SpringMVC返回到浏览器中文乱码**
 原因：Spring在处理`@ResponseBody`这个annotation的时候，org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter使用了org.springframework.http.converter.StringHttpMessageConverter进行处理，而StringHttpMessageConverter默认了字符编码为`ISO-8859-1`（如下）：

```java
 public static final Charset DEFAULT_CHARSET = Charset.forName("ISO-8859-1");

```

- 解决办法一
  使用SpringMVC提供的在@RequestMapping中加入produces注解

```
@RequestMapping(value = "/xxx", method = RequestMethod.GET,produces = "text/plain;charset=UTF-8")

```

- 解决办法二(自行验证)
  链接：
   [解决Spring MVC @ResponseBody返回中文字符串乱码问题](http://www.cnblogs.com/dyllove98/p/3180158.html)
   [Who sets response content-type in Spring MVC (@ResponseBody)](https://stackoverflow.com/questions/3616359/who-sets-response-content-type-in-spring-mvc-responsebody)

------

------

- 2017.9.23记录 **linux服务器能ping通ip不能ping通域名**
  做的电商项目调用物流网站的API时服务器一直请求不到，而本地电脑可以，还以为时json解析的问题，排查了好久发现是服务器解析域名的问题
   问题：`ping: unknown host www.kuaidi100.com`
   先查看
   ` # cat /etc/resolv.conf`
   修改DNS服务
   ` # vi /etc/resolv.conf`
   添加nameserver(公共DNS服务器)，如

> nameserver 8.8.8.8
> nameserver 223.5.5.5

------

- 2017.9.26记录 **Java中Date加一天,和Calendar转化**

```java
	 Calendar curCal= Calendar.getInstance();
     curCal.setTime(new Date());
     curCal.add(Calendar.DATE,-1);
     Date newDate=curCal.getTime();

```

------

转自[老D博客](https://laod.cn/design/page/javascript-texiao.html)：
一、添加鼠标点击特效：
添加鼠标点击特效，("富强", "民主", "文明", "和谐", "自由", "平等", "公正" ,"法治", "爱国", "敬业", "诚信", "友善")等。

```js
<script type="text/javascript">
/* 鼠标点击特效 */
var a_idx = 0;
jQuery(document).ready(function($) {
    $("body").click(function(e) {
var a = new Array("富强", "民主", "文明", "和谐", "自由", "平等", "公正" ,"法治", "爱国", "敬业", "诚信", "友善");
var $i = $("<span/>").text(a[a_idx]);
        a_idx = (a_idx + 1) % a.length;
var x = e.pageX,
        y = e.pageY;
        $i.css({
"z-index": 999999999999999999999999999999999999999999999999999999999999999999999,
"top": y - 20,
"left": x,
"position": "absolute",
"font-weight": "bold",
"color": "#ff6651"
        });
        $("body").append($i);
        $i.animate({
"top": y - 180,
"opacity": 0
        },
        1500,
function() {
            $i.remove();
        });
    });
});
</script>

```

效果如图：
![点击特效](http://img.blog.csdn.net/20171003111954981?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlZXRhaGxvdmVy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
二、添加评论框输入特效：
添加以下代码到主题尾部调用即可

```js
<script src="activate-power-mode.js"></script>
<script>
POWERMODE.colorful = true; // 控制开启/开启礼花特效  
POWERMODE.shake = false; // 控制开启/关闭屏幕震动特效  
document.body.addEventListener('input', POWERMODE);
</script>

```

在输入框打字时跟随光标弹出烟花很是炫酷（还可以开启屏幕震动哦！）
true 为开启
false 为关闭
效果如图：
![评论特效](http://img.blog.csdn.net/20171003112131947?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlZXRhaGxvdmVy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

------

------

- 2017.12.25 Redis 记录
  Redis keys通配符
   keys的模式支持下面通配符：
  \*通配符。该通配符表示所有的意思。如：keys * 匹配数据库中所有 key 。
  ?通配符。表示一个任意字符。如：keys h?llo 命令匹配 hello ， hallo 和 hxllo 等。
  *通配符。表示任何字符。如：keys h*llo 表示任何一h开头，llo结尾的key，如匹配 hllo 和 heeeeello 等。
  []通配符。这和正则表示中的[]类似，每次可以配其中任何一个字符。如：keys h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo，因为i不在[ae]中 。

参考Blog：[Redis操作总结](http://blog.csdn.net/LanSeTianKong12/article/details/50148937)

------

- 2018.1.5 集合操作`java.lang.UnsupportedOperationException`
  对于Collections.EMPTY_LIST或Arrays.asList(3, 5)等构建的固定大小的集合不能对其使用add,remove,addAll等方法改变大小，因其没有重写addAll方法，也不需要重写这些不合理（因其大小固定）的方法。同理可以推断以后遇到该异常的时候可以查看其是否支持，尤其是重写该方法了。

------

- 2018.1.24 **分布式和集群**
  分布式：一个业务分拆多个子业务，部署在不同的服务器上，集群：同一个业务，部署在多个服务器上
   集群用在解决高可用、负载均衡等，如redis集群，分布式将核心业务抽取出来单独作为独立服务，一个系统由多个子业务组成，如Dubbo和Git

------

- 2018.2.2  **快递100免费物流查询自用方法(返回物流信息列表json)**

```
    /**
     * 获取订单物流详情
     *
     * @return
     */
    @RequestMapping(value = "/consultLogstics", method = RequestMethod.GET)
    public JsonResp consultLogstics(Long orderId) {
        log.debug("获取订单物流详情");
        if (null == orderId)
            return JsonResp.toFail("订单id不能为空");
        Orders orders = ordersService.selectById(orderId);
        JSONObject jsonObject = new JSONObject();
        if (orders.getIsLogistics() == 0)
            return JsonResp.ok(jsonObject);
        String type = orders.getLogiCom();
        String postid = orders.getLogisticsNumber();
       /* String type="shentong";
        String postid="3333872084245";*/
        String consultUrl = "http://www.kuaidi100.com/query?type=" + type + "&postid=" + postid;
        String result = HttpClientUtil.sendGetSSLRequest(consultUrl, "UTF-8");
        Object obj = JSON.parse(result); 
        return JsonResp.ok(obj);
    }

```

------

- 2018.2.27  **网关Gateway相关摘录**
  网关实质上是一个网络通向其他网络的IP地址。用于两个高层协议不同的网络互连，网关的IP地址是具有路由功能的设备的IP地址，具有路由功能的设备有路由器、启用了路由协议的服务器（实质上相当于一台路由器）、代理服务器（也相当于一台路由器）

------

- 2018.3.15 **幂等性**
  *幂等性*：对同一个系统，使用同样的条件，一次请求和重复的多次请求对系统资源的影响是一致的；在数学中表示为f(x)=f(f(x))；常用于分布式系统设计中，在电商、互联网金融领域尤为重要。以此避免重复地调用接口造成数据被多次修改。
   refer via 简书：[幂等性浅谈](https://www.jianshu.com/p/475589f5cd7b)

------

- 2018.6.6 **SpringMVC中的重定向参数RedirectAttributes的addFlashAttribute()  **
  SpringMVC中的重定向参数RedirectAttributes的addFlashAttribute方法，是将参数的键值存到session中，而在分布式环境中时，如果没有做统一session管理，就会出现重定向以后接不到参数的情况。 

------

- 2018.6.12 **BigDecimal精度丢失**
  有小数点的值（浮点数）用BigDecimal构造函数初始化时，不论加减乘除，如果不添加双引号，会发生丢失精度的情况；
  比如`BigDecimal bd= new BigDecimal(2.48);`
  会变成`2.4799999999999999999999998223`
  所以，用字符串进行初始化`BigDecimal bd= new BigDecimal("2.48");` ，不会出现这个问题，还有记得除不尽时加 *scale* 和 *roundMode* 。

------

```java
response推json:
   /**
     * 将某个对象转换成json格式并发送到客户端
     * @param response
     * @param obj
     * @throws Exception
     */
    public static void sendJsonMessage(HttpServletResponse response, Object obj) throws Exception {
        response.setContentType("application/json; charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.print(JSONObject.toJSONString(obj, SerializerFeature.WriteMapNullValue,
                SerializerFeature.WriteDateUseDateFormat));
        writer.close();
        response.flushBuffer();
    }

```

------

- 2018.7.2 读取配置文件

```java
    public static String get(String key) {
        if (prop != null)
            return prop.getProperty(key);
        try {
            InputStream file = GlobEnvUtil.class.getClassLoader().getResourceAsStream("globEnv.properties");
            prop = new Properties();
            prop.load(file);
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return prop.getProperty(key);
    }

    static Properties prop = null;
    static {
        try {
            InputStream file = GlobEnvUtil.class.getClassLoader().getResourceAsStream("globEnv.properties");
            Properties prop = new Properties();
            prop.load(file);
        } catch (Exception e) {

            System.out.println(e.getMessage());
        }
    }

```

固定读取相应环境的"globalEnv.properties"文件，
*举个栗子*

> globalEnv.properties配置：
> `server.login.graphCode = https://ip:port/module/path`
> valocity的toolbox.xml配置:
>
> ```<?xml version="1.0" encoding="UTF-8" ?>
> <toolbox>
> <xhtml>true</xhtml>
> <tool>
> 	<key>env</key>
> 	<scope>application</scope>
> 	<class>com.xx.utils.velocity.GlobEnvUtil</class>
> </tool>
> 
> ```
>
> velocity读取：
> `var path = $env.getPath('server.login.graghCode');`

------

- 2018.7.4 **拆箱装箱和==、equals**
  包装类型的自动拆箱和基本类型的自动装箱方便了两者的互换和书写阅读。
  用==可以用于基本类型和包装类型之间的比较，只比较拆装后两者的值是否相等如

```java
       Integer zz = 213;
       System.out.println(zz==(short)213);//结果为true

```

Character、Byte、Short、Integer、Long等包装类型的equals方法，比较相等时先判断比较对象是否是该类型，类型不同直接返回false，类型相同再先强转为包装类，调用intValue()、shortValue()、longValue()等取到其基本类型的值，比较两者基本类型的值是否相等。

这里可以引申因为基本数据类型的存在而讨论Java是否是完全面向对象的语言？

*举栗：Integer类的equals方法*

```java
    /**
     * Compares this object to the specified object.  The result is
     * {@code true} if and only if the argument is not
     * {@code null} and is an {@code Integer} object that
     * contains the same {@code int} value as this object.
     *
     * @param   obj   the object to compare with.
     * @return  {@code true} if the objects are the same;
     *          {@code false} otherwise.
     */
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }

```

举栗：

```java
        Integer ez = 13;
        Integer ez1 = new Integer(127);
        Integer ez2 = new Integer(127);
        System.out.println(ez1==ez2);//false
        System.out.println(2.00f==(short)2);//true
        System.out.println(ez.equals((short)13));//false
        System.out.println("233".equals(233));//false

```

------

- 2018.7.16 **MySQL同一事务**
  MySQL同一事务中，后面的读数据会读到前面更新操作后的数据。
    比如原来count=1，更新操作commit后count应为0，那么同一个事务的后续查询会读到count=0，即使它还没有commit。

------

- 2018.7.17 **Linux日志查询**

1. 按关键字查找日志行号：`cat -n test.log | grep 关键字`
2. 找到行号后打印前后某几行日志：用tail和head 打印，`tail -n +行号 文件名 | head -n 20` 或者 用sed `sed -n '23882,24182p' test.log`
3. sed打印某段时间内的日志：`sed -n '/2018-08-01 14:52:00/,/2018-08-01 14:53:00/p' test.log`
4. less打印某关键词所在位置日志，可以按N和n分别查找前后一个关键词的日志，并且高亮该关键词，`less -N +?keyword  test.log` 或者`less -N +/keyword  test.log`，"?" 和"/"的区别是 按n和N不同方向匹配文本。用more不会高亮该关键词。

> less ?keyword　　向上查找
> n    向上匹配下一处匹配文本
> N    向下匹配下一处匹配文本 
>
> less /keyword　　向下查找
> n    向下匹配下一处匹配文本
> N    向上匹配下一处匹配文本



------

- 2018.7.19 **有赞权限系统实现关键点摘录**

```
SAM(Security Access Manager)权限系统模型的实现
在传统的 RBAC 模型中，通常通过一张关系表来保存角色与权限集的对应关系，实现权限与角色相关联。可以预见的是，随着零售业务的不断发展会积累下不计其数的功能点，导致关联表的数据极难维护和使用。SAM 权限系统利用进制转换的策略解决了这个问题 ，同时提高了存储效率以及权限判定效率。一个基本类型为 Long 的十进制数字，它也可以看做是由 64 位 0 或 1 组成的二进制。在 SAM 系统模型设计中，每一个功能点定义为一个权限点，该权限点由 idx 和 pos 两个属性确保是全局唯一的权限点。idx 表示第几个 Long 型空间，pos 表示 Long 型对应的二进制数中所处的位置，64 位长度即可代表 64 个不同能功能点。当 64 位满时无法再容放更多的功能点，这时 idx 属性会自增，重新申请一个 Long 型空间。如此一个 64 位的 Long 数字，通过 0 或 1 的组合，即可表示最多对 64 个不同的功能点所拥有权限的状态描述。
例如：权限集{1}表示拥有 idx=0, pos=0 对应功能点的权限，权限集{-1,1}表示拥有idx=0,pos∈[0,1,2,..,63]与 idx=1,pos=0 对应功能点的权限。
SAM 权限系统将资源与所代表的功能点的关联关系通过进制的方式管理起来，采用计算机进制的思想，抽象出功能集换算公式来完成资源与二进制之间的映射，以及角色与二进制的映射。
权

作者：有赞技术

```

------

- 2018.7.25 **MySQL比较两个时间字段大小，取较大者**
  两个时间字段中有可能有的为NULL，先取时间not null的，都有时间则取较大者。
   思路：用case when配合if语句将取值统一用时间戳比较大小，NULL作为时间戳的0。

```sql
select 
(CASE WHEN IF (time_field_A IS NULL,0,UNIX_TIMESTAMP(time_field_A)) > IF (time_field_B IS NULL,0,UNIX_TIMESTAMP(time_field_B)) THEN time_field_A ELSE time_field_B END) greater_time
from table

```

------

- 2018.7.26 **MySQL通过存储过程批量造数据**

```sql
/*
--  批量新增
*/
DELIMITER //
DROP PROCEDURE
IF EXISTS createTest ; CREATE PROCEDURE createTest ()
BEGIN
	DECLARE
		num INT ;
	SET num = 1 ;
	WHILE num < 1002 DO

	SET @DATE = '2018-07-26' ;
	SET @NAME = concat('名字', num) ; 
INSERT INTO tableName (field1, field2, field3)
	VALUES
		(@DATE, @NAME, '99',) ;
	SET num = num + 1 ;
	END
	WHILE ;
	END ; 
CALL createRemindTest () ;

```

------

- 2018.9.5 **Java List<T>和Set<T>检查重复和去重**

LIst和Set的contains(Object e)方法都用e的equals()方法检查集合中是否存在e对象。和hashCode方法，重写即可

Set的HashSet实现类的的add()方法和addAll()方法实际调用的HashMap的put再调putVal方法，将元素值作为HashMap的put方法的key作检查，可以看到是从hash值和equals方法判断是否已有重复元素，所以要判重或去重时按需重写对象的**equals()和hashCode()** 方法。。

**`HashSet中的add()方法`    **

```
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

```

**`HashMap.java中的put方法`**

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

```

------

- 2018.9.6 **集合的Fail-fast机制记录**

> 有两个线程（线程A，线程B），其中线程A负责遍历list、线程B修改list。线程A在遍历list过程的某个时候（此时expectedModCount = modCount=N），线程启动，同时线程B增加一个元素，这是modCount的值发生改变（modCount + 1 = N + 1）。线程A继续遍历执行next方法时，通告checkForComodification方法发现expectedModCount  = N  ，而modCount = N + 1，两者不等，这时就抛出ConcurrentModificationException 异常，从而产生fail-fast机制。

**volatile**关键字：

> Java提供了volatile关键字来保证可见性。
> <br>
> 当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。  
> <br>
> 而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

------

- 2018.9.10 **volatile关键字相关记录**

***参考:*** [并发编程中的volatile关键字](http://www.techug.com/post/java-volatile-keyword.html)
***相关资料:*** [全面理解Java内存模型(JMM)及volatile关键字](https://blog.csdn.net/javazejian/article/details/72772461)

**volatile特性：**

> 1.保证了不同线程对该变量操作的内存可见性;
> 2.禁止指令重排序

**volatile底层的实现机制**：

> " 如果把加入volatile关键字的代码和未加入volatile关键字的代码都生成汇编代码，会发现加入volatile关键字的代码会多出一个lock前缀指令。 lock前缀指令实际相当于一个内存屏障，内存屏障提供了以下功能： 1.重排序时不能把后面的指令重排序到内存屏障之前的位置 2.使得本CPU的Cache写入内存 ** **3.写入动作也会引起别的CPU或者别的内核无效化其Cache，相当于让新写入的值对别的线程可见 "
>
> > 

但是**volatile**关键字不能保证操作的 原子性 如下程序想得到结果为 `10000`
,实际结果极大概率为`10000` 不到一点，因变量`inc++` 操作实际分为

```java
public class Test {
    public volatile int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) {
        final Test test = new Test();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++)
                    test.increase();
            }).start();
        }

        while (Thread.activeCount() > 1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}

```

------

- 2018.9.14 **获取当周起讫时间**

```java
    Calendar calendar = Calendar.getInstance();
        calendar.set(Calendar.HOUR_OF_DAY, 0);
        calendar.set(Calendar.MINUTE, 0);
        calendar.set(Calendar.SECOND, 0);
        calendar.set(Calendar.MILLISECOND, 0);
        int dayOfWeek = calendar.get(Calendar.DAY_OF_WEEK);
        //周日——周六 有不同的代表value 1-7
        if (dayOfWeek == 1) {
            dayOfWeek += 7;
        }
        int addOrSub = 2-dayOfWeek;
        //获取当周周一00:00和周日23:59
        Date weekStart = DateUtils.addOrSubDay(calendar.getTime(),addOrSub);
        Date weekEnd = DateUtils.addOrSubDay(weekStart, 6);
        weekEnd = DateUtils.getFinalTimeOfDay(weekEnd);

```

------

- 2018.10.26 **linux获取进程pid，杀进程命令**

```shell
P_ID=`ps -ef | grep -w "$SERVICE_NAME" | grep -v "grep" | awk '{print $2}'`

		if [ "$P_ID" == "" ]; then
            echo "=== $SERVICE_NAME process not exists or stop success"
        else
            echo "=== $SERVICE_NAME process pid is:$P_ID"
            echo "=== begin kill $SERVICE_NAME process, pid is:$P_ID"
            kill -9 $P_ID
        fi

```

重启脚本 服务.sh：

```shell
#!/bin/sh

## java env
export JAVA_HOME=/usr/java/jdk1.8.0_131
export JRE_HOME=$JAVA_HOME/jre


SERVICE_DIR=/usr/local/目录
SERVICE_NAME=服务名
JAR_NAME=$SERVICE_NAME\.jar
PID=$SERVICE_NAME\.pid

cd $SERVICE_DIR

case "$1" in

    start)
        echo "$JRE_HOME/bin/java -Xms128m -Xmx512m -Dfile.encoding=UTF-8 -jar $SERVICE_DIR/$JAR_NAME"
        echo $! > $SERVICE_DIR/$PID
        echo "=== start $SERVICE_NAME"
#!/bin/sh

## java env
export JAVA_HOME=/usr/java/jdk1.8.0_131
export JRE_HOME=$JAVA_HOME/jre


SERVICE_DIR=/usr/local/目录
SERVICE_NAME=服务名
JAR_NAME=$SERVICE_NAME\.jar
PID=$SERVICE_NAME\.pid

cd $SERVICE_DIR

case "$1" in

    start)
        nohup $JRE_HOME/bin/java -server -Xms256m -Xmx2048m -XX:+UseG1GC -XX:ParallelGCThreads=4 -XX:SurvivorRatio=8 -XX:+DisableExplicitGC -XX:HeapDumpPath=/usr/java/activity_heap.dump -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8 -jar $SERVICE_DIR/$JAR_NAME >$SERVICE_DIR/logs/服务名.out 2>&1 &
        echo "$JRE_HOME/bin/java -Xms128m -Xmx512m -Dfile.encoding=UTF-8 -jar $SERVICE_DIR/$JAR_NAME"
        echo $! > $SERVICE_DIR/$PID
        echo "=== start $SERVICE_NAME"
        ;;

    stop)
        kill `cat $SERVICE_DIR/$PID`
        rm -rf $SERVICE_DIR/$PID
        echo "=== stop $SERVICE_NAME"

        sleep 5
        P_ID=`ps -ef | grep -w "$SERVICE_NAME" | grep -v "grep" | awk '{print $2}'`
        if [ "$P_ID" == "" ]; then
            echo "=== $SERVICE_NAME process not exists or stop success"
        else
            echo "=== $SERVICE_NAME process pid is:$P_ID"
            echo "=== begin kill $SERVICE_NAME process, pid is:$P_ID"
            kill -9 $P_ID
        fi
        ;;

    restart)
        $0 stop
        sleep 2
        $0 start
        echo "=== restart $SERVICE_NAME"
        ;;

    *)
        ## restart

```

------

- 2018.11.20 **Linux sz rz命令**

> <font color=red>**sz**</font> 命令（Send ZMODEM）通过ZMODEM协议，可将多个文件从远程服务器下载到本地。注意不能下载文件夹，如果下载文件夹，请先打包再下载

比如下载多个文件

```shell
 sz  file1 file2 file3

```

> <font color = red>**rz**</font> 命令（Receive ZMODEM），使用ZMODEM协议，将本地文件批量上传到远程Linux/Unix服务器，注意不能上传文件夹。

比如在XShell或其他虚拟终端软件中敲rz和回车可以打开系统资源管理器选择本地文件上传。

------

- 2018.12.12 **IDEA refactor remark**
  如下图，IDEA用refactor重构方法名的时候默认会把字符串、注释等中的部分也包括进去，这样当字符串中的某一部分和方法名相同时就会被一起refactor掉。太智能惹的锅？
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181212140845648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZWV0YWhsb3Zlcg==,size_16,color_FFFFFF,t_70)

------

------

- 2018.12.13 **Linux开放端口**

```shell
 1.  查看已打开的端口  # netstat -anp
 2. 查看想开的端口是否已开 # firewall-cmd --query-port=666/tcp
  若此提示 FirewallD is not running 
  表示为不可知的防火墙 需要查看状态并开启防火墙

 3. 查看防火墙状态  # systemctl status firewalld
 running 状态即防火墙已经开启
 dead 状态即防火墙未开启
 4. 开启防火墙，# systemctl start firewalld  没有任何提示即开启成功
 5. 开启防火墙 # service firewalld start  
   关闭防火墙 # systemctl stop firewalld
   centos7.3 上述方式可能无法开启，可以先#systemctl unmask firewalld.service 然后 # systemctl start firewalld.service

 6. 查看想开的端口是否已开 # firewall-cmd --query-port=666/tcp    提示no表示未开
 7. 开永久端口号 firewall-cmd --add-port=666/tcp --permanent   提示    success 表示成功
 8. 重新载入配置  # firewall-cmd --reload    比如添加规则之后，需要执行此命令
 9. 再次查看想开的端口是否已开  # firewall-cmd --query-port=666/tcp  提示yes表示成功
 10. 若移除端口 # firewall-cmd --permanent --remove-port=666/tcp

 11. 修改iptables  有些版本需要安装iptables-services # yum install iptables-services 然后修改进目录 /etc/sysconfig/iptables   修改内容

```

------

- 2019.1.10 **Js中的对象类型精确判断**

  在js 中 , 有一种判断数据基本类型的方法 typeof , 只能判断5中基本类型:即 “number”，”string”，”undefined”，”boolean”，”object” 五种。
  可见: typeof 会根据对象类型返回对应的类型字符串, 但是有几个缺点:

  对于数组、函数、对象来说，其关系错综复杂，使用 typeof 都会统一返回 “object” 字符串,null也会返回’object’,对NaN返回是’number’ 
  Object.prototype.toString.call(obj) == "[object String]" ?

```js
console.log(Object.prototype.toString.call("jerry"));//[object String]
console.log(Object.prototype.toString.call(12));//[object Number]
console.log(Object.prototype.toString.call(true));//[object Boolean]
console.log(Object.prototype.toString.call(undefined));//[object Undefined]
console.log(Object.prototype.toString.call(null));//[object Null]
console.log(Object.prototype.toString.call({name: "jerry"}));//[object Object]
console.log(Object.prototype.toString.call(function(){}));//[object Function]
console.log(Object.prototype.toString.call([]));//[object Array]
console.log(Object.prototype.toString.call(new Date));//[object Date]
console.log(Object.prototype.toString.call(/\d/));//[object RegExp]
function Person(){};
console.log(Object.prototype.toString.call(new Person));//[object Object]
```

------

- 2019.01.15  **泛型方法**
  定义泛型方法，其格式是：修饰符 <类型参数列表> 返回类型 方法名(形参列表) { 方法体 }。例如：
   `public static <T, S> int func(List<T> list, Map<Integer, S> map) { ... }`
   其中T和S是泛型类型参数。

------

- 2019.01.31 **设计原则Remark 《Head First设计模式》**
  1. 找出应用中可能需要变化之处, 把它们独立出来. (设计系统时可以预先考虑独立出来)
  2. 针对接口编程, 而不是针对实现编程(如父类引用指向子类实例Animal a = new Dog())
  3. 多用组合，少用继承

***

- 2019.02.13 **反向代理 Reverse Proxy**

  **正向代理:** (前向代理)表示对客户端请求的代理, 隐藏了客户端的真实地址 一定程度上保护了客户端的隐私。作为客户端的代理，将从互联网上服务器上获取资源, 再打包返回给一个或多个的客户端，服务端（如Web服务器）只知道代理的IP地址而不知道客户端的IP地址。如常见的科学上网工具就是客户端代理。

  **反向代理:** 则相对于正向代理而言, 而反向代理是作为服务器端（如Web服务器）的代理使用,隐藏真实服务端，而不是客户端。简单地说就是服务器端的一个负载均衡器, 单台服务器的处理请求能力有限, 可以配置多个服务器进行负载均衡, **也意味着用户做服务器域名解析时，解析得到的IP其实是负载均衡的IP，而不是服务器的IP**

**反向代理**代理的对象从客户端换成了服务器, 收发服务器的响应, 隐藏真实服务器的IP地址。![](https://i.imgur.com/0e01FRp.jpg)



<div style="background-color:#ddd">
	由此可见, 正向代理和反向代理的概念并不冲突, 两者可以同时存在 分别服务于客户端和服务器。(BTW, 个人不是很感冒将这样的形式称为正向和反向代理, 这个名词造的真的是不太形象, 也许是为了区别原来客户端代理用了单词"*Proxy*", 命名的人用了"*Reverse Proxy*"这样随意一点词不达意的命名, 导致我们现在很难从字面上一眼看出这大概是个什么概念, 个人觉得也许用"*Server Proxy*"这样的命名会不会更好一些呢) </div>



#### 功能(via Wikipedia)

反向代理的主要作用为：

- 对客户端隐藏服务器（簇）的IP地址
- 安全：作为[应用层防火墙](https://zh.wikipedia.org/wiki/%E6%87%89%E7%94%A8%E5%B1%A4%E9%98%B2%E7%81%AB%E7%89%86)，为网站提供对基于Web的攻击行为（例如[DoS](https://zh.wikipedia.org/wiki/DoS)/[DDoS](https://zh.wikipedia.org/wiki/DDoS)）的防护，更容易排查[恶意软件](https://zh.wikipedia.org/wiki/%E6%83%A1%E6%84%8F%E8%BB%9F%E9%AB%94)等
- 为后端服务器（簇）统一提供加密和[SSL](https://zh.wikipedia.org/wiki/SSL)加速（如SSL终端代理）
- [负载均衡](https://zh.wikipedia.org/wiki/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)，若服务器簇中有负荷较高者，反向代理通过[URL重写](https://zh.wikipedia.org/wiki/URL%E9%87%8D%E5%AF%AB)，根据连线请求从负荷较低者获取与所需相同的资源或备援
- 对于静态内容及短时间内有大量访问请求的动态内容提供[缓存服务](https://zh.wikipedia.org/wiki/Web%E7%BC%93%E5%AD%98)
- 对一些内容进行[压缩](https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%A3%93%E7%B8%AE)，以节约[带宽](https://zh.wikipedia.org/wiki/%E9%A0%BB%E5%AF%AC)或为网络带宽不佳的网络提供服务
- 减速上传
- 为在私有网络下（如[局域网](https://zh.wikipedia.org/wiki/%E5%8D%80%E5%9F%9F%E7%B6%B2%E8%B7%AF)）的服务器簇提供[NAT穿透](https://zh.wikipedia.org/wiki/NAT%E7%A9%BF%E9%80%8F)及外网发布服务
- 提供HTTP访问认证[[2\]](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86#cite_note-2)
- 突破[互联网封锁](https://zh.wikipedia.org/wiki/%E9%98%B2%E7%81%AB%E9%95%BF%E5%9F%8E)（不常用，因为反向代理与客户端之间的连线不一定是加密连线，非加密连线仍有遭内容审查进而遭封禁的风险；此外面对针对[域名](https://zh.wikipedia.org/wiki/%E7%B6%B2%E5%9F%9F%E5%90%8D%E7%A8%B1)的[关键字过滤](https://zh.wikipedia.org/wiki/%E9%97%9C%E9%8D%B5%E5%AD%97%E9%81%8E%E6%BF%BE)、[DNS缓存污染/投毒攻击](https://zh.wikipedia.org/wiki/DNS%E6%8A%95%E6%AF%92%E6%B1%A1%E6%9F%93)乃至[深度数据包检测](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E5%B0%81%E5%8C%85%E6%AA%A2%E6%B8%AC)也无能为力）

***