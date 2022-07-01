# SignalR

ASP.NET Core SignalR 是一个开放源代码库，可用于简化向应用添加实时 Web 功能。 实时 Web 功能使服务器端代码能够将内容推送到客户端。

适合 SignalR 的候选项：

- 需要从服务器进行高频率更新的应用。 示例包括游戏、社交网络、投票、拍卖、地图和 GPS 应用。
- 仪表板和监视应用。 示例包括公司仪表板、即时销售更新或旅行警报。
- 协作应用。 协作应用的示例包括白板应用和团队会议软件。
- 需要通知的应用。 社交网络、电子邮件、聊天、游戏、旅行警报和很多其他应用都需使用通知。

## SignalR的基本使用

1、创建web API 项目

2、创建一个继承自Hub类

3、在服务中启用 .NTE6（builder.Services.AddSignalR() ）； .NET5（endpoints.MapHub<MyHub>("/SendMessage")）注：app.MapControllers()之前调app.MapHub<ChatRoomHub>("路由")。

4、启用CORS

5、编写前端项目。安装SignalR的JavaScript客户端SDK：npm install @microsoft/signalr

示例（基于Vue）：

~~~vue
<template>
  <div class="hello">
    <div>
      消息：<input type="text" v-model="state.publicMessage" />
      <button @click="onPublicMessageClick" >发送</button>
    </div>
    <ul>
      <li v-for="(message,index) in state.messageArray " :key="index">{{message}}</li>
    </ul>
  </div>
</template>

<script>

// 引用signalR组件
import * as signalR from '@microsoft/signalr';
// 引用reactive和onMounted组件
import {reactive,  onMounted } from '@vue/runtime-core';

// 定义 连接器 全局变量
let connection;
export default {
  name: 'HelloWorld',
  // 可以简单理解为程序入口
  setup(){
    // 定义变量 类似于vue2中的data
    // 注意：使用 reactive 将对象包裹，使数据变成响应式数据
    const state=reactive({
        messageArray:[],
        publicMessage:"",
      });

      // 给所有人发送消息的点击事件
      const onPublicMessageClick=()=>{
        // 第一个参数是需要调用的方法(后台)名称；需要和后台保持一致
        // 第二个参数是后台方法需要的参数(这里指送的消息)
        connection.invoke("SendPublicMessage",state.publicMessage);
      };

      // 就是生命周期中的Mounted钩子函数
      onMounted(async ()=>{
        // 实例化连接器
        connection = new signalR
          .HubConnectionBuilder()
          // 这里指的是要访问的后台路由(要写全);这个是在后台的Config方法中配置的（MapHub<MyHub>("/SendMessage");）
          .withUrl('http://localhost:5000/SendMessage')
          //.withAutomaticReconnect()
          .build();

        // 打开连接器
        await connection.start();
        
        // 连接器的回调函数；主要用于接受服务器返回的消息
        // 第一个参数是函数名称，要和后台保持一致；
        // 第二个参数是返回的参数；后台返回几个参数，这里就必须接受几个参数
        connection.on("ReceivePublicMessage",message=>{
          state.messageArray.push(message);
        })
      })

      // 将data 和 onClick事件返回
      return {state,onPublicMessageClick};
  },
}
</script>
<style scoped>

</style>

~~~

~~~C#
// Hub
 public class MyHub:Hub
    {
        // 发送公共消息
        public Task SendPublicMessage(string message)
        {
            // 拿到客户端的ConnectionId(注:在生产环境下，不能将ConnectionId暴露给客户端)
            string connId = this.Context.ConnectionId;
            string msg = $"{connId}：{message}";
            // 将消息发送给所有连接到这个服务上的客户端
            // 第一个参数是指前台连接器响应的方法名称；需要和前台保持一致
            // 第二个是发送个前台的数据(或者说是参数)；这里写几个，前台就要接受几个
            return Clients.All.SendAsync("ReceivePublicMessage", msg);
        }
    }
~~~

~~~C#
// 添加服务
    // 添加SignalR服务
    services.AddSignalR();
    
    // 需要配置Cors
    services.AddCors(opt=> {
    opt.AddPolicy("MyCors", policy =>
        {
            policy.AllowCredentials()
            .AllowAnyHeader()
            .AllowAnyMethod()
            .WithOrigins("http://localhost:8080");
        });
    });
            
