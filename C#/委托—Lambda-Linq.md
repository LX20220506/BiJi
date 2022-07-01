# 反射

## 反射是什么

### C#编译运行过程

高级语言->编译->dll/exe文件->CLR/JIT->机器码

![](E:\筆記\img\反射\1.png)

### 原理解析

metadata：元数据数据清单，记录了dll中包含了哪些东西,是一个描述。

IL：中间语言，编译把高级语言编译后得到的C#中最真实的语言状态，面向对象语言。

反射：来自于System.Reflection，是一个帮助类库，可以读取dll/exe中metadata，使用metadata创建对象。

Emit：一种反射技术，可以动态创建dll/exe。

反编译工具：ILSpy可以反编译dll/exe，查看对应的C#/IL代码。

### 程序集

程序集是经由编译器编译得到的，供进一步编译执行的中间产物。
在Windows系统中，它一般表现为后缀为.dll（库文件）或者.exe（可执行文件）的格式。

### 元数据

元数据是用来描述数据的数据。即程序中的类、类中的函数、变量等信息就是程序的元数据。有关程序以及类型的数据被称为元数据，他们保存在程序集中。

### 反射的概念

程序正在运行时，可以查看其它程序集或者自身的元数据。
一个运行的程序查看本身或者其它程序的元数据的行为叫做反射。
说的通俗点就是在程序运行时，通过反射可以得到其它程序集或者自己程序集代码的各种信息，包括类、函数、变量等来实例化它们，执行它们，操作它们。

### 反射的作用

因为反射可以在程序编译后获得信息，所以它提高了程序的拓展性和灵活性。
1、程序运行时得到所有元数据，包括元数据的特性。
2、程序运行时实例化对象，操作对象。
3、程序运行时创建新的对象，用这些对象执行任务。

## Type

Type是类的信息类，它是反射功能的挤出，访问元数据的主要方式。
使用Type的成员获取有关类型（如构造函数、方法、字段、属性和类的事件等）声明的信息。

### 获取Type

得到类的程序集信息
1、object中的GetType()可以获取对象的Type

```C#
        int a = 42;
        Type type = a.GetType();
        Console.WriteLine(type);
```
打印结果是System.Int32即为a的类型

2、通过typeof关键字，传入类名可以获取对象的Type

```C#
        Type type2 = typeof(int);
        Console.WriteLine(type2);
```

打印结果是System.Int32即为int的类型

3、通过类的名字可以获取类型，这里要注意类名必须包含明明空间，否则会找不到。

```c#
        Type type3 = Type.GetType("Int32");
        Console.WriteLine(type3);
        Type type4 = Type.GetType("System.Int32");
        Console.WriteLine(type4);
```
打印结果type3为空，type4为System.Int32

得到类的程序集信息
可以通过Type得到类型所在程序集信息。

```C#
        Console.WriteLine(type.Assembly);
        Console.WriteLine(type2.Assembly);
        Console.WriteLine(type4.Assembly);
```
## 读取程序集

- LoadFrom：dll全名称，需要后缀
- LoadFile：全路径，需要dll后缀
- Load：dll名称不需要后缀

~~~C#
//1、动态读取dll的三种方式
//（1）LoadFrom：dll全名称，需要后缀                        
Assembly assembly = Assembly.LoadFrom("Business.DB.SqlServer.dll");
//（2）LoadFile：全路径，需要dll后缀
//Assembly assembly1 = Assembly.LoadFile(@"dll文件全路径");
//（3）Load：dll名称 不需要后缀
//Assembly assembly2 = Assembly.Load("Business.DB.SqlServer");

~~~

## 获取类型

- GetTypes()：获取集合（推荐使用）
- GetType()：获取单个类型（在程序集中只有有一个cs文件时可以使用，不推荐）

~~~C#
//2、获取某一个具体的类型，参数需要是类的全名称
Type type1 = assembly.GetType("Business.DB.SqlServer.SqlServerHelper");
~~~

## 通过反射创建对象

