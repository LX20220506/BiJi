## 传递参数

前台请求	Content-Type(请求格式) 要设置为application/json(json格式)

[FromQuery]  通过？那数据

[FromRoute]  拿到[HttpGet("{num}")] 中的num   [FromRoute]int num

[FromBody]  获取application/fromdata格式的数据

 [FromHeader]	获取http headers 数据

[FromServices] 可以设置只有在需要的地方依赖注入，大多数情况不要这样做，但是比较只消耗资源较大且不经常使用的情况下才用

## 配置

### 配置方式及加载顺序

1)加载现有的IConfiguration。

2)加载项目根目录下的appsettings.json。

3)加载项目根目录下的appsettings.{Environment}.json。

4)当程序运行在开发环境下，程序会加载“用户机密”配置

5)加载环境变量中的配置。

6)加载命令行中的配置。

该加载顺序是从上到下依次执行；若出现重复配置，后执行的配置信息会覆盖之前配置的信息

### 读取方法(.NET 6)

- app.Environment.EnvironmentName：读取环境变量中的配置
- app.Environment.IsDevelopment()：在开发环境下，读取的配置；例如在开发环境下才会运行UseSwaggerUI界面

### 防止机密配置外泄

- 把不方便放到appsettings.json中的机密信息放到一个不在项目中的json文件中。
- 在ASP.NET Core项目上单击鼠标右键，选择【管理用户机密】；该文件存在本地的C盘下
- csproj文件中的<UserSecretsId>表示所在的文件夹名称

## 过滤器

ASP.NET Core中的Filter的五种类型：

- Authorization filter：授权过滤器
- Resource filter：资源过滤器
- Action filter：方法过滤器
- Exception filter：异常过滤器
- Result filter：结果过滤器

> 不同的筛选器需要继承对应的接口
>
> 所有筛选器一般有同步和异步两个版本，比如IActionFilter、IAsyncActionFilter接口。
>
> 在服务中添加过滤器的时候，一定要注意添加的顺序，根据优先级调整添加顺序；过滤器的执行顺序是 `后添加的先执行`

### Exception Filter

主要用于处理应用的异常信息

~~~C#
// 创建ExceptionFilter
// 需要继承IAsyncExceptionFilter
    public class MyExceptionFilter : IAsyncExceptionFilter
    {
        private readonly IHostEnvironment host;

        // 注入IHostEnvironment 得知运行环境。
        public MyExceptionFilter(IHostEnvironment host)
        {
            this.host = host;
        }

        public Task OnExceptionAsync(ExceptionContext context)
        {
            string message = context.Exception.Message;

            if (host.IsDevelopment()) // 判断是否是开发模式
            {
                Console.WriteLine(message);
            }
            else
            {
                message = "程序中出现异常";
            }
            context.ExceptionHandled = true; // 是否停止执行

            ObjectResult result = new ObjectResult(message) { StatusCode = 500 }; // 设置返回信息
            context.Result = result; // 返回结果

            return Task.CompletedTask; // 结束任务
        }
    }
~~~

~~~C#
// 添加服务
#region 添加Filter
    services.Configure<MvcOptions>(options =>
    {
    	options.Filters.Add<MyExceptionFilter>();
    });
#endregion
~~~

### Action Filter

~~~C#
// Action过滤器
public class MyActionFilter : IAsyncActionFilter
    {
        public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
            Console.WriteLine("执行前");
            ActionExecutedContext err = await next.Invoke(); // 必须要有next，不然程序不会向下执行
            if (err.Exception==null)
            {
                Console.WriteLine("执行成功");
            }
            else
            {
                Console.WriteLine("执行错误");
            }
        }
    }
~~~

~~~C#
// 添加服务
services.Configure<MvcOptions>(option=> {
    option.Filters.Add<MyActionFilter>();
});
~~~

使用事务的过滤器

