# Spring5 框架新功能

1、整个 Spring5 框架的代码基于 Java8，运行时兼容 JDK9，许多不建议使用的类和方 法在代码库中删除

## 整合 Log4j2 日志

 Spring 5.0 框架自带了通用的日志封装 

（1）Spring5 已经移除 Log4jConfigListener，官方建议使用 Log4j2 

（2）Spring5 框架整合 Log4j2 

​		第一步 引入 jar 包

~~~xml
<!-- 使用 log4j2 的适配器进行绑定 包含了slf4j-api、log4j-api、log4j-core-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.11.2</version>
        </dependency>
~~~

​		第二步 配置文件 log4j2.xml（名称固定）

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 日志级别优先顺序：OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!-- configuration 后面的status 用于设置log4j2自身内部的信息输出，
     当设置为trace时,可以看到log4j2内部详细信息输出-->
<configuration status="warn">
    <!--
    集中配置属性进行管理
    使用时通过:${name}
    -->
    <properties>
        <property name="LOG_HOME">E:/logs/study</property>
        <!-- 格式化输出：
        %date / %d  表示日期;  %thread / %t 表示线程名;
        %logger{36} / %l  表示Logger名字最长36个字符;
        %msg / %m 表示日志消息;  %n是换行符;
        %-5level   级别从左显示5个字符宽度;
        -->
        <property name="LOG_CONSOLE_OUT">%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] [%-5level] %l{36} - %m%n</property>
        <property name="LOG_FILE_OUT">%date{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%-5level] %logger{36} - %msg%n
        </property>
    </properties>

    <!-- 日志处理 -->
    <appenders>
        <!-- 1.控制台输出   SYSTEM_OUT 输出黑色，SYSTEM_ERR输出红色 -->
        <console name="console" target="SYSTEM_OUT">
            <!-- 文件输出格式-->
            <PatternLayout pattern="${LOG_CONSOLE_OUT}"/>
        </console>

        <!-- 2.文件输出-->
        <!-- 文件输出还可以配置  File、RandomAccessFile、RollingFile、RollingRandomAccessFile 等节点 实现不同要求的文件输出 -->
        <!-- fileName：文件拆分前的名称-->
        <!-- filePattern：文件拆分后的名称，满足拆分条件 进行拆分-->
        <RollingRandomAccessFile name="rollingRandomAccessFile"
                                 fileName="${LOG_HOME}/rollingRandomAccessFile.log"
                                 filePattern="${LOG_HOME}/$${date:yyyy-MM-dd}/%d{yyyy-MM-dd HHmm}.log">
            <PatternLayout pattern="${LOG_FILE_OUT}"/>

            <!-- 日志级别过滤器  info (上) > debug (下)-->
            <!--
            onMatch="ACCEPT" 表示匹配该级别及以上
            onMatch="DENY" 表示不匹配该级别及以上
            onMatch="NEUTRAL" 表示该级别及以上的，由下一个filter处理，如果当前是最后一个，则表示匹配该级别及以上
            onMismatch="ACCEPT" 表示匹配该级别以下
            onMismatch="NEUTRAL" 表示该级别及以下的，由下一个filter处理，如果当前是最后一个，则不匹配该级别以下的
            onMismatch="DENY" 表示不匹配该级别以下的
            -->
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>

            <!-- 文件拆分条件：满足拆分条件，则新建空文件，把已有文件内容存放到 filePattern 配置的地址中-->
            <Policies>
                <!-- 按照时间节点拆分，规则根据filePattern定义的 -->
                <TimeBasedTriggeringPolicy/>
                <!-- 按照文件大小拆分，10MB -->
                <SizeBasedTriggeringPolicy size="2MB"/>
                <!-- 每启动一次系统，则生成一个新的日志文件 -->
                <!-- OnStartupTriggeringPolicy/>-->
            </Policies>
        </RollingRandomAccessFile>
    </appenders>

    <!-- loggers 定义 -->
    <Loggers>
        <!--appenders 定义的子节点，只有在 logggers 中使用 AppenderRef 进行引用之后才会生效-->
        <root level="info">
            <AppenderRef ref="console"/>
            <AppenderRef ref="rollingRandomAccessFile"/>
        </root>
        <!-- 为指定包 设置日志级别 , JdbcTemplate  打印sql语句-->
        <logger name="org.springframework.jdbc.core.JdbcTemplate" level="DEBUG">
            <!-- 如果定义了AppenderRef
                 则该日志信息只在配置的AppenderRef中生效，
                 否则继承root下的全部AppenderRef -->
            <AppenderRef ref="console"/>
        </logger>
    </Loggers>
</configuration>
~~~

> 此时可以自动输出内容



### 手动进行日志记录

第一步 先定义 org.slf4j.Logger

第二步 进行手动日志输出

~~~java
public class TestDemo{
    // 第一步
	private static final Logger log = LoggerFactory.getLogger(TestDemo.class);
    
    public void main(String[] args){
        // 第二步
     	log.error("手动记录日志内容1")
     	log.info("手动记录日志内容2")
    }
}
~~~

## 支持@Nullable 注解 

（1）@Nullable 注解可以使用在方法上面，属性上面，参数上面，表示方法返回可以为空，属性值可以 为空，参数值可以为空 

（2）注解用在方法上面，方法返回值可以为空 

~~~java
@Nullable
String getId();
~~~

（3）注解使用在方法参数里面，方法参数可以为空

~~~java
public ClassPathXmlApplicationContext(String[] configLocations, @Nullable ApplicationContext parent)
    throws BeansException {

    this(configLocations, true, parent);
}
~~~

（4）注解使用在属性上面，属性值可以为空

~~~java
@Nullable
private UserDao userDao;
~~~

## 支持函数式风格 GenericApplicationContext

~~~java
//函数式风格创建对象，交给 spring 进行管理
@Test
public void testGenericApplicationContext() {
    //1 创建 GenericApplicationContext 对象
    GenericApplicationContext context = new GenericApplicationContext();
    //2 调用 context 的方法对象注册
    context.refresh();
    context.registerBean("user1",User.class,() -> new User());
    //3 获取在 spring 注册的对象
    // User user = (User)context.getBean("com.atguigu.spring5.test.User");
    User user = (User)context.getBean("user1");
    System.out.println(user);
}
~~~

## 支持整合 JUnit5

### 整合 JUnit4 

第一步 引入 Spring 相关针对测试依赖

第二步 创建测试类，使用注解方式完成

~~~java
@RunWith(SpringJUnit4ClassRunner.class) //单元测试框架
@ContextConfiguration("classpath:bean1.xml") //加载配置文件
public class JTest4 {
    @Autowired
    private UserService userService;
    @Test
    public void test1() {
        userService.accountMoney();
    }
}
~~~

### 整合 JUnit5

第一步 引入 JUnit5 的 jar 包

~~~xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.1</version>
    <scope>test</scope>
</dependency>
~~~

第二步 创建测试类，使用注解完成

~~~java
@ExtendWith(SpringExtension.class) 
@ContextConfiguration("classpath:bean1.xml") 
public class JTest5 { 
    @Autowired 
    private UserService userService; 
    
    @Test 
    public void test1() { 
        userService.accountMoney(); 
    }
~~~

使用一个复合注解替代上面两个注解完成整合

~~~java
@SpringJUnitConfig(locations = "classpath:bean1.xml")
public class JTest5 {
    @Autowired
    private UserService userService;
    @Test
    public void test1() {
        userService.accountMoney();
    }
}
~~~