~~~C#
// 创建对象
// 1.拿到程序集
Assembly assembly = Assembly.Load("Server1");
// 2.拿到类型集合
Type[] types = assembly.GetTypes();
// 3.创建对象（注：types[0].Name无法创建对象，要使用types[0].FullName才行）
IServerHelper serverHelper = assembly.CreateInstance(stypes[1].FullName) as IServerHelper;
serverHelper.Show();
~~~

~~~C#
// 有参构造函数
// 使用Activator可以快速创建对象
Type[] typea = Assembly.Load("Server1").GetTypes();
Type type = Assembly.Load("Server1").GetTypes().SingleOrDefault(a=>a.Name=="DBHelper");
// 不同有参构造函数，创建对象
DBHelper dBHelper1 = (DBHelper)Activator.CreateInstance(type);
DBHelper dBHelper2 = (DBHelper)Activator.CreateInstance(type,"sssss");
DBHelper dBHelper3 = (DBHelper)Activator.CreateInstance(type,123,"AAAAAAA");
~~~

~~~C#
// 创建泛型对象
Type type = Assembly.Load("Server1").GetType("Server1.FilterHelper`1");

// 泛型类 获取到类型后需要先确定类型
Type newType = type.MakeGenericType(typeof(string));

FilterHelper<string> filterHelper = (FilterHelper<string>)Activator.CreateInstance(newType);

// 创建泛型方法
MethodInfo method = newType.GetMethod("Show2");
// 泛型方法 获取到方法后需要先确定方法类型
MethodInfo nweMethod =method.MakeGenericMethod(new Type[] { typeof(int), typeof(string) });
// 执行方法
nweMethod.Invoke(filterHelper,new object[] { 12, "tom","jie" });
~~~

## 反射操作属性和成员

~~~C#
// 获取属性
Type type = Assembly.Load("Server1").GetType("Server1.CacheHelper");

var propList= type.GetProperties();// 拿到所有的属性

foreach (var prop in propList)
{
    Console.WriteLine(prop.Name);
}

// 获取成员
var membersList = type.GetFields();// 拿到所有的成员

foreach (var prop in membersList)
{
    Console.WriteLine(prop.Name);
}
~~~

~~~C#
CacheHelper cacheHelper = new CacheHelper() {
                    IP="1027.0.0.1",
                    Pwd="123456"
					};

//反射方式赋值取值
Console.WriteLine("***********反射方式赋值取值*************");
Type type = typeof(CacheHelper);
object pObject = Activator.CreateInstance(type);
foreach (var prop in type.GetProperties())
{
    if (prop.Name.Equals("IP"))
    {
        // 给pObject 对象的 prop属性赋值为 "0.0.0.0"
        prop.SetValue(pObject, "0.0.0.0");
    }
    else if (prop.Name.Equals("Pwd"))
    {
        prop.SetValue(pObject, "WWWW");
    }
}
foreach (var prop in type.GetProperties())
{
    Console.WriteLine($"people.{prop.Name}={prop.GetValue(pObject)}");
}
~~~

## Activator

Activator是用于快速实例化对象的类，用于将Type对象快捷实例化为对象。
先得到Type，然后快速实例化一个对象。
Activator.CreateInstance默认调用无参构造函数。
1、无参构造函数：

~~~C#
Type test = typeof(Test);
Test testObj = Activator.CreateInstance(test) as Test;
Console.WriteLine(testObj.str);
~~~

2、有参构造函数：
如果要调用有参构造函数，在后面一次添加参数即可。

~~~C#
 testObj = Activator.CreateInstance(test, 99) as Test;
 testObj = Activator.CreateInstance(test, 55, "11111") as Test;
 Console.WriteLine(testObj.str);
~~~

## Assembly

Assembly类其实就是程序集类。主要用来加载其他程序集，加载后才能用Type来使用其他程序集中的信息，如果想要使用不是自己程序集中的内容，需要先加载程序集（比如dll文件）。
三种加载程序集的函数：
1、一般用来加载同一文件下的其他程序集
Assembly assembly = Assembly.Load(“程序集名称”);

2、一般用来加载不再同一文件下的其他程序集
Assembly assembly = Assembly.LoadFrom(“包含程序集清单的文件的名称或路径”);
Assembly assembly = Assembly.LoadFile(“要加载的文件的完全限定路径”);

