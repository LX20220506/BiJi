# SSM整合

## 1、ContextLoaderListener

Spring提供了监听器ContextLoaderListener，实现ServletContextListener接口，可监听
ServletContext的状态，在web服务器的启动，读取Spring的配置文件，创建Spring的IOC容器。web
应用中必须在web.xml中配置

~~~xml
<listener>
<!--
配置Spring的监听器，在服务器启动时加载Spring的配置文件
Spring配置文件默认位置和名称：/WEB-INF/applicationContext.xml
可通过上下文参数自定义Spring配置文件的位置和名称
-->
    <listenerclass>
    org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!--自定义Spring配置文件的位置和名称-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring.xml</param-value>
</context-param>
~~~



## 2、准备工作

### ①创建Maven Module

### ②导入依赖

~~~xml
<packaging>war</packaging>

<properties>
    <spring.version>5.3.1</spring.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!--springmvc-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- Mybatis核心 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>
    <!--mybatis和spring的整合包-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.6</version>
    </dependency>
    <!-- 连接池 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.9</version>
    </dependency>
    <!-- junit测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <!-- MySQL驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.16</version>
    </dependency>
    <!-- log4j日志 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>5.2.0</version>
    </dependency>
    <!-- 日志 -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
    <!-- ServletAPI -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.12.1</version>
    </dependency>
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.1</version>
    </dependency>
    <!-- Spring5和Thymeleaf整合包 -->
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring5</artifactId>
        <version>3.0.12.RELEASE</version>
    </dependency>
</dependencies>
~~~



### ③创建表

