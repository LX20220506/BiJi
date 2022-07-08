# Spring_Ioc的入门案例（xml）

> Spring的核心由：beans、context、core、expression 四部分组成

## 1.配置ioc的环境

需要的jar包

- commons-logging-1.1.1.jar
- spring-beans-5.3.21.jar
- spring-context-5.3.21.jar
- spring-core-5.3.21.jar
- spring-expression-5.3.21.jar

## 2.配置xml文件

1. 创建一个spring的xml配置文件

2. 在配置文件中的`beans`标签中添加如下代码

   ~~~xml
   <!-- 对Calculation类进行配置；通过id找到对应的类-->
   <bean id="Calculation" class="demo.Calculation" />
   ~~~

完整示例：

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 对Calculation类进行配置；通过id找到对应的类-->
    <bean id="Calculation" class="demo.Calculation" />
</beans>
~~~

## 3.测试

1. 随便创建一个类 

   1. ~~~java
      /**
       * 创建一个计算类，用于测试
       */
      public class Calculation {
      
          /**
           * 创建一个加法的方法
           */
          public void Add(){
              System.out.println("Add.........");
          }
      }
      ~~~

2. 测试ioc

   1. ~~~java
      public class IocTest {
      
          /**
           * 测试依赖注入的流程
           */
          @Test
          public void Test(){
              // 通过读取类路径下的 XML 格式的配置文件；创建 IOC 容器对象（最好是提到外面去，定义为私有的全局变量）
              ApplicationContext application= new ClassPathXmlApplicationContext("application.xml");
      
              // 通过读取配置文件中id为Calculation的标签，拿到他的class属性，
              // 通过反射创建object类型，再使用类型强转，转换为需要的类型
              Calculation calculation =(Calculation) application.getBean("Calculation");
      
              // 调用类中的方法
              calculation.Add();
      
          }
      }
      ~~~



# IOC容器

## 一、IOC容器

###  1、什么是IOC（控制反转）

1.  把对象创建和对象之间的调用过程，交给Spring进行管理

2.  使用IOC目的：为了降低耦合度


###  2、IOC底层

-  **xml解析、工厂模式、反射**
  - 1.读取xml文件配置文件，拿到class的名称
  - 2.有了类的名称，可以通过反射获取类型，并创建对象
  - 3.返回对象
  
  ![](./img/6.png)

###  3、Spring提供的IOC容器实现的两种方式（两个接口）

 a）BeanFactory接口：IOC容器基本实现是Spring内部接口的使用接口，不提供给开发人员进行使用（加载配置文件时候不会创建对象，在获取对象时才会创建对象。）

 b）ApplicationContext接口：BeanFactory接口的子接口，提供更多更强大的功能，提供给开发人员使用（加载配置文件时候就会把在配置文件对象进行创建）推荐使用！

###  4、ApplicationContext接口的实现类

| 类型名                          | 简介                                                         |
| ------------------------------- | :----------------------------------------------------------- |
| ClassPathXmlApplicationContext  | 通过读取类路径下的 XML 格式的配置文件创建 IOC 容器对象       |
|                                 |                                                              |
| FileSystemXmlApplicationContext | 通过文件系统路径读取 XML 格式的配置文件创建 IOC 容器对象     |
|                                 |                                                              |
| ConfigurableApplicationContext  | ApplicationContext 的子接口，包含一些扩展方法 refresh() 和 close() ，让 ApplicationContext 具有启动、关闭和刷新上下文的能力。 |
|                                 |                                                              |
| WebApplicationContext           | 专门为 Web 应用准备，基于 Web 环境创建 IOC 容器对象，并将对象引入存入 ServletContext 域中。 |



## 二、IOC容器-Bean管理（xml）

> 注意：配置xml时，它默认使用的是无参构造函数，必须配置构造函数需要的参数；若没有无参构造函数，会报错

### IOC操作Bean管理

​		 a）Bean管理就是两个操作：

​				（1）Spring创建对象；

​				（2）Spring注入属性

###  基于XML配置文件创建对象

~~~xml
<!--1 配置User对象创建-->
<bean id="user" class="com.atguigu.spring5.User"></bean>

~~~

###  基于XML方式注入属性（DI：依赖注入（注入属性））

####  a）set方式注入

> 使用属性注入时，必须要对注入的属性创建set方法；否则会报错