~~~C#
namespace 自动启用事务的筛选器
{
    public class TransactionScopeFilter : IAsyncActionFilter
    {
        public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
			bool hasNotTransactionalAttribute = false;
			// 判断这个action 是不是控制器中的action
			if (context.ActionDescriptor is ControllerActionDescriptor)
			{
				// 将action转换成controller的action
				var actionDesc = (ControllerActionDescriptor)context.ActionDescriptor;
				hasNotTransactionalAttribute = actionDesc.MethodInfo
					.IsDefined(typeof(NotTransactionalAttribute));
			}
			if (hasNotTransactionalAttribute)
			{
				await next();
				return;
			}
			// 启用事物
			using var txScope =
					new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
			var result = await next();
			if (result.Exception == null)
			{
				txScope.Complete();
			}
		}
    }
}
~~~

#### 配合特性使用ActionFailter（工作单元的实现）

示例：通过特性和过滤器，实现各个数据库上下文对象的自动保存；利用EF Core 中的SaveChangesAsync()的特性实现工作单元；即要么全部执行成功，要么全部不执行(或执行失败)

~~~C#
// 特性
using Microsoft.EntityFrameworkCore;
using System;

namespace User.WebApi
{
    // AttributeTargets.Class：      允许标记在类上
    // AttributeTargets.Method： 允许标记在方法上
    // AllowMultiple：               是否允许叠加
    // Inherited：               是否允许继承
    [AttributeUsage(AttributeTargets.Class|AttributeTargets.Method,AllowMultiple =false,Inherited =true)]
    public class UnitOfWorkAttribute:Attribute
    {
        public Type[] DBContextTypes { get; init; }
        public UnitOfWorkAttribute(params Type[] dbContextTypes) {
            this.DBContextTypes = dbContextTypes;

            foreach (var item in DBContextTypes)
            {
                // 判断item的类型是否继承 DbContext
                if (!typeof(DbContext).IsAssignableFrom(item))
                {
                    // 若不继承   报错（参数异常）
                    throw new ArgumentException($"{item} must inherit from DbContext");
                }
            }

        }

        //public UnitOfWorkAttribute(params Type[] dbContextTypes)
        //{
        //    this.DBContextTypes = dbContextTypes;
        //    foreach (var type in dbContextTypes)
        //    {
        //       // 判断type是否继承自DbContext
        //        if (!typeof(DbContext).IsAssignableFrom(type))
        //        {
        //            throw new ArgumentException($"{type} must inherit from DbContext");
        //        }
        //    }
        //}
    }
}
~~~

~~~C#
// ActionFilter
using Microsoft.AspNetCore.Mvc.Abstractions;
using Microsoft.AspNetCore.Mvc.Controllers;
using Microsoft.AspNetCore.Mvc.Filters;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using System.Collections.Generic;
using System.Reflection;
using System.Threading.Tasks;

namespace User.WebApi
{
    public class UnitOfWorkFilter : IAsyncActionFilter
    {
        public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
            // 获取特性 UnitOfWorkAttribute
            var unArr = GetUnitOfWorkAttribute(context.ActionDescriptor);
            if (unArr==null)
            {
                await next();
                return;
            }

            List<DbContext> dbContexts = new List<DbContext>();

            // 遍历特性中的DBContextTypes属性  同时保证服务的状态
            foreach (var item in unArr.DBContextTypes)
            {
                //用HttpContext的RequestServices
                //确保获取的是和请求相关的Scope实例
                var sp = context.HttpContext.RequestServices;
                var dbcontext = (DbContext)sp.GetRequiredService(item);
                dbContexts.Add(dbcontext);
            }

            var result = next();

            // 判断是否报错
            if (result.Exception==null)
            {
                foreach (var item in dbContexts)
                {
                    // 若不存在错误，将数据保存到各个数据源
                    await item.SaveChangesAsync();
                }
            }
        }

