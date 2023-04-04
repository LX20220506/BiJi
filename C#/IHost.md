.NET Core中的Host是一个重要的概念，它提供了一个通用的应用程序启动、配置和服务管理机制。下面是一个详细的笔记：

~~~C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
using Microsoft.VisualBasic;

namespace ConsoleApp1
{
    public class Sample04
    {
        /// <summary>
        /// 温度
        /// </summary>
        public interface ITemperatureCollector
        {
            int Get();
        }

        /// <summary>
        /// 湿度
        /// </summary>
        public interface IHumidityCollector
        {
            int Get();
        }

        /// <summary>
        /// 空气质量
        /// </summary>
        public interface IAirQualityCollector
        {
            int Get();
        }


        public class Collector: ITemperatureCollector, IHumidityCollector, IAirQualityCollector
        {
            int ITemperatureCollector.Get()
            {
                var random = new Random();
                return random.Next(0, 100);
            }

            int IHumidityCollector.Get()
            {
                var random = new Random();
                return random.Next(0, 100);
            }

            int IAirQualityCollector.Get()
            {
                var random = new Random();
                return random.Next(0, 100);
            }
        }

        /// <summary>
        /// 配置类
        /// </summary>
        public class AirEnvironmentOptions
        {
            public long Interval { get; set; }
        }


        public class AirEnvironmentPublisher
        {
            // 日志模板
            private const string Template = "温度：{temperature, -10}" +
                                             "湿度：{humidity, -10}" +
                                             "空气质量：{airQuality, -10}" +
                                             "时间:{now}";
            private readonly Action<ILogger, int,int,int,string, Exception> _logAction;
            private readonly ILogger _logger;

            public AirEnvironmentPublisher(ILogger<AirEnvironmentPublisher> logger)
            {
                _logger = logger;

                // 创建一个可以根据 日志等级 和 日志模板 输出的委托
                _logAction = LoggerMessage.Define<int, int, int, string>(LogLevel.Information, 0, Template);
            }

            /// <summary>
            /// 输出配置信息
            /// </summary>
            /// <param name="temp">温度</param>
            /// <param name="humi">湿度</param>
            /// <param name="airq">空气质量</param>
            public void Publish(int temp, int humi, int airq)
            {
                // 输出日志
                _logAction(_logger, temp, humi, airq, DateTime.Now.ToLongTimeString(), null);
            }
        }

        public class AirEnvironmentService : IHostedService // 使用IHostedService接口，创建host
        {
            private readonly ITemperatureCollector _temperatureCollector;
            private readonly IHumidityCollector _humidityCollector;
            private readonly IAirQualityCollector _airQualityCollector;
            private readonly AirEnvironmentPublisher _publisher;
            private readonly AirEnvironmentOptions _options;

            private Timer _timer;

            public AirEnvironmentService(
                ITemperatureCollector temperatureCollector,
                IHumidityCollector humidityCollector,
                IAirQualityCollector airQualityCollector,
                AirEnvironmentPublisher publisher,
                IOptions<AirEnvironmentOptions> options // 注入配置选项类
            )
            {
                _temperatureCollector = temperatureCollector;
                _humidityCollector = humidityCollector;
                _airQualityCollector = airQualityCollector;
                _publisher = publisher;
                _options = options.Value; // 获取配置
            }


            /// <summary>
            /// 来自IHostedService接口，host启动时，会执行该方法
            /// </summary>
            /// <param name="cancellationToken"></param>
            /// <returns></returns>
            public Task StartAsync(CancellationToken cancellationToken)
            {
                // 计时器
                _timer = new Timer(state =>
                {
                    // 打印信息
                    _publisher.Publish(
                        _temperatureCollector.Get(),
                        _humidityCollector.Get(), 
                        _airQualityCollector.Get());
                }, null,0, _options.Interval);

                return Task.CompletedTask;
            }


            /// <summary>
            /// 来自IHostedService接口，host结束时，会执行此方法
            /// </summary>
            /// <param name="cancellationToken"></param>
            /// <returns></returns>
            public Task StopAsync(CancellationToken cancellationToken)
            {
                _timer?.Dispose();
                return Task.CompletedTask;
            }
        }