~~~java
//（1）传统方式： 创建类，定义属性和对应的set方法
public class Book {
        //创建属性
        private String bname;

        //创建属性对应的set方法
        public void setBname(String bname) {
            this.bname = bname;
        }

   }
~~~

~~~xml
<!--（2）spring方式： set方法注入属性-->
<bean id="book" class="com.atguigu.spring5.Book">
    <!--使用property完成属性注入
        name：类里面属性名称
        value：向属性注入的值
    -->
    <property name="bname" value="Hello"></property>
    <property name="bauthor" value="World"></property>
</bean>
~~~

####  b）有参构造函数注入

~~~java
//（1）传统方式：创建类，构建有参函数
public class Orders {
    //属性
    private String oname;
    private String address;
    //有参数构造
    public Orders(String oname,String address) {
        this.oname = oname;
        this.address = address;
    }
  }
~~~

~~~xml
<!--（2）spring方式：有参数构造注入属性-->
<bean id="orders" class="com.atguigu.spring5.Orders">
    <!--   constructor-arg：在构造参数中，为属性赋值
                name：表示构造函数中，传入的属性名称
                value：表示要设置的值
                -->
    <constructor-arg name="oname" value="Hello"></constructor-arg>
    <constructor-arg name="address" value="China！"></constructor-arg>
</bean>
~~~

####  c）p名称空间注入（了解即可）

~~~xml
<!--1、添加p名称空间在配置文件头部-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"		<!--在这里添加一行p-->

	<!--2、在bean标签进行属性注入（算是set方式注入的简化操作）-->
    <bean id="book" class="com.atguigu.spring5.Book" p:bname="very" p:bauthor="good">
</bean>
~~~

###  注入空值和特殊符号

~~~xml
<bean id="book" class="com.atguigu.spring5.Book">
    <!--（1）null值-->
    <property name="address">
        <null/><!--属性里边添加一个null标签-->
    </property>
    

    <!--（2）特殊符号赋值-->
     <!--属性值包含特殊符号
       a 把<>进行转义 &lt; &gt;
       b 把带特殊符号内容写到CDATA
      -->
        <property name="address">
            <value><![CDATA[<<南京>>]]></value>
        </property>
</bean>
~~~

### 注入属性-外部bean

####  a）创建两个类service和dao类

~~~java
public class UserService {//service类

    //创建UserDao类型属性，生成set方法
    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public void add() {
        System.out.println("service add...............");
        userDao.update();//调用dao方法
    }

}

public class UserDaoImpl implements UserDao {//dao类

    @Override
    public void update() {
        System.out.println("dao update...........");
    }

}
~~~

####  b）在spring配置文件中进行配置

~~~xml
<!--1 service和dao对象创建-->
<bean id="userService" class="com.atguigu.spring5.service.UserService">
    <!--注入userDao对象
        name属性：类里面属性名称
        ref属性：创建userDao对象bean标签id值
    -->
    <property name="userDao" ref="userDaoImpl"></property>
</bean>
<bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>
~~~

###  基于XML方式注入内部bean和级联赋值

####  a）注入属性-内部bean

（1）一对多关系：部门和员工
	一个部门有多个员工，一个员工属于一个部门（部门是一，员工是多）
（2）在实体类之间表示一对多关系，员工表示所属部门，使用对象类型属性进行表示

~~~java
//部门类
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
}

//员工类
public class Emp {
    private String ename;
    private String gender;
    //员工属于某一个部门，使用对象形式表示
    private Dept dept;
    

    public void setDept(Dept dept) {
        this.dept = dept;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }

}
~~~



（3）在spring配置文件中配置

~~~xml
<!--内部bean-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="Andy"></property>
        <property name="gender" value="女"></property>
        <!--设置对象类型属性-->
        <property name="dept">
            <bean id="dept" class="com.atguigu.spring5.bean.Dept"><!--内部bean赋值-->
                <property name="dname" value="宣传部门"></property>
            </bean>
        </property>
    </bean>
~~~



####  b）注入属性-级联赋值

~~~xml
<!--方式一：级联赋值-->
<bean id="emp" class="com.atguigu.spring5.bean.Emp">
    <!--设置两个普通属性-->
    <property name="ename" value="Andy"></property>
    <property name="gender" value="女"></property>
    <!--级联赋值-->
    <property name="dept" ref="dept"></property>
