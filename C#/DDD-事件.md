# DDD-事件

1、DDD中的事件分为两种类型：领域事件（Domain Events）和集成事件（Integration Events）。

2、领域事件：在同一个微服务内的聚合之间的事件传递。使用进程内的通信机制完成。

3、集成事件：跨微服务的事件传递。使用事件总线（EventBus）实现。

# 领域事件

1、实现方式1：C#的事件机制。

var bl = new ProcessBusinessLogic();

bl.ProcessCompleted += bl_ProcessCompleted;

bl.StartProcess();

缺点：需要显式地注册。

2、实现方式2：进程内消息传递的开源库MediatR。事件的发布和事件的处理之间解耦。MediatR中支持“一个发布者对应一个处理者”和**“一个发布者对应多个处理者”**这两种模式

# **用**MediatR**实现领域事件**

可以理解为：当执行一个方法或者事件时，需要执行其他无关业务逻辑的代码，例如发送短信，邮件等，可以让其他方法或事件进行独立的操作。类似于触发器，当执行某个方法时，会发送消息给另一个事件，让它做其他的事情

> 进程内消息传递的开源库MediatR。事件的发布和事件的处理之间解耦。MediatR中支持“一个发布者对应一个处理者”和“一个发布者对应多个处理者”这两种模式

- Publish()：一个发布者对应多个处理者
- Send()：一个发布者对应一个处理者

## MediatR基本用法

1、创建一个ASP.NET Core项目，NuGet安装MediatR.Extensions.Microsoft.DependencyInjection

2、Program.cs中调用AddMediatR()

3、定义一个在消息的发布者和处理者之间进行数据传递的类，这个类需要实现INotification接口。一般用record类型。

4、消息的处理者要实现NotificationHandler<TNotification>接口，其中的泛型参数TNotification代表此消息处理者要处理的消息类型。

5、在需要发布消息的的类中注入IMediator类型的服务，然后我们调用Publish方法来发布消息。Send()方法是用来发布一对一消息的，而Publish()方法是用来发布一对多消息的。

示例：获取数据时，发布消息

~~~C#
// INotification的实现类  是消息传输过程中消息的载体 用于承载消息
using MediatR;

namespace WebApplication12
{
    public class Message:INotification
    {
        public string Content { get; set; }
    }
}

~~~

~~~C#
// 创建消息处理者1
using MediatR;
using System;

namespace WebApplication12
{
    public class TextNoification_1 : NotificationHandler<Message>
    {
        protected override void Handle(Message notification)
        {
            Console.WriteLine("接接收的信息1："+ notification.Content);
        }
    }
}



// 创建消息处理者2
using MediatR;
using System;

namespace WebApplication12
{
    public class TextNoification_2 : NotificationHandler<Message>
    {
        protected override void Handle(Message notification)
        {
            Console.WriteLine("接接收的信息2："+ notification.Content);
        }
    }
}
~~~

~~~C#
// service配置
    // 添加MediatR
    // 需要传递一个程序集参数  扫描该程序集下实现INotification接口的类 Assembly.GetExecutingAssembly()指当程序集
    services.AddMediatR(Assembly.GetExecutingAssembly());
~~~

~~~C#
// 发布者
using MediatR;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Linq;

namespace WebApplication12.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        private readonly ILogger<WeatherForecastController> _logger;

        // 需要注入IMediator类型的对象 用于发送消息
        private readonly IMediator _mediator;

        public WeatherForecastController(ILogger<WeatherForecastController> logger, IMediator mediator)
        {
            _logger = logger;
            _mediator = mediator;
        }

        [HttpGet]
        public IEnumerable<WeatherForecast> Get()
        {
            var rng = new Random();

            // 发送消息 （一个发布者对应多个处理者）
            _mediator.Publish(new Message {Content="获取数据" });

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = Summaries[rng.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }
}

~~~

## **EF Core**中发布领域事件的时机

1、在聚合根的实体对象的ChangeName()、构造方法等方法中立即发布领域事件，因为无论是应用服务还是领域服务，最终要调用聚合根中的方法来操作聚合，我们这样做可以确保领域事件不会被漏掉。

2、缺点：

1）存在重复发送领域事件的情况；

2）领域事件发布的太早：在实体类的构造方法中发布领域事件，但是有可能因为数据验证没通过等原因，我们最终没有把这个新增的实体保存到数据库中，我们这样在构造方法中过早地发布领域事件就会导致“误报” 。

### 实现

把领域事件的发布延迟到上下文保存修改时。实体中只是注册要发布的领域事件，然后在上下文 的SaveChanges方法被调用时，我们再发布事件。

~~~C#
// 实体
namespace WebApplication13
{
    // 继承领域接口
    public class User: BaseEvents
    {
        public int Id { get; set; }
        public string  Name { get; set; }
        public string  Pwd { get; private set; }