        /// <summary>
        /// 获取拥有UnitOfWorkAttribute特性的controller或action
        /// </summary>
        /// <param name="actionDescriptor"></param>
        /// <returns></returns>
        public UnitOfWorkAttribute GetUnitOfWorkAttribute( ActionDescriptor actionDescriptor) {
            // 将 操作描述符 转换成 控制器操作描述符
            var controllerAction = actionDescriptor as ControllerActionDescriptor;
           
            if (controllerAction==null)
            {
                return null;
            }

            // 获取控制器的自定义特性
            var controllerAttribute =  controllerAction.ControllerTypeInfo.GetCustomAttribute<UnitOfWorkAttribute>();
            if (controllerAttribute!=null)
            {
                return controllerAttribute;
            }
            else
            {
                // 获取方法的自定义特性
                return controllerAction.MethodInfo.GetCustomAttribute<UnitOfWorkAttribute>();
            }
        }
    }
}
~~~

## 中间件

### 什么是中间件

- ASP.NET Core中的中间件指ASP.NET Core中的一个组件
- 中间件由前逻辑、next、后逻辑3部分组成
  - 前逻辑为第一段要执行的逻辑代码、
  - next为指向下一个中间件的调用、
  - 后逻辑为从下一个中间件执行返回所执行的逻辑代码。
- 每个HTTP请求都要经历一系列中间件的处理，每个中间件对于请求进行特定的处理后，再转到下一个中间件，最终的业务逻辑代码执行完成后，响应的内容也会按照处理的相反顺序进行处理，然后形成HTTP响应报文返回给客户端
- 中间件组成一个管道，整个ASP.NET Core的执行过程就是HTTP请求和响应按照中间件组装的顺序在中间件之间流转的过程。开发人员可以对组成管道的中间件按照需要进行自由组合。

### 中间件的三个概念

> 中间件是由Map、Use和Run三部分组成
>
> Map用来定义一个管道可以处理哪些请求，Use和Run用来定义管道，一个管道由若干个Use和一个Run组成，每个Use引入一个中间件，而Run是用来执行最终的核心应用逻辑。

### 简单的自定义中间件

1、如果中间件的代码比较复杂，或者我们需要重复使用一个中间件的话，我们最好把中间件的代码放到一个单独的“中间件类”中。

2、中间件类是一个普通的.NET类，它不需要继承任何父类或者实现任何接口，但是这个类需要有一个构造方法，构造方法至少要有一个`RequestDelegate类型的参数`，这个参数用来指向下一个中间件。这个类还需要定义一个名字为`Invoke或InvokeAsync的方法`，方法`至少有一个HttpContext类型的参数`，方法的`返回值必须是Task类型`。中间件类的构造方法和Invoke（或InvokeAsync）方法还可以定义其他参数，`其他参数的值会通过依赖注入自动赋值`。

示例：

~~~C#
// 中间件
public class MyMiddleware
    {
        private readonly RequestDelegate next;

        public MyMiddleware(RequestDelegate next)
        {
            this.next = next;
        }

        public async Task InvokeAsync(HttpContext context) {
            string pwd = context.Request.Query["pwd"];// 获取Query中的数据
            if (pwd=="123")
            {
                context.Items["ID"] = 123;// 将id存入context中，向下传递；其他中间件可以访问到这个中间件存的数据
                await next(context);// 下一步
                return;
            }
            else
            {
                context.Response.StatusCode = 404;// 返回状态码
            }
           
        }
    }
~~~

~~~C#
// 添加到管道
app.UseMiddleware<MyMiddleware>();// 使用UseMiddleware添加自定义的中间件
~~~

### MarkdownMiddleware(自定义中间件)

该中间件是将Markdown格式（后缀为.md）的文件转换成html格式文件，在浏览器输出

1. 创建mvc项目
2. 在wwwroot文件夹下添加.md的文件
3. 在浏览器的导航栏直接输入文件名称即可查看效果