</bean>
<bean id="dept" class="com.atguigu.spring5.bean.Dept">
    <property name="dname" value="公关部门"></property>
</bean>
~~~


~~~java
 //方式二：生成dept的get方法（get方法必须有！！）
    public Dept getDept() {
        return dept;
    }
~~~

~~~xml
<!--级联赋值-->
<bean id="emp" class="com.atguigu.spring5.bean.Emp">
    <!--设置两个普通属性-->
    <property name="ename" value="jams"></property>
    <property name="gender" value="男"></property>
    <!--级联赋值-->
    <property name="dept" ref="dept"></property>
    <property name="dept.dname" value="技术部门"></property>
</bean>
<bean id="dept" class="com.atguigu.spring5.bean.Dept">
</bean>
~~~

### 对集合属性进行注入

#### a）集合的注入

​		学生类

~~~java

public class Stu {
    private String[] cname;
    private List list;
    private Map map;
    private Set set;

    public void setCname(String[] cname) {
        this.cname = cname;
    }

    public void setList(List list) {
        this.list = list;
    }

    public void setMap(Map map) {
        this.map = map;
    }

    public void setSet(Set set) {
        this.set = set;
    }

    @Override
    public String toString() {
        return "Stu{" +
                "cname=" + Arrays.toString(cname) +
                ", list=" + list +
                ", map=" + map +
                ", set=" + set +
                '}';
    }
}
~~~

​		xml配置

~~~xml
<!--集合属性的注入-->
    <bean id="stu" class="com.entity.Stu" >
        <!--数组注入-->
        <property name="cname">
            <array>
                <value>java</value>
                <value>python</value>
            </array>
        </property>

        <!--集合注入-->
        <property name="list">
            <list>
                <value>list1</value>
                <value>list2</value>
            </list>
        </property>

        <!--map注入-->
        <property name="map">
            <map>
                <entry key="map_key1" value="map_value1" />
                <entry key="map_key2" value="map_value2" />
            </map>
        </property>

        <!--set注入-->
        <property name="set">
            <set>
                <value>set1</value>
                <value>set2</value>
            </set>
        </property>
    </bean>
~~~

​		测试

~~~java
	/**
     * 对集合属性赋值：赋值类型为String
     */
    @Test
    public void TestStu1(){
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("config1.xml");
        Stu stu = (Stu) applicationContext.getBean("stu");

        System.out.println(stu.toString());
    }
~~~

#### b）注入泛型集合

​		学生类

~~~java
public class Stu {
    
    private List<Course> courseList;

    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }

    public List<Course> getCourseList() {
        return courseList;
    }
}

~~~

​		xml配置

~~~xml
<bean id="stu" class="com.entity.Stu">
    <property name="courseList">
        <list>
            <ref bean="cou1" />
            <ref bean="cou2"/>
        </list>
    </property>
</bean>

<bean id="cou1" class="com.entity.Course">
    <property name="courseName" value="java"></property>
</bean>
<bean id="cou2" class="com.entity.Course">
    <property name="courseName" value="c#"></property>
</bean>
~~~

​		测试

~~~java
	/**
     * 对集合属性赋值：赋值类型为引用类型
     */
    @Test
    public void TestStu2(){
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("config2.xml");
        Stu stu = (Stu) applicationContext.getBean("stu");
        stu.getCourseList().forEach(System.out::println);
    }
~~~

### FactoryBean

1.  Spring 有两种类型 bean，一种普通 bean，另外一种工厂 bean（FactoryBean）

2.  普通 bean：在配置文件中定义 bean 类型就是返回类型

3.  工厂 bean：在配置文件定义 bean 类型<font color=blue size=4><b>可以和返回类型不一样</b> </font>第一步 创建类，让这个类作为工厂 bean，实现接口 FactoryBean 第二步 实现接口里面的方法，在实现的方法中定义返回的 bean 类型

~~~java
public class MyBean implements FactoryBean<Course> {

    //定义返回bean
    @Override
    public Course getObject() throws Exception {
        Course course = new Course();
        course.setCname("abc");
        return course;
    }

}
~~~

~~~xml
<bean id="myBean" class="com.atguigu.spring5.factorybean.MyBean">
</bean>
~~~


