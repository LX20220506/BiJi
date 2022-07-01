# EF Core

[TOC]



## 创建Model类

安装 Microsoft.EntityFrameworkCore.Design 包

~~~C#
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace MvcMovie.Models
{
    public class Movie
    {
        public int Id { get; set; }
        public string? Title { get; set; }

        [Display(Name = "Release Date")]
        [DataType(DataType.Date)]
        public DateTime ReleaseDate { get; set; }
        public string? Genre { get; set; }

        [Column(TypeName = "decimal(18, 2)")]
        public decimal Price { get; set; }
        public string? Rating {  get; set; }
    }
}
~~~



## 数据库连接字符串

在配置文件(appsettings.json )中添加数据库连接字符串

~~~json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
    //数据库连接字符串
  "ConnectionStrings": {
    "MvcMovieContext": "Server=(localdb)\\mssqllocaldb;Database=MvcMovieContext1;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
~~~



## 创建数据库上下文类（依赖注入）

安装  Microsoft.EntityFrameworkCore.SqlServer 和 Microsoft.EntityFrameworkCore.Tools 包

创建一个数据库上下文类，将可以访问的数据表(数据模型)添加进去

~~~c#
using Microsoft.EntityFrameworkCore;
using MvcMovie.Models;

namespace MvcMovie.Data
{
    public class MvcMovieContext : DbContext
    {
        public MvcMovieContext (DbContextOptions<MvcMovieContext> options): base(options)
        {
        }

        public DbSet<Movie> Movie { get; set; }//Movie表示需要访问的数据表(数据模型)
    }
}
~~~

在服务(Startup类)中注入

~~~C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();

    //注入数据库上下文对象
    services.AddDbContext<MvcMovieContext>(options =>
    options.UseSqlServer(Configuration.GetConnectionString("MvcMovieContext")));
}
~~~

## 创建数据库上下文类（普通实现）

~~~c#
namespace Demo
{
    public class DemoDBContext : DbContext
    {
        public DbSet<Article> Articles { get; set; }
        public DbSet<Comment> Comments { get; set; }

        //重写 OnConfiguring 方法，设置连接字符串
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) {
            // 设置连接字符串
            optionsBuilder.UseSqlServer("Server=.;Database=Demo;uid=sa;pwd=Lx141238792.;");
            // 控制台输出Log
            optionsBuilder.LogTo(Console.WriteLine);
        }

        // 重写 OnModelCreating 方法，配置实体的映射
        protected override void OnModelCreating(ModelBuilder modelBuilder) {
            // 调用父类方法
            base.OnModelCreating(modelBuilder);
            // 应用程序集中的配置(实现 FluentAPI 的配置)
            modelBuilder.ApplyConfigurationsFromAssembly(this.GetType().Assembly);
        }
    }
}
~~~



## 创建数据库

使用 EF Code First 自动创建数据库时，Code First 将：

- 将表添加到数据库，以跟踪数据库的架构。
- 验证数据库与生成它的模型类是否同步。 如果它们不同步，EF 则会引发异常。 这使查找不一致的数据库/代码问题变得更加轻松。

~~~c#
// 如果是首次迁移，会生成的类包含用于创建数据库架构的代码。 数据库架构基于在数据库上下文类中指定的模型
Add-Migration InitialCreate
Update-Database
~~~



## 数据库迁移

 添加NuGet 包 Microsoft.EntityFrameworkCore.Design

~~~C#
//添加初始迁移 使用初始迁移来更新数据库。
Add-Migration InitialCreate
Update-Database
//Add-Migration InitialCreate：生成 Migrations/{timestamp}_InitialCreate.cs 迁移文件。
//InitialCreate 参数是迁移名称。 可以使用任何名称，但是按照惯例，会选择可说明迁移的名称。 如果是首次迁移，会生成的类包含用于创建数据库架构的代码。 数据库架构基于在数据库上下文类中指定的模型
//Update-Database：将数据库更新到上一个命令创建的最新迁移。 此命令在用于创建数据库的 Migrations/{time-stamp}_InitialCreate.cs 文件中运行 Up 方法。   

//添加字段的迁移
Add-Migration Rating
Update-Database
//Add-Migration 命令会通知迁移框架使用当前 Movie DB 架构检查当前 Movie 模型，并创建必要的代码，将 DB 迁移到新模型。
//名称“Rating”是任意的，用于对迁移文件进行命名。 为迁移文件使用有意义的名称是有帮助的。

// 多个数据库迁移    
Add-Migration InitialCreate -Context BlogContext -OutputDir Migrations\SqlServerMigrations
Add-Migration InitialCreate -Context SqliteBlogContext -OutputDir Migrations\SqliteMigrations

//将数据库更新为指定的迁移
//第一个示例使用迁移名称，第二个示例使用迁移 ID 和指定的连接：
Update-Database InitialCreate
Update-Database 20180904195021_InitialCreate -Connection your_connection_string

//(删除有点问题，只能删除最新添加的迁移；不能删除指定迁移)
//删除上次迁移（回退为迁移所做的代码更改）  删除之前要执行 Update-Database [InitialCreate] 命令
Remove-Migration
    
// 生成SQL脚本
Update-Database

// 生成指定版本A 到 版本B 的SQL脚本 (不包括A)
Update-Database A版本 B版本
    
// 生成最新SQL脚本C
Update-Database C版本
~~~

## Entity Framework Core 的 SQL 日志记录

~~~json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyDB-2;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
        //这里是添加SQL日志记录
     ,"Microsoft.EntityFrameworkCore.Database.Command": "Information"
    }
  },
  "AllowedHosts": "*"
}
~~~



## 设定数据库种子

使用以下代码在 Models 文件夹中创建一个名为 `SeedData` 的新类：