~~~C#
// 定义中间件
public class mdMiddleware
    {
        private readonly RequestDelegate next;
        private readonly IWebHostEnvironment webHost; // 用来获取服务器的环境

        public mdMiddleware(RequestDelegate next, IWebHostEnvironment webHost)
        {
            this.next = next;
            this.webHost = webHost;
        }

        public async Task InvokeAsync(HttpContext context) {
            string path = context.Request.Path.Value ?? "";
            if (!path.EndsWith(".md"))
            {
                await next(context);
                return;
            }
           var file = webHost.WebRootFileProvider.GetFileInfo(path);// 拿到服务器环境下的文件
            if (!file.Exists)// 判断文件是否存在
            {
                // 若文件不存在，提交到下一个中间件
                await next(context);
                return;
            }
			
            //将文件转换成流的形式
            using Stream stream = file.CreateReadStream();
			
            // 需要引用Ude.NetStandard包
            // 获取文件的编码格式
            CharsetDetector cdet = new CharsetDetector();
            cdet.Feed(stream);
            cdet.DataEnd();
            string charset = cdet.Charset ?? "UTF-8";// 若cdet.Charset为null 那么charset就为UTF-8
            stream.Position = 0; // 因为上面判断文件编码格式时，流已经被扫描一遍了，所以要将扫描的位置调回开始位置

            // 按照获取的文本格式读取流的内容
            // Encoding.GetEncoding(charset)获取charset编码；用该编码读取stream;ReadToEnd()将stream读到结尾
            string mdText = new StreamReader(stream,Encoding.GetEncoding(charset)).ReadToEnd();// 读取文件的内容，读取成为Markdown(.md)格式的文本

            //引用 MarkdownSharp 包
            // 自动将Markdown(.md)格式装换成html格式
            Markdown markdown = new Markdown();
            string html=markdown.Transform(mdText);

            // 设置响应信息配置
            context.Response.ContentType = "text/html;charset=utf-8";
            context.Response.StatusCode = 200;
            await context.Response.WriteAsync(html);
        }
    }
~~~

~~~C#
// startup.cs 下使用中间件（Middleware）
// 这个中间件一定要放在UseStaticFiles()之前；不然会被UseStaticFiles拦截
	app.UseMiddleware<mdMiddleware>();
~~~

## 托管

### 什么是托管

就是在程序运行时，在程序后台执行的代码；托管执行的代码和程序执行的代码并不冲突，我简单理解为同时执行(异步)

### 托管的使用

1、场景，代码运行在后台。比如服务器启动的时候在后台预先加载数据到缓存，每天凌晨3点把数据导出到备份数据库，每隔5秒钟在两张表之间同步一次数据。

2、托管服务实现IHostedService接口，一般编写从BackgroundService继承的类。

3、services.AddHostedService<DemoBgService>();// 添加到服务中 

示例：延迟若干秒再读取文件，再延迟，再输出

~~~C#
// 编写BackgroundService继承类
	public class DemoBackgroundServer : BackgroundService
    {
        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            try
            {
                for (int i = 0; i < 5; i++)
                {
                    await Task.Delay(5000);
                    string txt = await File.ReadAllTextAsync("1.txt");
                    Console.WriteLine(txt);
                    Console.WriteLine("--------------------------------------------");
                }
            }
            catch (Exception ex)
            {

                Console.WriteLine(ex.Message);
            }
        }
    }
	
~~~

~~~C#
// 在服务中添加
    #region 托管
    	services.AddHostedService<DemoBackgroundServer>();// 可以添加多个托管类
    #endregion
~~~

### 托管服务的异常问题

1、从.NET 6开始，当托管服务中发生未处理异常的时候，程序就会自动停止并退出。可以把HostOptions.BackgroundServiceExceptionBehavior设置为Ignore，程序会忽略异常，而不是停止程序。不过推荐采用默认的设置，因为“异常应该被妥善的处理，而不是被忽略”。