~~~java
@Test
public void test3() {
 ApplicationContext context =
 new ClassPathXmlApplicationContext("bean3.xml");
 Course course = context.getBean("myBean", Course.class);//返回值类型可以不是定义的bean类型！
 System.out.println(course);
}
~~~

###  bean 作用域

 在 Spring 里面，默认情况下，bean 是单实例对象，下面进行作用域设置：

> （1）在 spring 配置文件 bean 标签里面有属性（scope）用于设置单实例还是多实例
>
> （2）scope 属性值 第一个值 默认值，singleton，表示是单实例对象 第二个值 prototype，表示是多实例对象
>

~~~xml
<bean id="book" class="com.atguigu.spring5.collectiontype.Book" scope="prototype"><!--设置为多实例-->
        <property name="list" ref="bookList"></property>
</bean>
~~~

> （3）singleton 和 prototype 区别
>
> -  a）singleton 单实例，prototype 多实例
>
> -  b）设置 scope 值是 singleton 时候，<font color=blue><b>加载 spring 配置文件时候就会创建单实例对象</b> </font>；设置 scope 值是 prototype 时候，<font color=blue><b>不是在加载 spring 配置文件时候创建对象，在调用 getBean 方法时候创建多实例对象</b></font>
>

###  bean 生命周期

> 1、生命周期 ：从对象创建到对象销毁的过程
>
> 2、bean 生命周期
>
>  （1）通过构造器创建 bean 实例（无参数构造）
>
>  （2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）
>
>  （3）调用 bean 的初始化的方法（需要进行配置初始化的方法）
>
>  （4）bean 可以使用了（对象获取到了）
>
>  （5）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）
>
> 3、演示 bean 生命周期 ：

```java
    public class Orders {
     //无参数构造
     public Orders() {
     System.out.println("第一步 执行无参数构造创建 bean 实例");
     }
     private String oname;
     public void setOname(String oname) {
     this.oname = oname;
     System.out.println("第二步 调用 set 方法设置属性值");
     }
     //创建执行的初始化的方法
     public void initMethod() {
     System.out.println("第三步 执行初始化的方法");
     }
     //创建执行的销毁的方法
     public void destroyMethod() {
     System.out.println("第五步 执行销毁的方法");
     }
    }
```

~~~java
public class MyBeanPost implements BeanPostProcessor {//创建后置处理器实现类
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}
~~~


~~~xml
<!--配置文件的bean参数配置-->
<bean id="orders" class="com.atguigu.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">	<!--配置初始化方法和销毁方法-->
    <property name="oname" value="手机"></property><!--这里就是通过set方式（注入属性）赋值-->
</bean>

<!--配置后置处理器-->
<bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
~~~


 ~~~java
 @Test
  public void testBean3() {
 // ApplicationContext context =
 // new ClassPathXmlApplicationContext("bean4.xml");
  ClassPathXmlApplicationContext context =
  new ClassPathXmlApplicationContext("bean4.xml");
  Orders orders = context.getBean("orders", Orders.class);
  System.out.println("第四步 获取创建 bean 实例对象");
  System.out.println(orders);
  //手动让 bean 实例销毁
  context.close();
  }
 ~~~

4、bean 的后置处理器，bean 生命周期有七步 （正常生命周期为五步，而配置后置处理器后为七步）

> -  （1）通过构造器创建 bean 实例（无参数构造）
>
> -  （2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）
>
> -  （3）把 bean 实例传递 bean 后置处理器的方法 postProcessBeforeInitialization
>
> -  （4）调用 bean 的初始化的方法（需要进行配置初始化的方法）
>
> -  （5）把 bean 实例传递 bean 后置处理器的方法 postProcessAfterInitialization
>
> -  （6）bean 可以使用了（对象获取到了）
>
> -  （7）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）
>

###  外部属性文件

方式一：直接配置数据库信息 ：（1）配置Druid（德鲁伊）连接池 （2）引入Druid（德鲁伊）连接池依赖 jar 包

~~~xml
	<!--直接配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
~~~


方式二：引入外部属性文件配置数据库连接池

（1）创建外部属性文件，properties 格式文件，写数据库信息（jdbc.properties）