~~~C#
using Microsoft.EntityFrameworkCore;
using RazorPagesMovie.Data;

namespace RazorPagesMovie.Models
{
    public static class SeedData
    {
        public static void Initialize(IServiceProvider serviceProvider)
        {
            //数据库上下文对象
            using (var context = new RazorPagesMovieContext(serviceProvider.GetRequiredService<DbContextOptions<RazorPagesMovieContext>>()))
            {
                //没有对应的数据库或数据表
                if (context == null || context.Movie == null)
                {
                    throw new ArgumentNullException("Null RazorPagesMovieContext");
                }
				
                #region 看需求；感觉用不到
				  //看movie表中是否有数据
                    if (context.Movie.Any())
                    {
                        return;   //进来说明有数据，则结束方法
                    }
            	#endregion
                
                //在movie表中添加种子数据
                context.Movie.AddRange(
                    new Movie
                    {
                        Title = "When Harry Met Sally",
                        ReleaseDate = DateTime.Parse("1989-2-12"),
                        Genre = "Romantic Comedy",
                        Price = 7.99M
                    },

                    new Movie
                    {
                        Title = "Ghostbusters ",
                        ReleaseDate = DateTime.Parse("1984-3-13"),
                        Genre = "Comedy",
                        Price = 8.99M
                    },

                    new Movie
                    {
                        Title = "Ghostbusters 2",
                        ReleaseDate = DateTime.Parse("1986-2-23"),
                        Genre = "Comedy",
                        Price = 9.99M
                    },

                    new Movie
                    {
                        Title = "Rio Bravo",
                        ReleaseDate = DateTime.Parse("1959-4-15"),
                        Genre = "Western",
                        Price = 3.99M
                    }
                );
                context.SaveChanges();
            }
        }
    }
}
~~~

## 分层的EF Core（Code First）

步骤汇总：

1、建类库项目，放实体类、DbContext、配置类等

DbContext中不配置数据库连接，而是为DbContext增加一个DbContextOptions类型的构造函数。

2、EFCore项目安装对应数据库的EFCore Provider

3、asp.net core项目引用EFCore项目，并且通过AddDbContext来注入DbContext及对DbContext进行配置。

4、Controller中就可以注入DbContext类使用了。

5、让开发环境的Add-Migration知道连接哪个数据库

在EFCore项目中创建一个实现了`IDesignTimeDbContextFactory`的类。

并且在CreateDbContext返回一个连接开发数据库的DbContext。

```C#
 public MyDbContext CreateDbContext(string[] args)
    {
        // 创建 数据库上下文配置 生成器
        DbContextOptionsBuilder<MyDbContext> builder = new DbContextOptionsBuilder<MyDbContext>();
        
        string connStr = "Data Source=.;Initial Catalog=demo666;Integrated Security=SSPI;";
        builder.UseSqlServer(connStr);
        
        // 创建数据库上下文
        MyDbContext ctx = new MyDbContext(builder.Options);
        return ctx;
    }
```

如果不在乎连接字符串被上传到Git，就可以把连接字符串直接写死到CreateDbContext；如果在乎，那么CreateDbContext里面很难读取到VS中通过简单的方法设置的环境变量，所以必须把连接字符串配置到Windows的正式的环境变量中，然后再 Environment.GetEnvironmentVariable读取。

6、正常执行Add-Migration、Update-Database迁移就行了。需要把EFCore项目设置为启动项目，并且在【程序包管理器控制台】中也要选中EFCore项目，并且安装Microsoft.EntityFrameworkCore.SqlServer、Microsoft.EntityFrameworkCore.Tools

示例：

EF Core类库：

需要引用 

1. Microsoft.EntityFrameworkCore.Relational ；
2. Microsoft.EntityFrameworkCore.SqlServer；
3. Microsoft.EntityFrameworkCore.Tools

~~~C#
// 模型
public class Book
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public decimal Price { get; set; }
    }
~~~

~~~C#
// 实体配置
public class BookConfig : IEntityTypeConfiguration<Book>
    {
        public void Configure(EntityTypeBuilder<Book> builder)
        {
            builder.ToTable("Book");
        }
    }
~~~

~~~C#
// 数据库上下文
public class BookDBContext:DbContext
    {
        public DbSet<Book> Books { get; set; }
        public BookDBContext(DbContextOptions options) : base(options) { }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);
            modelBuilder.ApplyConfigurationsFromAssembly(this.GetType().Assembly);
        }
    }
~~~

~~~ C#
// 创建 数据库上下文类
public class BookDBContextDesignFactory : IDesignTimeDbContextFactory<BookDBContext>
    {
        public BookDBContext CreateDbContext(string[] args)
        {
            DbContextOptionsBuilder<BookDBContext> builder = new DbContextOptionsBuilder<BookDBContext>();
            builder.UseSqlServer("server=.;database=BookDB;uid=sa;pwd=Lx141238792.;");
            BookDBContext bookDB = new BookDBContext(builder.Options);
            return bookDB;
        }
    }
// 注意：该类只在开发环境下才会使用，生产环境下不会使用；该类用于创建数据库
~~~

API

需要引用 

1. EF Core 的类库项目

~~~C#
// Startup 的 ConfigureServices()方法
 #region 添加数据库服务
    services.AddDbContext<BookDBContext>(option =>
    {
    option.UseSqlServer(Configuration.GetConnectionString("Book"));
    });
#endregion
~~~



## 多个数据库上下文

当数据库的表特别多少时，若将这些表的实体全都放在一个数据库上下文中的话，在创建数据库上下文时会全部执行一遍，会影响性能；所以当数据库表很多时，可以创建多个数据库上下文对象（例如一个数据库上下文中只存在2-5个实体），这样可以有效的提高数据库上下文创建的性能
