# Identity

详细细节看微软文档

## Identity介绍

1、标识（Identity）框架：采用基于角色的访问控制（Role-Based Access Control，简称RBAC）策略，内置了对用户、角色等表的管理以及相关的接口，支持外部登录、2FA等。

2、标识框架使用EF Core对数据库进行操作，因此标识框架支持几乎所有数据库。

3、该框架会自动生成权限数据表，可以自行配置或扩展

## **Identity**框架使用

1、IdentityUser<TKey>、IdentityRole<TKey>，TKey代表主键的类型。我们一般编写继承自IdentityUser<TKey>、IdentityRole<TKey>等的自定义类，可以增加自定义属性。

2、NuGet安装Microsoft.AspNetCore.Identity.EntityFrameworkCore。

3、创建继承自IdentityDbContext的类

4、可以通过IdDbContext类来操作数据库，不过框架中提供了RoleManager、UserManager等类来简化对数据库的操作。

5、部分方法的返回值为Task<IdentityResult>类型，查看、讲解IdentityResult类型定义。

6、向依赖注入容器中注册标识框架相关的服务

~~~C#
// 创建实体类
	// 角色
    public class RoleManager:IdentityRole<int>// 这里的int表示主键的类型
    {
        // 可以对IdentityRole进行扩展
    }
    
    // 用户
    public class UserManager:IdentityUser<int>// 这里的int表示主键的类型
    {
        // 可以对IdentityUser进行扩展
        public DateTime CreateTime { get; set; }
    }
~~~

~~~C#
// 创建数据库类，用于连接实体，生成数据库

	// 对IdentityUser和IdentityRole进行扩展时
    // 需要指定IdentityUser和IdentityRole的子类，不然会使用默认的类，并且需要指定他们主键的类型(这里是int)
    public class MyDBContext:IdentityDbContext<UserManager, RoleManager, int>
    {
        public MyDBContext(DbContextOptions options) : base(options) { }
        
    }
~~~

~~~C#
// 服务配置

	services.AddDbContext<MyDBContext>(optios =>
            {
                optios.UseSqlServer("server=.;database=IdentityDemo1;uid=sa;pwd=Lx141238792.;");
            });

            services.AddDataProtection();// 添加数据保护；加密服务

            
            services.AddIdentityCore<UserManager>(options =>
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

            // 对Identity进行配置 
            var idBuilder = new IdentityBuilder(typeof(UserManager), typeof(RoleManager), services);
            idBuilder.AddEntityFrameworkStores<MyDBContext>() // 和实体类建立关系
                .AddDefaultTokenProviders() //添加默认令牌提供程序
                .AddRoleManager<RoleManager<RoleManager>>()// 角色管理器；可以直接对Role操作
                .AddUserManager<UserManager<UserManager>>();// 用户管理器；可以直接对User操作
~~~

~~~C#
// controller
	[ApiController]
    [Route("[controller]/[action]")]
    public class Test1Controller : ControllerBase
    {
        private readonly ILogger<Test1Controller> logger;
        private readonly RoleManager<Role> roleManager;// 注入RoleManager
        private readonly UserManager<User> userManager;// 注入UserManager
        
        public Test1Controller(ILogger<Test1Controller> logger,
            RoleManager<Role> roleManager, UserManager<User> userManager)
        {
            this.logger = logger;
            this.roleManager = roleManager;
            this.userManager = userManager;
        }
        
		[HttpPost]
		public async Task<ActionResult> CreateUserRole()// 创建角色和用户
		{
			bool roleExists = await roleManager.RoleExistsAsync("admin");
			if (!roleExists)// 判断是否存在 admin 角色
			{
                // 若不存在，创建角色
				Role role = new Role { Name="Admin"};
				var r = await roleManager.CreateAsync(role);
				if (!r.Succeeded)// 若创建失败，结束，并返回错误
				{
					return BadRequest(r.Errors);
				}
			}
            
            // 查询用户
			User user = await this.userManager.FindByNameAsync("yzk");
			if (user == null)
			{
                // 若用户不存在，则创建用户
				user = new User{UserName="yzk",Email="yangzhongke8@gmail.com",EmailConfirmed=true};
                // 创建用户并设置密码
				var r = await userManager.CreateAsync(user, "123456");
				if (!r.Succeeded)// 若创建失败，结束，并返回错误
				{
					return BadRequest(r.Errors);
				}
                // 将刚才创建的user用户，赋予admin角色
				r = await userManager.AddToRoleAsync(user, "admin");
				if (!r.Succeeded)// 若添加失败，结束，并返回错误
				{
					return BadRequest(r.Errors);
				}
			}
			return Ok();
		}

		[HttpPost]
		public async Task<ActionResult> Login(LoginRequest req)// 登录(LoginRequest是自定义的类，里面有登录账号和密码)
		{
			string userName = req.UserName;
			string password = req.Password;
            // 查找是否存在userName用户
			var user = await userManager.FindByNameAsync(userName);
			if (user == null)
			{
				return NotFound($"用户名不存在{userName}");
			}
            
            // LockedOut可以设置每次登录失败后，在用户的登录失败字段上的次数加一，当登录失败次数达到一定次数后，会禁止该用户多长时间内禁止登录；若用户中间有一次登录成功，则登录失败次数会清零。（类似于手机解锁）
            
            // 判断user的登录错误次数是否上线
			if (await userManager.IsLockedOutAsync(user))
			{
				return BadRequest("LockedOut");
			}
            // 判断用户的密码是否匹配(即登录是否成功)
			var success = await userManager.CheckPasswordAsync(user, password);
			if (success)
			{
				return Ok("Success");
			}
			else
			{
                // 不成功
				var r = await userManager.AccessFailedAsync(user);
				if (!r.Succeeded)
				{
					return BadRequest("AccessFailed failed");
				}
				return BadRequest("Failed");
			}
		}

        
        // 重置密码；模拟向邮箱发送验证码
		[HttpPost]
		public async Task<IActionResult> SendResetPasswordToken(
					SendResetPasswordTokenRequest req)
		{
			string email = req.Email;// 获取邮箱
            // 判断该邮箱是否存在用户的信息中
			var user = await userManager.FindByEmailAsync(email);
			if (user == null)
			{
				return NotFound($"邮箱不存在{email}");
			}
            // 生成token(验证码)
			string token = await userManager.GeneratePasswordResetTokenAsync(user);
			logger.LogInformation($"向邮箱{user.Email}发送Token={token}");// 发送
			return Ok();
		}
	}
~~~