// 注册服务
    // 添加Cors服务
    app.UseCors("MyCors");

	app.UseEndpoints(endpoints =>
                     {
                         // 配置前台访问后台服务器的路由
                         endpoints.MapHub<MyHub>("/SendMessage");// 要在Controllers() 之前添加
                         endpoints.MapControllers();
                     });
~~~

## **SignalR**的协议协商

### 协议协商

1、SignalR支持多种服务器推送方式：Websocket、Server-Sent Events、长轮询。默认按顺序尝试。

2、F12查看协商过程。

3、websocket和HTTP是不同的协议，为什么能用同一个端口。

4、在【开发人员工具】的【网络】页签中看WebSocket通信过程。

### 协议协商的问题

1、集群中协议协商的问题：“协商”请求被服务器A处理，而接下来的WebSocket请求却被服务器B处理。

2、解决方法：粘性会话和禁用协商。

3、 “粘性会话”（Sticky Session）：把来自同一个客户端的请求都转发给同一台服务器上。缺点：因为共享公网IP等造成请求无法被平均的分配到服务器集群；扩容的自适应性不强。

4、“禁用协商”：直接向服务器发出WebSocket请求。WebSocket连接一旦建立后，在客户端和服务器端直接就建立了持续的网络连接通道，在这个WebSocket连接中的后续往返WebSocket通信都是由同一台服务器来处理。缺点：无法降级到“服务器发送事件”或“长轮询”，不过不是大问题。

禁用协议协商的方式：

~~~js
// skipNegotiation:是否跳过协商;这个协商是前台和后台相互判断，看本次连接使用哪种连接方式
// transport:指定传输协议格式；这里使用的是WebSocket，表示使用该格式传输
const options = { skipNegotiation: true, transport: signalR.HttpTransportType.WebSockets  };

connection = new signalR
	.HubConnectionBuilder()
	.withUrl('https://localhost:7047/Hubs/ChatRoomHub', options) // 添加第二个参数
	//.withAutomaticReconnect()
	.build();

~~~

## **SignalR**的分布式部署

### SignalR的分布式问题

1、四个客户端被连接到不同的两个服务器上，会……

2、解决方案：所有服务器连接到同一个消息中间件。

3、官方方案：Redis backplane。

1）NuGet：Microsoft.AspNetCore.SignalR.StackExchangeRedis

2）

~~~c#
builder.Services.AddSignalR().AddStackExchangeRedis("127.0.0.1", options => {
        	options.Configuration.ChannelPrefix = "Test1_";
        });
~~~

## **SignalR**向部分客户端发消息

通过Clients对象的不同的方法或属性，来调用实现群发、私发、分组发送等

示例：实现聊天室私聊

~~~vue
// vue
<template>
  <div class="hello">
      <div>
          <h1>登录</h1>
          <p>账号：<input type="text" v-model="state.account.name"></p>
          <p>密码：<input type="password" v-model="state.account.pwd"></p>
          <p><input type="button" value="login" @click="onLogin"></p>
      </div>
    <div>
      消息：<input type="text" v-model="state.publicMessage" />
      <button @click="onPublicMessageClick" >群发</button>
    </div>
    <div>
        对<input type="text" v-model="state.privateName" >发送<input v-model="state.privateMessae" type="text">
        <button @click="onPrivateMessage" >私发</button>
    </div>
    <ul>
      <li v-for="(message,index) in state.messageArray " :key="index">{{message}}</li>
    </ul>
  </div>
</template>

<script>

// 引用signalR组件
import * as signalR from '@microsoft/signalr';
// 引用reactive和onMounted组件
import {reactive } from '@vue/runtime-core';
import axios from "axios";