使用方法：
1、首先加载一个指定程序集：

~~~c#
Assembly assembly = Assembly.LoadFrom (@"C:\Users\01\Desktop\ConsoleApp1\TestDLL\bin\Debug\TestDLL.dll");

Type[] types = assembly.GetTypes();

for (int i = 0; i < types.Length; i++)
{
    Console.WriteLine(types[i]);
}

~~~

2、再加载程序集中的一个类对象，之后才能使用反射

~~~c#
//得到dll中的Class1类
Type c = assembly.GetType("TestDLL.Class1");
object o = Activator.CreateInstance(c);
//调用Class1类中的Speak方法
MethodInfo speak = c.GetMethod("Speak");
speak.Invoke(o,null);
~~~

其中TestDLL.dll为自己封装的dll，内容如下：

~~~c#
namespace TestDLL
{
    public class Class1
    {
        public int i = 0;
        public string str = "123";
        public void Speak()
        {
            Console.WriteLine("speak");
        }
    }
}
~~~



# 委托

委托是一个方法的变量，他指向方法，使他拥有方法的功能。

委托的别名:代理，句柄
委托是**自定义类型**
委托是**引用类型**

## 语法

> 使用 delegate 关键字定义
>
> 委托的 返回值,参数个,位置,要和 返回值,参数个数,位置,方法的一致

~~~C#
	// 无参无返回值
	delegate void Test1();
	// 无参有返回值
    delegate int Test2();
	// 有参无返回值
    delegate int Test4();
	// 有参有返回值
    delegate int Test3(int a,int b);

	// 调用委托
	// 两种方式
	Test1 t1 = F1;
	t1();

    Test1 test = new Test1(F1);
    test();
	
~~~

## 多播委托

委托可以像数字类型一样，可以正常的加减，执行过程会根据加减的顺序执行

> 注：在多波委托中，若方法有返回值，他只会返回最后执行的方法

示例：

~~~C#
namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            Test1 t1 = F1;
            Test1 test = new Test1(F1);
            t1();
            test();

            Test2 test2 = F2;
            test2();

            Test3 test3 = Reduce;
            test3(2, 3);

            test3 += Add;

            var ss = test3(3,5);
            Console.WriteLine(ss);
        }

        static void F1() {
            Console.WriteLine("F1");
        }

        static int F2() {
            return 100;
        }

        static int Add(int a, int b) {
            return a + b;
        }

        static int Reduce(int a, int b) {
            return b - a;
        }
    }

    delegate void Test1();

    delegate int Test2();

    delegate int Test3(int a,int b);
}
~~~

## Action和Func

>Action<>()：是一个系统自定义的无返回值的委托
>
>Func<>()：是一个系统自定义的有返回值得委托(该委托至少有一个类型 Tout，该类型为委托的返回值类型；若有多个类型，则最后一个类型为 Func<>() 的返回值类型)



示例：

~~~c#
    //  action
	int a = 5;
    Action<int> action = s => {
        int a=s + 5;
    };
    action(a);

	// func
	int a = 5;
    int b;
    Func<int, int> func = s => s * s;
    b = func(a);
~~~

# Lambda

Lambda 表达式是一种可用于创建委托或表达式目录树类型的匿名函数。通过使用 lambda 表达式，可以写入可作为参数传递或作为函数调用值返回的本地函数。Lambda 表达式对于编写 LINQ 查询表达式特别有用。

他只是一种语法糖，通常用于搭配LINQ，委托，匿名方法 使用

示例：

~~~C#
namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            int a=1;
            int b = 0;
            
            // 若只有一行代码，可以不用{}，也不用return
            // 只有一个参数a
            Test test = a => a + 1;
            b = test(a);

			// 没有参数
            Test2 test2 = () => 1;
            b = test2();

			
            // 两个参数
            int x = 1;
            int y = 5;
            Test3 test3 = (x, y) =>
            {
                x += y;
                x *= y;
                return x;
            };
            b = test3(x, y);
        }
    }

    delegate int Test(int a);
    delegate int Test2();
    delegate int Test3(int x,int y);
}
~~~

# Linq

Linq