        public static void Start()
        {
            var collector = new Collector();
            var host = new HostBuilder() // HostBuilder用于构建host主键
                .ConfigureAppConfiguration((context, builder) => // 配置应用程序的配置
                    builder.AddJsonFile("appsettings.json") // 配置json文件
                ) 
                .ConfigureServices((context, collection) => collection //配置应用程序服务
                    .AddSingleton<ITemperatureCollector>(collector)
                    .AddSingleton<IHumidityCollector>(collector)
                    .AddSingleton<IAirQualityCollector>(collector)
                    .AddSingleton<AirEnvironmentPublisher>()
                    .AddHostedService<AirEnvironmentService>() // 添加host服务
                    .AddOptions() // 添加配置选项服务
                    .Configure<AirEnvironmentOptions>( // 配置选项，将配置映射为AirEnvironmentOptions类
                        context.Configuration.GetSection("AirEnvironment"))
                )
                .ConfigureLogging(builder => builder.AddConsole()) // 配置日志信息
                .Build(); // 构建host

            // 运行host
            host.Run();
        }
    }
}

~~~



1. 什么是Host

Host 是一个通用的应用程序启动、配置和服务管理机制，可以用来构建 Web 应用程序、控制台应用程序、Azure 函数等等。

1. Host 的类型

.NET Core 提供了以下 3 种 Host 类型：

- Generic Host：通用的 Host，可以用于大多数应用程序场景，例如控制台应用程序、后台服务等等。
- Web Host：Web 应用程序的 Host，使用 Kestrel 作为 Web 服务器。
- Azure Functions Host：Azure Functions 的 Host。

1. Generic Host 的使用

Generic Host 可以用于大多数应用程序场景，包括控制台应用程序、后台服务等等。

使用步骤如下：

1）创建一个 Console Application 项目；

2）添加 NuGet 包 Microsoft.Extensions.Hosting；

3）在 Program.cs 文件中编写以下代码：

```c#
using Microsoft.Extensions.Hosting;

class Program
{
    static async Task Main(string[] args)
    {
        await CreateHostBuilder(args).RunConsoleAsync();
    }

    static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
                // 注册服务
            });
}
```

4）在 ConfigureServices 方法中，注册需要使用的服务。例如：

```c#
services.AddHttpClient();
services.AddSingleton<MyService>();
```

5）在 RunConsoleAsync 方法中，编写应用程序的主逻辑。

1. HostBuilder

HostBuilder 是创建 Host 的核心类，用于配置 Host 的行为。通常，我们使用 HostBuilder 来配置应用程序的服务集合、配置文件、日志、环境变量等等。

使用 HostBuilder 的步骤如下：

1）创建一个 HostBuilder 对象；

2）通过调用对象上的方法来配置 Host 的行为，例如 UseEnvironment、ConfigureAppConfiguration、ConfigureLogging 等等；

3）调用 Build 方法来创建 Host 对象。

示例代码：

```C#
using Microsoft.Extensions.Hosting;

class Program
{
    static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .UseEnvironment(EnvironmentName.Production)
            .ConfigureAppConfiguration((hostContext, config) =>
            {
                // 配置应用程序的配置
            })
            .ConfigureLogging(logging =>
            {
                // 配置日志
            })
            .Build();

        await host.RunAsync();
    }
}
```

1. 主机生命周期事件

Host 提供了一些生命周期事件，可以在应用程序启动、停止等时刻执行需要的操作。例如，可以在应用程序启动时初始化数据库连接池，在应用程序关闭时释放资源等等。

常见的生命周期事件如下：

- HostStarting：Host 开始启动时触发。
- HostStarted：Host 启动完成时触发。
- HostStopping：Host 开始停止时触发。
- HostStopped：Host 停止完成时触发。

使用步骤如下：

1）实现 IHostedService 接口，并在 StartAsync 和 StopAsync 方法中编写需要执行的操作；

2）在 Host 中注册 IHostedService 类型的服务。

示例代码：

```C#
class MyService : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        // 初始化操作
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        // 释放资源操作
    }
}

class Program
{
    static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .ConfigureServices(services =>
            {
                services.AddHostedService<MyService>();
            })
            .Build();

        await host.RunAsync();
    }
}
```

1. 总结

Host 是一个通用的应用程序启动、配置和服务管理机制，可以用于构建各种类型的应用程序。在应用程序中，我们可以使用 HostBuilder 来配置 Host 的行为，使用 IHostedService 接口来编写应用程序的生命周期