// 定义 连接器 全局变量
let connection;
export default {
  name: 'HelloWorld',
  // 可以简单理解为程序入口
  setup(){
    // 定义变量 类似于vue2中的data
    // 注意：使用 reactive 将对象包裹，使数据变成响应式数据
    const state=reactive({
        messageArray:[],
        publicMessage:"",
        account:{
            name:"",
            pwd:""
        },
        token:"",
        privateName:"",
        privateMessae:""
      });
    
    // 初始化连接
    const signalRInit = async ()=>{
       if(connection==null){
           const transport = signalR.HttpTransportType.WebSockets;
        const options = { skipNegotiation: true, transport: transport };
        options.accessTokenFactory = () => state.token;


        connection = new signalR.HubConnectionBuilder()
            .withUrl('http://localhost:5000/SendMessage', options)
            .withAutomaticReconnect().build();
        try {
            await connection.start();
        } catch (err) {
            alert(err);
            return;
        } 

        // 注意:所有的SignlR响应事件都需要写在打开后的connection后面
        connection.on("ReceivePublicMessage",message=>{
          state.messageArray.push(message);
        })

        connection.on("ReceivePrivateMessage",message=>{
            console.log(message);
            state.messageArray.push(message);
        })

       }else{
           return;
       }
    }
    
      
      // 给所有人发送消息的点击事件
      const onPublicMessageClick= async ()=>{
        connection.invoke("SendPublicMessage",state.publicMessage);
      };

    // 登录
      var onLogin = async ()=>{
          await axios.post("http://localhost:5000/api/Account/Login",state.account).then(async res=>{
              state.token=res.data;
              await signalRInit();
          }).catch(()=>{
              console.log("login error");
          })
      }
    
    // 私发信息
      const onPrivateMessage=async ()=>{
          connection.invoke("SendPrivateMessage",state.privateMessae,state.privateName)
      }


      return { state, onPublicMessageClick, onLogin, onPrivateMessage };
  },
}
</script>
<style scoped>

</style>

~~~

~~~C#
// JWTHelper
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Security.Claims;
using Microsoft.AspNetCore.Identity;
using Microsoft.IdentityModel.Tokens;
using System.Text;
using System.IdentityModel.Tokens.Jwt;

namespace WebApplication10
{
    public static class JwtHelper
    {

        public static async Task<string> GetToken(UserManager<User> userManager, User user)
        {
            List<Claim> claims = new List<Claim>();
            claims.Add(new Claim(ClaimTypes.Name, user.UserName));
            var roleList = await userManager.GetRolesAsync(user);

            foreach (var item in roleList)
            {
                claims.Add(new Claim(ClaimTypes.Role, item));
            }
			
            // 这里需要ClaimTypes.NameIdentifier，因为我使用的SignalR的用户表示是ClaimTypes.NameIdentifier(默认)，所有如果不写这个，私发信息时会找不到指定用户
            claims.Add(new Claim(ClaimTypes.NameIdentifier, user.UserName));

            // 生成toke的key
            SymmetricSecurityKey key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("adksjhasduhaksjdbaiysgdh"));

            JwtSecurityToken token = new JwtSecurityToken(
                issuer: "server",
                audience: "client",
                claims: claims,
                notBefore: DateTime.Now,
                expires: DateTime.Now.AddDays(1),
                signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256)// 签名
                );

            // 生成token 
            var jwtToken = new JwtSecurityTokenHandler().WriteToken(token);

            return jwtToken;

        }
    }
}
~~~

Identity的实体和数据库

~~~C#
// User
using Microsoft.AspNetCore.Identity;

namespace WebApplication10
{
    public class User:IdentityUser<int>// int表示主键的数据类型
    {
    }
}


// Role
using Microsoft.AspNetCore.Identity;

namespace WebApplication10
{
    public class Role:IdentityRole<int>
    {
    }
}
~~~

~~~C#
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace WebApplication10
{
    public class PowerDBContext:IdentityDbContext<User,Role,int>// 指定和数据表相关的实体，和他们的主键类型
    {
        public PowerDBContext(DbContextOptions options) : base(options)
        {

        }
    }
}
~~~

~~~C#
// Hub
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace WebApplication10
{
    [Authorize]
    public class MyHub:Hub
    {
        private readonly UserManager<User> _userContext;

        public MyHub(UserManager<User> userContext)
        {
            _userContext = userContext;
        }

        // 发送公共消息
        public Task SendPublicMessage(string message)
        {
            // 拿到客户端的ConnectionId(注:在生产环境下，不能将ConnectionId暴露给客户端)
            string connId = this.Context.ConnectionId;
            string msg = $"{connId}：{message}";
            // 将消息发送给所有连接到这个服务上的客户端
            // 第一个参数是指前台连接器响应的方法名称；需要和前台保持一致
            // 第二个是发送个前台的数据(或者说是参数)；这里写几个，前台就要接受几个
            return Clients.All.SendAsync("ReceivePublicMessage", msg);
        }

        // 发送私信
        public async Task SendPrivateMessage(string message,string userName)
        {
            //User user = await _userContext.FindByNameAsync(userName);
            //await this.Clients.User(user.Id.ToString()).SendAsync("ReceivePrivateMessage", $"{user.UserName}：{message}");
            await this.Clients.User(userName).SendAsync("ReceivePrivateMessage",
                message);
        }
    }
}

