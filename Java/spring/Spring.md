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

### 1、IOC操作Bean管理

​		 a）Bean管理就是两个操作：

​				（1）Spring创建对象；

​				（2）Spring注入属性

###  2、基于XML配置文件创建对象

~~~xml
<!--1 配置User对象创建-->
<bean id="user" class="com.atguigu.spring5.User"></bean>

~~~

###  3、基于XML方式注入属性（DI：依赖注入（注入属性））

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

###  4、注入空值和特殊符号

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

### 5、注入属性-外部bean

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

###  6、基于XML方式注入内部bean和级联赋值

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