```properties
prop.driverClass=com.mysql.jdbc.Driver
prop.url=jdbc:mysql://localhost:3306/userDb
prop.userName=root
prop.password=root
```
（2）把外部 properties 属性文件引入到 spring 配置文件中 —— 引入 context 名称空间

~~~xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"><!--引入context名称空间-->
    

    <!--引入外部属性文件-->
	<context:property-placeholder location="classpath:jdbc.properties"/>

    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.password}"></property>
    </bean>

</beans>
~~~

## 三、IOC 操作 Bean 管理(基于注解方式)

### 什么是注解

 （1）注解是代码特殊标记，格式：@注解名称(属性名称=属性值, 属性名称=属性值…)

 （2）使用注解，注解作用在类上面，方法上面，属性上面

 （3）使用注解目的：简化 xml 配置

###  Spring 针对 Bean 管理中创建对象提供注解

 下面四个注解功能是一样的，都可以用来创建 bean 实例

 （1）@Component

 （2）@Service

 （3）@Controller

 （4）@Repository

###  基于注解方式实现对象创建

 第一步 引入依赖 （引入spring-aop jar包）

 第二步 开启组件扫描

~~~xml
<!--开启组件扫描
 1 如果扫描多个包，多个包使用逗号隔开
 2 扫描包上层目录
-->
<context:component-scan base-package="com.atguigu"></context:component-scan>
~~~


 第三步 创建类，在类上面添加创建对象注解

~~~java
//在注解里面 value 属性值可以省略不写，
//默认值是类名称，首字母小写
//UserService -- userService
@Component(value = "userService") //注解等同于XML配置文件：<bean id="userService" class=".."/>
public class UserService {
    public void add() {
        System.out.println("service add.......");
    }
}
~~~

### 开启组件扫描细节配置

~~~xml
<!--示例 1
 use-default-filters="false" 表示现在不使用默认 filter，自己配置 filter
 context:include-filter ，设置扫描哪些内容
-->
<context:component-scan base-package="com.atguigu" use-default-filters="false">
 <context:include-filter type="annotation"
                         
expression="org.springframework.stereotype.Controller"/><!--代表只扫描Controller注解的类-->
</context:component-scan>


<!--示例 2
 下面配置扫描包所有内容
 context:exclude-filter： 设置哪些内容不进行扫描
-->
<context:component-scan base-package="com.atguigu">
 <context:exclude-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--表示Controller注解的类之外一切都进行扫描-->
</context:component-scan>
~~~

###  于注解方式实现属性注入

 （1）@Autowired：根据属性类型进行自动装配

 第一步 把 service 和 dao 对象创建，在 service 和 dao 类添加创建对象注解

第二步 在 service 注入 dao 对象，在 service 类添加 dao 类型属性，在属性上面使用注解

~~~java
@Service
public class UserService {
 //定义 dao 类型属性
 //不需要添加 set 方法
 //添加注入属性注解
 @Autowired
 private UserDao userDao;
 public void add() {
 System.out.println("service add.......");
 userDao.add();
 }
}

//Dao实现类
@Repository
//@Repository(value = "userDaoImpl1")
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("dao add.....");
    }
}
~~~


 （2）@Qualifier：根据名称进行注入，这个@Qualifier 注解的使用，和上面@Autowired 一起使用

~~~java
//定义 dao 类型属性
//不需要添加 set 方法
//添加注入属性注解
@Autowired //根据类型进行注入
//根据名称进行注入（目的在于区别同一接口下有多个实现类，根据类型就无法选择，从而出错！）
@Qualifier(value = "userDaoImpl1") 
private UserDao userDao;
~~~


（3）@Resource：可以根据类型注入，也可以根据名称注入（它属于javax包下的注解，不推荐使用！）

~~~java
//@Resource //根据类型进行注入
@Resource(name = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;
~~~


 （4）@Value：注入普通类型属性

~~~java
@Value(value = "abc")
private String name
~~~

###  完全注解开发

 （1）创建配置类，替代 xml 配置文件

~~~java
@Configuration //作为配置类，替代 xml 配置文件
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {
    
}
~~~

 （2）编写测试类

~~~java
@Test
public void testService2() {
 //加载配置类
 ApplicationContext context
 = new AnnotationConfigApplicationContext(SpringConfig.class);
 UserService userService = context.getBean("userService",
UserService.class);
 System.out.println(userService);
 userService.add();
}
~~~