~~~sql
CREATE TABLE `t_emp` (
`emp_id` int(11) NOT NULL AUTO_INCREMENT,
`emp_name` varchar(20) DEFAULT NULL,
`age` int(11) DEFAULT NULL,
`sex` char(1) DEFAULT NULL,
`email` varchar(50) DEFAULT NULL,
PRIMARY KEY (`emp_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
~~~



## 3、配置web.xml

~~~xml
<!-- 配置Spring的编码过滤器 -->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filterclass>
        org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
<!-- 配置处理请求方式PUT和DELETE的过滤器 -->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filterclass>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
<!-- 配置SpringMVC的前端控制器 -->
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servletclass>
    <!-- 设置SpringMVC的配置文件的位置和名称 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:SpringMVC.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
<!-- 设置Spring的配置文件的位置和名称 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:Spring.xml</param-value>
</context-param>
<!-- 配置Spring的监听器 -->
<listener>
    <listenerclass>
        org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
~~~



## 4、创建SpringMVC的配置文件并配置

~~~xml
<!--扫描组件-->
<context:component-scan base-package="com.atguigu.ssm.controller">
</context:component-scan>

<!--配置视图解析器-->
<bean id="viewResolver"
      class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
    <property name="order" value="1"/>
    <property name="characterEncoding" value="UTF-8"/>
    <property name="templateEngine">
        <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
            <property name="templateResolver">
                <bean
                      class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                    <!-- 视图前缀 -->
                    <property name="prefix" value="/WEB-INF/templates/"/>
                    <!-- 视图后缀 -->
                    <property name="suffix" value=".html"/>
                    <property name="templateMode" value="HTML5"/>
                    <property name="characterEncoding" value="UTF-8" />
                </bean>
            </property>
        </bean>
    </property>
</bean>

<!-- 配置访问首页的视图控制 -->
<mvc:view-controller path="/" view-name="index"></mvc:view-controller>

<!-- 配置默认的servlet处理静态资源 -->
<mvc:default-servlet-handler />

<!-- 开启MVC的注解驱动 -->
<mvc:annotation-driven />
~~~



## 5、搭建MyBatis环境

### ①创建属性文件jdbc.properties

~~~properties
jdbc.user=root
jdbc.password=123456
jdbc.url=jdbc:mysql://localhost:3306/ssm?serverTimezone=UTC
jdbc.driver=com.mysql.cj.jdbc.Driver
~~~



### ②创建MyBatis的核心配置文件mybatis-config.xml

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--将下划线映射为驼峰-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    
    <plugins>
        <!--配置分页插件-->
        <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
    </plugins>
</configuration>
~~~

### ③创建Mapper接口和映射文件

~~~java
public interface EmployeeMapper {
    List<Employee> getEmployeeList();
}
~~~

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.ssm.mapper.EmployeeMapper">
    <select id="getEmployeeList" resultType="Employee">
        select * from t_emp
    </select>
</mapper>
~~~

### ④创建日志文件log4j.xml

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <param name="Encoding" value="UTF-8" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS}
                                                   %m (%F:%L) \n" />
        </layout>
    </appender>
    <logger name="java.sql">
        <level value="debug" />
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info" />
    </logger>
    <root>
        <level value="debug" />
        <appender-ref ref="STDOUT" />
    </root>
</log4j:configuration>
~~~



## 6、创建Spring的配置文件并配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           https://www.springframework.org/schema/context/spring-context.xsd">
    <!--扫描组件-->
    <context:component-scan base-package="com.atguigu.ssm">
        <context:exclude-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!-- 引入jdbc.properties -->
    <context:property-placeholder location="classpath:jdbc.properties">
    </context:property-placeholder>
    <!-- 配置Druid数据源 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"></property>
        <property name="url" value="${jdbc.url}"></property>
        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
    </bean>
    <!-- 配置用于创建SqlSessionFactory的工厂bean -->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 设置MyBatis配置文件的路径（可以不设置） -->
        <property name="configLocation" value="classpath:mybatis-config.xml">
        </property>
        <!-- 设置数据源 -->
        <property name="dataSource" ref="dataSource"></property>
        <!-- 设置类型别名所对应的包 -->
        <property name="typeAliasesPackage" value="com.atguigu.ssm.pojo">
        </property>
        <!--
设置映射文件的路径
若映射文件所在路径和mapper接口所在路径一致，则不需要设置
-->
        <!--
<property name="mapperLocations" value="classpath:mapper/*.xml">
</property>
-->
    </bean>
    <!--
配置mapper接口的扫描配置
由mybatis-spring提供，可以将指定包下所有的mapper接口创建动态代理
并将这些动态代理作为IOC容器的bean管理
-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.atguigu.ssm.mapper"></property>
    </bean>
</beans>
~~~

## 7、测试功能

### ①创建组件

实体类Employee

~~~java
public class Employee {
    private Integer empId;
    private String empName;
    private Integer age;
    private String sex;
    private String email;
    public Employee() {
    }
    public Employee(Integer empId, String empName, Integer age, String sex,
                    String email) {
        this.empId = empId;
        this.empName = empName;
        this.age = age;
        this.sex = sex;
        this.email = email;
    }
    public Integer getEmpId() {
        return empId;
    }
    public void setEmpId(Integer empId) {
        this.empId = empId;
    }
    public String getEmpName() {
        return empName;
    }
    public void setEmpName(String empName) {
        this.empName = empName;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
        this.email = email;
    }
}
~~~

创建控制层组件EmployeeController

~~~java
@Controller
public class EmployeeController {
    
    @Autowired
    private EmployeeService employeeService;
    
    @RequestMapping(value = "/employee/page/{pageNum}", method =
                    RequestMethod.GET)
    public String getEmployeeList(Model model, @PathVariable("pageNum") Integer
                                  pageNum){
        PageInfo<Employee> page = employeeService.getEmployeeList(pageNum);
        model.addAttribute("page", page);
        return "employee_list";
    }
}
~~~

创建接口EmployeeService

~~~java
public interface EmployeeService {
    PageInfo<Employee> getEmployeeList(Integer pageNum);
}
~~~

创建实现类EmployeeServiceImpl

~~~java
@Service
public class EmployeeServiceImpl implements EmployeeService {
    @Autowired
    private EmployeeMapper employeeMapper;
    
    @Override
    public PageInfo<Employee> getEmployeeList(Integer pageNum) {
        PageHelper.startPage(pageNum, 4);
        List<Employee> list = employeeMapper.getEmployeeList();
        PageInfo<Employee> page = new PageInfo<>(list, 5);
        return page;
    }
}
~~~

### ②创建页面

~~~html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Employee Info</title>
        <link rel="stylesheet" th:href="@{/static/css/index_work.css}">
    </head>
    
    <body>
        
        <table>
            <tr>
                <th colspan="6">Employee Info</th>
            </tr>
            <tr>
                <th>emp_id</th>
                <th>emp_name</th>
                <th>age</th>
                <th>sex</th>
                <th>email</th>
                <th>options</th>
            </tr>
            
            <tr th:each="employee : ${page.list}">
                <td th:text="${employee.empId}"></td>
                <td th:text="${employee.empName}"></td>
                <td th:text="${employee.age}"></td>
                <td th:text="${employee.sex}"></td>
                <td th:text="${employee.email}"></td>
                <td>
                    <a href="">delete</a>
                    <a href="">update</a>
                </td>
            </tr>
            <tr>
                <td colspan="6">
                    <span th:if="${page.hasPreviousPage}">
                        <a th:href="@{/employee/page/1}">首页</a>
                        <a th:href="@{'/employee/page/'+${page.prePage}}">上一页</a>
                    </span>
                    <span th:each="num : ${page.navigatepageNums}">
                        <a th:if="${page.pageNum==num}"
                           th:href="@{'/employee/page/'+${num}}" th:text="'['+${num}+']'" style="color:
                                                                                                 red;"></a>
                        <a th:if="${page.pageNum!=num}"
                           th:href="@{'/employee/page/'+${num}}" th:text="${num} "></a>
                    </span>
                    <span th:if="${page.hasNextPage}">
                        <a th:href="@{'/employee/page/'+${page.nextPage}}">下一页</a>
                        <a th:href="@{'/employee/page/'+${page.pages}}">末页</a>
                    </span>
                </td>
            </tr>
        </table>
    </body>
</html>
~~~

### ③访问测试分页功能

localhost:8080/employee/page/1





~~~json
{"cityid":"101280601",
 "city":"深圳",
 "cityEn":"shenzhen",
 "country":"中国",
 "countryEn":"China",
 "update_time":"2022-07-22 14:24:02",
 "data":[
     {"day":"22日（星期五）",
      "date":"2022-07-22",
      "week":"星期五",
      "wea":"晴",
      "wea_img":"qing",
      "wea_day":"晴",
      "wea_day_img":"qing",
      "wea_night":"晴",
      "wea_night_img":"qing",
      "tem":"34",
      "tem1":"35",
      "tem2":"28",
      "humidity":"59%",
      "visibility":"30km",
      "pressure":"1003",
      "win":["无持续风向","无持续风向"],
      "win_speed":"<3级",
      "win_meter":"10km\/h",
      "sunrise":"05:51",
      "sunset":"19:09",
      "air":"36",
      "air_level":"优",
      "air_tips":"",
      "phrase":"大部地区晴朗",
      "narrative":"大部晴朗。 最高 37ºC。 西南 风 10 到 15 每 km \/ h 。",
      "moonrise":"00:39",
      "moonset":"13:56",
      "moonPhrase":"残月",
      "rain":"4",
      "uvIndex":"11",
      "uvDescription":"极强",
      "alarm":[]
     },{"day":"23日（星期六）","date":"2022-07-23","week":"星期六","wea":"晴","wea_img":"qing","wea_day":"晴","wea_day_img":"qing","wea_night":"晴","wea_night_img":"qing","tem":"35","tem1":"35","tem2":"28","humidity":"57%","visibility":"","pressure":"","win":["无持续风向","无持续风向"],"win_speed":"<3级","win_meter":"","sunrise":"05:51","sunset":"19:09","air":"40","air_level":"优","air_tips":"","phrase":"晴朗","narrative":"大部晴朗。 最高 36ºC。 西南偏西 风 10 到 15 每 km \/ h 。","moonrise":"01:15","moonset":"14:49","moonPhrase":"残月","rain":"5","uvIndex":"11","uvDescription":"极强"},{"day":"24日（星期日）","date":"2022-07-24","week":"星期日","wea":"晴","wea_img":"qing","wea_day":"晴","wea_day_img":"qing","wea_night":"晴","wea_night_img":"qing","tem":"35","tem1":"35","tem2":"28","humidity":"57%","visibility":"","pressure":"","win":["无持续风向","无持续风向"],"win_speed":"<3级","win_meter":"","sunrise":"05:51","sunset":"19:08","air":"27","air_level":"优","air_tips":"","phrase":"晴朗","narrative":"大部晴朗。 最高 36ºC。 西南偏西 风 10 到 15 每 km \/ h 。","moonrise":"01:55","moonset":"15:44","moonPhrase":"残月","rain":"5","uvIndex":"11","uvDescription":"极强"},{"day":"25日（星期一）","date":"2022-07-25","week":"星期一","wea":"晴","wea_img":"qing","wea_day":"晴","wea_day_img":"qing","wea_night":"晴","wea_night_img":"qing","tem":"34","tem1":"34","tem2":"28","humidity":"59%","visibility":"","pressure":"","win":["无持续风向","无持续风向"],"win_speed":"<3级","win_meter":"","sunrise":"05:52","sunset":"19:08","air":"25","air_level":"优","air_tips":"","phrase":"大部地区晴朗","narrative":"大部晴朗。 最高 36ºC。 西南 风 10 到 15 每 km \/ h 。","moonrise":"02:38","moonset":"16:37","moonPhrase":"残月","rain":"5","uvIndex":"11","uvDescription":"极强"},{"day":"26日（星期二）","date":"2022-07-26","week":"星期二","wea":"晴","wea_img":"qing","wea_day":"晴","wea_day_img":"qing","wea_night":"晴","wea_night_img":"qing","tem":"34","tem1":"34","tem2":"28","humidity":"60%","visibility":"","pressure":"","win":["无持续风向","无持续风向"],"win_speed":"<3级","win_meter":"","sunrise":"05:52","sunset":"19:07","air":"27","air_level":"优","air_tips":"","phrase":"大部地区晴朗","narrative":"大部晴朗。 最高 36ºC。 西南偏南 风 10 到 15 每 km \/ h 。","moonrise":"03:25","moonset":"17:30","moonPhrase":"残月","rain":"5","uvIndex":"11","uvDescription":"极强"},{"day":"27日（星期三）","date":"2022-07-27","week":"星期三","wea":"晴","wea_img":"qing","wea_day":"晴","wea_day_img":"qing","wea_night":"晴","wea_night_img":"qing","tem":"34","tem1":"34","tem2":"28","humidity":"59%","visibility":"","pressure":"","win":["无持续风向","无持续风向"],"win_speed":"<3级","win_meter":"","sunrise":"05:53","sunset":"19:07","air":"36","air_level":"优","air_tips":"","phrase":"局部多云","narrative":"少云。 最高 36ºC。 西南 风 10 到 15 每 km \/ h 。","moonrise":"04:15","moonset":"18:18","moonPhrase":"残月","rain":"7","uvIndex":"11","uvDescription":"极强"},{"day":"28日（星期四）","date":"2022-07-28","week":"星期四","wea":"晴","wea_img":"qing","wea_day":"晴","wea_day_img":"qing","wea_night":"晴","wea_night_img":"qing","tem":"35","tem1":"35","tem2":"27","humidity":"60%","visibility":"","pressure":"","win":["无持续风向","无持续风向"],"win_speed":"<3级","win_meter":"","sunrise":"05:53","sunset":"19:07","air":"37","air_level":"优","air_tips":"","phrase":"大部地区晴朗","narrative":"大部晴朗。 最高 36ºC。 西南 风 10 到 15 每 km \/ h 。","moonrise":"05:09","moonset":"19:04","moonPhrase":"新月","rain":"9","uvIndex":"11","uvDescription":"极强"}],"nums":7108}
~~~