2、要在ExecuteAsync方法中把代码用try……catch包裹起来，当发生异常的时候，记录日志中或发警报等。

### 托管服务中使用DI

1、托管服务是以单例的生命周期注册到依赖注入容器中的。因此不能注入生命周期为范围或者瞬态的服务。比如注入EF Core的上下文的话，程序就会抛出异常。

2、可以通过构造方法注入一个IServiceScopeFactory服务，它可以用来创建一个IServiceScope对象，这样我们就可以通过IServiceScope来创建短生命周期的服务了。记得在Dispose中释放IServiceScope。

示例：注入数据库上下文，延迟若干秒，读取user的数量

> 注：一定要释放资源

~~~C#
// 注入其他服务
	public class DemoBackgroundServer : BackgroundService
    {
        private readonly MyDBContext _db;// 也可以注入UserManager
        private readonly IServiceScope _scope;

        public DemoBackgroundServer(IServiceScopeFactory service) {
            _scope= service.CreateScope();// 创建可以进行依赖注入(Scope)的对象
            _db = _scope.ServiceProvider.GetRequiredService<MyDBContext>(); // 将MyDBContext以scope方式注入到类中

            //scope.Dispose(); 这里不能直接释放资源，释放后MyDBContext对象无法使用
        }

        // 停止时；需要释放资源
        public override void Dispose()
        {
            _scope.Dispose(); // IServiceScope的对象需要释放资源
            _db.Dispose();// 数据库上下文需要释放资源

            base.Dispose();
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            try
            {
                for (int i = 0; i < 5; i++)
                {
                    await Task.Delay(5000);
                    int userCount = _db.Users.Count();
                    Console.WriteLine(userCount);
                    Console.WriteLine("--------------------------------------------");
                }

            }
            catch (Exception ex)
            {

                Console.WriteLine(ex.Message);
            }
        }
    }
~~~

## FluentValidation

FluentValidation是对实体类进行校验规则的配置

用类似于EF Core中Fluent API的方式进行校验规则的配置，也就是我们可以把对模型类的校验放到单独的校验类中。

### FluentValidation的使用

1、NuGet：FluentValidation.AspNetCore

2、编写继承自AbstractValidator的数据校验类

3、在services中注册使用

~~~C#
// 实体类
	public class User
    {
        public string Email { get; set; }
        public string Name { get; set; }
        public string Password { get; set; }
        public string Password2 { get; set; }
    }
~~~

~~~C#
// 配置类
namespace WebApplication9
{
    public class UserValidation:AbstractValidator<User>// 指定配置的类是User
    {
        // 使用构造函数配置实体
        public UserValidation() {
            // 配置邮箱
            RuleFor(u => u.Email)
                // 设置不能为空
                .NotNull().WithMessage("邮箱不能为空")
                // 设置邮箱的结尾格式为@qq.com或@163.com
                .Must(e => e.EndsWith("@qq.com") || e.EndsWith("@163.com")).WithMessage("邮箱格式不正确");

            // 配置名称
            RuleFor(u => u.Name)
                .NotNull().WithMessage("名称不能为空")
                // 配置名称长度
                .Length(1, 4).WithMessage("名称长度要在1-4直接");

            // 配置密码1
            RuleFor(u => u.Password).NotNull();

            // 配置密码2
            // 使用Equal()，判断两次密码是否一样
            RuleFor(u => u.Password2).Equal(p => p.Password).WithMessage(u => $"{u.Password}于{u.Password2}不一致")
                .NotNull();
        }
    }
}
~~~

### FluentValidation注入其它服务

> 可以通过构造方法来向数据校验类中注入服务

~~~C#
public Login3RequestValidator(TestDbContext dbCtx)
	{
		RuleFor(x => x.UserName).NotNull()
			.Must(name => dbCtx.Users.Any(u => u.UserName == name))
			.WithMessage(c => $"用户名{c.UserName}不存在");
	}
~~~