~~~

service配置

~~~c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.OpenApi.Models;
using Microsoft.EntityFrameworkCore;
using Microsoft.AspNetCore.Identity;
using Microsoft.IdentityModel.Tokens;
using System.Text;
using System;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using System.Threading.Tasks;

namespace WebApplication10
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {

            services.AddControllers();
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "WebApplication10", Version = "v1" });

                c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
                {
                    In = ParameterLocation.Header,
                    Type = SecuritySchemeType.ApiKey,
                    Description = "直接在下框中输入Bearer {token}（注意两者之间是一个空格）",
                    Name = "Authorization",
                    BearerFormat = "JWT",
                    Scheme = "Bearer"
                });
                c.AddSecurityRequirement(new OpenApiSecurityRequirement
        {
          {
            new OpenApiSecurityScheme
            {
              Reference=new OpenApiReference
              {
                Type=ReferenceType.SecurityScheme,
                Id="Bearer"
              }
            },
            new string[] {}
          }
        });
            });

            // 需要配置Cors
            services.AddCors(opt=> {
                opt.AddPolicy("MyCors", policy =>
                {
                    policy
                    .AllowAnyHeader()
                    .AllowAnyMethod()
                    .WithOrigins("http://localhost:8080")
                    .AllowCredentials();
                });
            });

            // 添加SignalR服务
            services.AddSignalR();
            //services.AddSignalR().AddStackExchangeRedis("127.0.0.1",options=> {
            //    options.Configuration.ChannelPrefix = "SignalR_";
            //});

            // 配置数据库
            services.AddDbContext<PowerDBContext>(options=> {
                options.UseSqlServer(Configuration.GetConnectionString("MyDB"));
            });

            services.AddDataProtection();//保护数据

            // 添加identity服务
            //services.AddIdentity()
            //    .AddEntityFrameworkStores<PowerDBContext>()
            //    .AddRoleManager<Role>()
            //    .AddUserManager<User>();

            services.AddIdentityCore<User>(options =>
            {
                //还可以配置lock，例如登录5次失败后，10分钟后才能重新登录

                // 配置密码
                options.Password.RequireDigit = false;// 数字
                options.Password.RequiredLength = 6; // 长度
                options.Password.RequireLowercase = false;// 小写字母
                options.Password.RequireNonAlphanumeric = false;// 特殊符号
                options.Password.RequireUppercase = false;// 大写字母

                // 配置Token
                options.Tokens.PasswordResetTokenProvider = TokenOptions.DefaultEmailProvider; // 重置密码（生成简单密码）
                options.Tokens.EmailConfirmationTokenProvider = TokenOptions.DefaultEmailProvider; // 邮箱验证规则；邮箱验证码(不配置的话会生成一个很长的字符串)
            });

            var identityBuilder = new IdentityBuilder(typeof(User), typeof(Role), services)
                .AddEntityFrameworkStores<PowerDBContext>()
                .AddUserManager<UserManager<User>>()
                .AddRoleManager<RoleManager<Role>>()
                .AddDefaultTokenProviders();


            services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
             .AddJwtBearer(options =>
             {
                 options.TokenValidationParameters = new TokenValidationParameters
                 {
                     ValidateIssuerSigningKey = true,
                     IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("adksjhasduhaksjdbaiysgdh")),
                     ValidateIssuer = true,
                     ValidIssuer = "server",
                     ValidateAudience = true,
                     ValidAudience = "client",
                     ValidateLifetime = true,
                     ClockSkew = TimeSpan.FromMinutes(60)
                 };

                 options.Events = new JwtBearerEvents
                 {
                     OnMessageReceived = context =>
                     {
                         var accessToken = context.Request.Query["access_token"];
                         var path = context.HttpContext.Request.Path;
                         if (!string.IsNullOrEmpty(accessToken) &&
                             (path.StartsWithSegments("/SendMessage"))) context.Token = accessToken;
                         return Task.CompletedTask;
                     }
                 };

             });

        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                app.UseSwagger();
                app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "WebApplication10 v1"));
            }
           

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthentication();
            app.UseAuthorization();

            // 添加Cors服务
            app.UseCors("MyCors");

            app.UseEndpoints(endpoints =>
            {
                // 配置前台访问后台服务器的路由
                endpoints.MapHub<MyHub>("/SendMessage");// 要在Controllers() 之前添加
                endpoints.MapControllers();
            });
        }
    }
}

~~~