        // 给EF Core用的无参私有构造函数
        private User() { 
        
        }

        public User(string name,string pwd) {
            this.Name = name;
            this.Pwd = pwd;
            // 加入要发送消息的事件集合
            base.AddDomainEvent(new Message {Content="create user" });
        }

        // 修改密码
        public string UpdatePwd(string oldPwd,string newPwd) {
            if (!oldPwd.Equals(this.Pwd))
            {
                return "oldPwd error";
            }
            this.Pwd = newPwd;
            // 加入要发送消息的事件集合
            base.AddDomainEvent(new Message {Content="修改pwd" });
            return "ok";
        }
    }
}

~~~

~~~C#
// 定义接口IDomainEvents 用于控制领域事件

using MediatR;
using System.Collections.Generic;

namespace WebApplication13
{
    // 定义领域接口
    public interface IDomainEvents
    {
        // 获取事件
        IEnumerable<INotification> GetMessages();
        // 添加事件
        void AddDomainEvent(INotification notification);
        // 清除事件
        void ClearDomainEvents();
    }
}

~~~

~~~C#
// INotification的实现类  是消息传输过程中消息的载体 用于承载消息
using MediatR;

namespace WebApplication13
{
    public class Message:INotification
    {
        public string  Content { get; set; }
    }
}
~~~

~~~C#
using MediatR;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations.Schema;

namespace WebApplication13
{
    // 实现领域接口
    public class BaseEvents : IDomainEvents
    {
        [NotMapped]
        private List<INotification> _notifications = new List<INotification>();
       
        // 添加领域事件
        public void AddDomainEvent(INotification notification)
        {
            _notifications.Add(notification);
        }

        // 删除领域事件
        public void ClearDomainEvents()
        {
            _notifications.Clear();
        }

        // 返回事件集合
        public IEnumerable<INotification> GetMessages()
        {
            return _notifications;
        }
    }
}
~~~

~~~C#
// 配置
services.AddDbContext<MyDBContext>(options=> {
                options.UseSqlServer("server=.;database=Test;uid=sa;pwd=Lx141238792.;");
            });

// 添加MediatR
services.AddMediatR(Assembly.GetExecutingAssembly());// 查找本地程序集
~~~

~~~C#
// 数据库上下文
using Microsoft.EntityFrameworkCore;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using MediatR;

namespace WebApplication13
{
    public class MyDBContext:DbContext
    {
        private readonly IMediator _mediator;
        public MyDBContext(DbContextOptions options, IMediator mediator) : base(options)
        {
            _mediator = mediator;
        }

        public override Task<int> SaveChangesAsync(bool acceptAllChangesOnSuccess, CancellationToken cancellationToken = default)
        {
            // ChangeTracker是DBcontext跟踪的对象
            // Entries<T> 指定跟踪实体的类型

            // 获取所有存在发布事件的对象
            var DomainEntitys =this.ChangeTracker.Entries<IDomainEvents>().Where(e => e.Entity.GetMessages().Any()).ToList();

            // 获取DomainEntitys中所有的待发布事件
            var DomainEvents = DomainEntitys.SelectMany(e => e.Entity.GetMessages()).ToList();

            // 删除事件集合
            DomainEntitys.ToList().ForEach(e => e.Entity.ClearDomainEvents());

            foreach (var item in DomainEvents)
            {
                // 将消息发布延迟到执行数据操作之前 这样可以保证数据操作执行成功后，才会发送消息
                _mediator.Publish(item);
            }

            // SaveChangesAsync（）具有强一致性
            return base.SaveChangesAsync(acceptAllChangesOnSuccess, cancellationToken);
        }

        public DbSet<User> Users { get; set; }
    }
}

~~~

~~~C#
// 实现
[Route("api/[controller]/[action]")]
    [ApiController]
    public class UserController : ControllerBase
    {
        private readonly MyDBContext _myDBContext;

        public UserController(MyDBContext myDBContext)
        {
            _myDBContext = myDBContext;
        }

        [HttpPost]
        public async Task<string> CreateUser(string name,string pwd) {
            User user = new User(name,pwd);

            await _myDBContext.Users.AddAsync(user);

            if (await _myDBContext.SaveChangesAsync()>0)
            {
                return "ok";
            }
            throw new Exception("add error");
        }

        [HttpPost]
        public async Task<string> AlterPwd(int id,string oldpwd,string newpwd) {
            User user = _myDBContext.Users.SingleOrDefault(u=>u.Id==id);
            user.UpdatePwd(oldpwd, newpwd);
            if (await _myDBContext.SaveChangesAsync()>0)
            {
                return "ok";
            }
            throw new Exception("update error");
        }
    }
~~~



