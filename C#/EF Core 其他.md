# EF Core

## 联合查询

使用 Include() 方法， 该方法在using Microsoft.EntityFrameworkCore;命名空间下

~~~C#
// 联合查询 老师和学生表
var list = await db.Teacher.Include(s => s.Students).ToListAsync();
~~~

## 特性说明

- 主键：`Key`

- 自增：`DatabaseGenerated( DatabaseGeneratedOption.Identity)`

- not null ：`Required`

- 自定义类型：`Column(TypeName ="nvarchar(20)")`

- 最大值：`MaxLength(200)`   MaxLength特性指定了属性的值所允许的最大值，然后在数据库中就生成相应列的最大值

- 不映射字段：`NotMapped`  不需要映射成数据库中的列的时候，可以使用特性。（默认情况下，EF为实体的每个属性映射数据列。   【必须包含get；和set;】。NotMapped特性重写了这个约定。EF不会为没有get;和set;的属性，创建列）

- 设置索引：`Index(string name, Properties:[IsClustered = bool],[IsUnique= bool])`  

  ​					用来在特定的列上面创建索引，默认情况下，索引的名称是IX_{属性的名称}。

  ​					name：索引名称

  ​					IsClustered` `: 是否是建聚合索引

  ​					IsUnique：是否是唯一索引

- 外键：` [ForeignKey(name string)]`  name：主表的列名

  ~~~ C#
  
  public class Student
  {
      public int StudentID { get; set; }
      public string StudentName { get; set; }
          
      [ForeignKey("TeacherId")]
      public Teacher teacher{ get; set; }
  }
   
  public class Teacher
  {
      public int TeacherId { get; set; }
      public string TeacherName { get; set; }
  }
  ~~~


## Fluent API

### 使用 Fluent API 配置模型

> 可在派生上下文中替代 `OnModelCreating` 方法，并使用 `ModelBuilder API` 来配置模型。 此配置方法最为有效，并可在不修改实体类的情况下指定配置。 Fluent API 配置具有最高优先级，并将替代约定和数据注释。

- 首先创建一个实体类的` Config`类，用于配合该实体；
- 将实体类的配置类继承 IEntityTypeConfiguration<T> 接口，并实现接口成员
- 使用 ` Configure`方法 ` EntityTypeBuilder<T> builde`对象配置 T 的实体约束
- ` Fluent API`的详情查看官网 https://docs.microsoft.com/zh-cn/ef/core/modeling/

示例：

~~~c#
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace EFCore
{
    // 实体
     public class Student
    {
        public int Id { get; set; }

        /// <summary>
        /// 名称
        /// </summary>
        public string  StudentName { get; set; }
        /// <summary>
        /// 生日
        /// </summary>
        public DateTime? Birthday { get; set; }
        /// <summary>
        /// 成绩
        /// </summary>
       // public List<Success> Success { get; set; } = new List<Success>();

    }
    
    // 实体配置类
    public class SuccessConfig : IEntityTypeConfiguration<Success>
    {
        public void Configure(EntityTypeBuilder<Success> builder)
        {
            builder.ToTable("Success");//创建数据表
            builder.Property(s => s.Score).IsRequired();// 配置字段的约束
            //builder.Property(s=>s.StudentId).HasOne<Student>()

            // 生成外键关系
            builder.HasOne(s => s.Student)
                    .WithMany()
                    .HasForeignKey(s=>s.StudentId);
        }
    }
    
}
~~~

​		注：实际开发中，实体和配置类是分开的，这里只是举例说明



在**数据库上下文**中实现配置

~~~C#
		// 重写 OnModelCreating 方法，配置实体的映射
        protected override void OnModelCreating(ModelBuilder modelBuilder) {
            // 调用父类方法
            base.OnModelCreating(modelBuilder);
            // 应用程序集中的配置(实现 FluentAPI 的配置)
            modelBuilder.ApplyConfigurationsFromAssembly(this.GetType().Assembly);
        }
~~~

### Fluent API数据校验

~~~C#
// user实体
namespace WebApplication9
{
    public class User
    {
        public string Email { get; set; }
        public string Name { get; set; }
        public string Password { get; set; }
        public string Password2 { get; set; }
    }
}
~~~

~~~C#
// fluent api Validation
namespace WebApplication9
{
    public class UserValidation:AbstractValidator<User>
    {
        public UserValidation() {
            RuleFor(u => u.Email)
                .NotNull().WithMessage("邮箱不能为空")
                .Must(e => e.EndsWith("@qq.com") || e.EndsWith("@163.com")).WithMessage("邮箱格式不正确");

            RuleFor(u => u.Name)
                .NotNull().WithMessage("名称不能为空")
                .Length(1, 4).WithMessage("名称长度要在1-4直接");

            RuleFor(u => u.Password).NotNull();

            RuleFor(u => u.Password2).Equal(p => p.Password).WithMessage(u => $"{u.Password}于{u.Password2}不一致")
                .NotNull();
        }
    }
}
~~~

~~~C#
// 配置 service
services.AddFluentValidation(fx=> {
                Assembly assembly = Assembly.Load("WebApplication9");
                fx.RegisterValidatorsFromAssembly(assembly); // 加载指定程序集中的 验证配置
                // fx.RegisterValidatorsFromAssemblies()  加载多个程序集
            });
~~~



## 主键

### 自增

> 在数据库中，我们通常将主键设置为自增

- 优点：简单
- 缺点：(1.)在高并发插入数据时，性能低；因为自增有锁的机制，他只会一条一条的插入，保证插入的Id等于上一条的id加一。(2.)当有多个数据库需要合并时，会导致id重复

### Guid

> Guid 是他自己生成的 id 由小写数字和小写字母组成的 32位字符串，是一个不会重复的主键

- 优点：可以支持高并发；当多个数据库合并时，不会出现id重复的问题
- 缺点：内存较大，消耗磁盘内存；不能设置为聚集索引(没有排序)；查询速度慢

> 注：MySQL 中的IndexDB不能使用Guid，因为该数据库强制要求主键为聚集索引

~~~C#
Guid guid = Guid.NewGuid();
~~~

### Hi/Lo算法

​	HiLo是High Low的简写，翻译成中文叫高低位模式。

​	HiLo是由“Hi”和“Lo”两部分生成主键的一种模式。“Hi”部分来自数据库，“Lo”部分在内存中生成以创建唯一值。请记住，“Lo”是一个范围数字，如0-100。因此，当“Hi”部分用完“Lo”范围时，再次进行数据库调用以获得下一个“Hi数字”。所以HiLo模式的优点在于您预先可以知道主键的值，而不用每次都与数库据发生交互。

​	总结有以下四点：

​		“Hi”部分由数据库分配，两个并发请求保证得到唯一的连续值；

​		一旦获取“Hi”部分，我们还需要知道“incrementSize”的值(“Lo”条目的数量)；

​		“Lo”取的范围：[0,incrementSize];

​		标识范围的公式是：(Hi - 1) * incrementSize) + 1 到 (Hi - 1) * incrementSize) + incrementSize)

​		当所有“Lo”值使用完时，需要重新从数据库中取出一个新的“Hi”值，并将“Lo”部分重置为0。

在这里演示在两个并发事务中的例子，每个事务插入多个实体：
![](E:\筆記\img\EF Core\2.png)



## 外键

Fluent API中的配置

~~~C#
    // 生成外键关系
    builder.HasOne(s => s.Student)
        .WithMany()
        .HasForeignKey(s=>s.StudentId);
~~~

## 对应关系

EF Core中的对应关系主要通过以下几个方法进行配置（以下方法也可以没有泛型）：

- HasOne<T>()：一个对几个
- HasMany<T>()：多个对几个
- WithOne<T>()：几个对一个
- WithMany<T>()：几个对多个

这四个方法实现了一对一，一对多，多对多的复杂关系映射

### 一对多

- 一对多使用的是HasOne<T>()和WithMany()
- 一对多的配置一般是在“多”的一方配置
- 在“一”的一方设置一个属性集合List
- 在“多”的一方设置一个“一”的对象

示例（文章和评论）：一张文章有多条评论

~~~C#
// 实体
	/// <summary>
    /// 文章
    /// </summary>
    public class Article
    {
        public int Id { get; set; }
        /// <summary>
        /// 标题
        /// </summary>
        public string Title { get; set; }
        /// <summary>
        /// 内容
        /// </summary>
        public string Text { get; set; }
        /// <summary>
        /// 评论
        /// </summary>     
        public List<Comment> Comment { get; set; } = new List<Comment>();
    }



	/// <summary>
    /// 评论
    /// </summary>
    public class Comment
    {
        public int Id { get; set; }
        /// <summary>
        /// 评论内容
        /// </summary>
        public string Context { get; set; }
        /// <summary>
        /// 文章id(后来加的) 这里有点问题，如果一开始生成数据库的话，会生成两个ArticleId，这个属性是我生成数据库之后添加的，生成数据库之后添加数据库不会再生成
        /// </summary>
        //public int ArticleId { get; set; }
        /// <summary>
        /// 文章
        /// </summary>
        public Article Article { get; set; }
    }

	
~~~



~~~C#
// Fluent API配置
	
	//Article 文章配置
	public class ArticleConfig : IEntityTypeConfiguration<Article>
    {
        public void Configure(EntityTypeBuilder<Article> builder)
        {
            builder.ToTable("Article");
            builder.Property(a => a.Title).HasMaxLength(50).IsRequired();
            builder.Property(a => a.Text);
        }
    }


	//comment 评论配置
	public class CommentConfig : IEntityTypeConfiguration<Comment>
    {
        public void Configure(EntityTypeBuilder<Comment> builder)
        {
            builder.ToTable("Comment");
            builder.Property(c => c.Context).IsRequired();
			//配置关系：Comment的Article 对应 Article的Comment
            builder.HasOne<Article>(a => a.Article).WithMany(c=>c.Comment);
        }
    }
~~~

注：这个示例中“一”中包含了多个评论List<Comment> Comment，这个是根据实际情况设置的；例如：员工--离职单，	请假单，采购单......的关系，要是还像示例中写，“一”中会包含很多“多”，这是不可取的，所以具体情况，具体分析



### 一对一

- 一对一使用的是HasOne()和WithOne()
- 一对一的配置在任何一方都可以
- 一对一关系在两侧都有引用导航属性
- 它们遵循与一对多关系相同的约定，但在外键属性上引入了一个唯一索引，以确保只有一个依赖项与每个主体相关

示例（订单和快递单）：一个订单对应一个快递单号

~~~C#
//实体

	/// <summary>
    /// 订单
    /// </summary>
    public class Order
    {
        public int Id { get; set; }
        /// <summary>
        /// 商品名称
        /// </summary>
        public string GoodsName { get; set; }
        /// <summary>
        /// 快递的导航属性
        /// </summary>
        public Express Express { get; set; }

    }
    
    
    /// <summary>
    /// 快递
    /// </summary>
    public class Express
    {
        public int Id { get; set; }
        /// <summary>
        /// 快递名称
        /// </summary>
        public string ExpressName { get; set; }
        /// <summary>
        /// 订单Id---用于设置和订单表的外键
        /// </summary>
        public int OrderId { get; set; }
        /// <summary>
        /// 订单导航属性
        /// </summary>
        public Order Order { get; set; }
    }
~~~

~~~C#
// Fluent API配置

	//订单
	public class OrderConfig : IEntityTypeConfiguration<Order>
    {
        public void Configure(EntityTypeBuilder<Order> builder)
        {
            builder.ToTable("Order");
            builder.Property(o => o.GoodsName).HasMaxLength(50).IsRequired();
        }
    }
    
    //快递
    public class ExpressConfig : IEntityTypeConfiguration<Express>
    {
        public void Configure(EntityTypeBuilder<Express> builder)
        {
            builder.ToTable("Express");
            builder.Property(e => e.ExpressName).HasMaxLength(20).IsRequired();
            // 一对一关系配置  注意外键的配置 和 泛型的使用
            builder.HasOne<Order>(e => e.Order).WithOne(o => o.Express).HasForeignKey<Express>(e => e.OrderId);
        }
    
~~~

### 多对多

- 多对多使用的是HasMany()和WithMany()
- 多对多的配置在任何一方都可以
- 多对多关系在两侧都需要设置集合的导航属性
- 多对多关系需要第三张表来帮我们绑定他们之间的关系，这张表EF Core会帮我们自动生成；若是直接操作数据库，需要自己建表
- 需要使用 UsingEntity() 方法，具体使用看示例（不使用的话EF会为我们自动创建表）

示例（老师和学生）：多个老师对应多个学生

~~~C#
// 实体
 	public class Student
    {
        public int Id { get; set; }
        public string StudentName { get; set; }
        // 都需要list确认关系
        public List<Teacher> Teachers { get; set; } = new List<Teacher>();
    }

	
	public class Teacher
    {
        public int Id { get; set; }
        public string TeacherName { get; set; }
        // 都需要list确认关系
        public List<Student> Students { get; set; } = new List<Student>();
    }
~~~

~~~C#
// Fluent API配置

	// 学生FluentAPI配置
	public class StudentConfig : IEntityTypeConfiguration<Student>
    {
        public void Configure(EntityTypeBuilder<Student> builder)
        {
            builder.ToTable("Student");
            builder.Property(s => s.StudentName).HasMaxLength(10).IsRequired();

            // 配置多对多关系 
            // UsingEntity（j => j.ToTable("Student_Teacher")） 相当于又创建了一个表，
            // 是用于连接 student 和 teacher 的关系的表，使用的是联合主键（student表的Id和teacher表的Id）
            builder.HasMany(s => s.Teachers).WithMany(t => t.Students).UsingEntity(j => j.ToTable("Student_Teacher"));
        }
    }


	// 老师FluentAPI配置
	 public class TeacherConfig : IEntityTypeConfiguration<Teacher>
    {
        public void Configure(EntityTypeBuilder<Teacher> builder)
        {
            builder.ToTable("Teacher");
            builder.Property(t => t.TeacherName).HasMaxLength(10).IsRequired();
        }
    }
	
~~~



### 自连接

- 自连接是在一个类中实现的
- 自连接使用的是HasOne()和WithMany()
- 自连接关系需要定义一个自己类型的属性(Parent)和List类型的类型(Children);就是需要一个自身的父节点和自己的若干个子节点
- 不需要主外键关系

示例（地球上国家地区的分类）：地球是根节点，下面有各个洲，各个洲有各个国家

~~~C#
// 实体
	/// <summary>
    /// 地区类  地球-->州-->国家
    /// </summary>
    // 自连接的关系：一个节点有且只有一个父节点，单有多个子节点；根节点只有子节点
    public class Address
    {
        public int Id { get; set; }
        public string Name { get; set; }
        /// <summary>
        /// 父节点
        /// </summary>
        public Address Parent { get; set; }
        /// <summary>
        /// 子节点
        /// </summary>

        public List<Address> Children { get; set; } = new List<Address>();

        //public bool IsDelete { get; set; }
    }
~~~

~~~C#
// Fluent API配置
	public class AddressConfig : IEntityTypeConfiguration<Address>
    {
        public void Configure(EntityTypeBuilder<Address> builder)
        {
            builder.ToTable("Address");
            builder.Property(a => a.Name).HasMaxLength(50).IsRequired();
            // 注意这里是一对多的关系
            builder.HasOne(a => a.Parent).WithMany(a=>a.Children);

            // 软删除 全局过滤
           // builder.HasQueryFilter(a => a.IsDelete == false);
        }
    }
~~~



## EF Core的数据数据加载

具体问题具体分析 

### IEnumerable<T> 

> 公开枚举器，该枚举器支持在指定类型的集合上进行简单迭代。也就是说：实现了此接口的object，就可以直接使用foreach遍历此object；
>
>  IEnumerable 和 IEnumerable<T> 接口在 .NET 中是非常重要的接口，它允许开发人员定义foreach语句功能的实现并支持非泛型方法的简单的迭代，IEnumerable和IEnumerable<T>接口是 .NET Framework 中最基本的集合访问器。它定义了一组扩展方法，用来对数据集合中的元素进行遍历、过滤、排序、搜索等操作。
>
> IEnumerable接口是非常的简单，只包含一个抽象的方法Get IEnumerator(T)，它返回一个可用于循环访问集合的IEnumerator对象。

EF Core中的IEnumerable

- **查询全部到内存**。当一个EF中Linq语句返回的类型为IEnumerable<T>类型时，EF不会在数据库中过滤筛选条件，而是直接select * from 表 拿到所有数据放到内存中进行筛选，他的底层操作是ADO.NET中断开式连接，一次性将拿到所有数据。
- **复杂方法过滤时，推荐使用IEnumerable<T>接口。**当遇见复杂的筛选过滤时，数据库压力过大，可以使用该方法，返回IEnumerable<T>类型的Linq表达式中是可以使用方法进行过滤的。具体问题具体分析 

`注意：只有遍历的时候，才会执行IEnumerable的方法`

###  IQueryable <T>

> 它继承 IEnumerable接口，而因为.net版本加入Linq和IQueryable后，使得IEnumerable不再那么单调，变得更加强大和丰富。

- **延迟加载机制**。当返回为IQueryable 接口类型时，不会立即查询，只有碰到遍历或者遇见终结方法时才会执行，例如 tolist，toarray，count，max，min......一个方法的返回值为IQueryable时，就不是终结方法 (简单理解)
- **条件筛选后进行查询。**因为IQueryable 接口并不会直接的从数据库中查询数据，所有我们可以通过条件对Linq进行二次(多次)拼接，已达到符合我们的查询需求。**当满足需求后执行遍历或者终结方法获取我们所需要的数据。**例如：多条件的筛选查询等......
- **持续连接数据库。**该方式底层是ADO.NET中的连接式访问，在查询过程中会议在连接数据库；这样会节省系统内存，但是如果处理慢会占用数据库连接，使用tolist，toarray等方法直接将数据加载到内存中

### IEnumerable和IQueryable 的区别

- 所有对于IEnumerable的过滤，排序等操作，都是在内存中发生的。也就是说数据已经从数据库中获取到了内存中，只是在内存中进行过滤和排序操作。
- 所有对于IQueryable的过滤，排序等操作，只有在数据真正用到的时候才会到数据库中查询。





## EF执行SQL语句

> FormattableString  对象是用于参数化插值的，一般使用  $ 符号表示；
>
> $ 参数化插值
>
> 原理：他会将字符串中要插入的参数的位置，放一个占位符，将插入的数据转换成参数，填到指定的占位符；
>
> 作用：当我们拼接SQL语句时，使用 $ 插值的方式，可以防止SQL注入
>
> 注意：$ 不是字符串拼接

EF执行SQL语句的三种方式：


```C#

        //-------1.执行非查询语句操作
        db.Database.ExecuteSqlInterpolated($"sql语句");// 执行非查询语句操作      

        //-------2.执行查询操作
        db.Address.FromSqlInterpolated($"sql语句").ToList(); // 执行查询操作 基于dbset执行（非立即执行，需要终结方法）           


        //-------3.使用ADO.NET   太复杂，不推荐，可以使用 Dapper
        DbConnection conn= db.Database.GetDbConnection();// 拿到context对应的底层connection
		// 判断连接是否打开
        if (conn.State!=System.Data.ConnectionState.Open)
        {
            conn.Open();// 打开连接
        }
		
		// 创建Command对象
        using (var comm=conn.CreateCommand())
        {
            comm.CommandText = "sql语句";
            using (var reader=comm.ExecuteReader())// 持续连接式访问
            {
                while (reader.Read())// 读取数据
                {
                    int id = reader.GetInt32(0);// 拿到第一列的数据转换成int类型
                    string name = reader.GetString(1);// 拿到第二列的数据转换成string类型
                    Console.WriteLine($"{id}:{name}");
                }
            }
        }
```

## ef的快照更改跟踪(占内存)

> 一个实体对象只要和数据库上下文中有相关的操作，EF都会对其进行跟踪
>
> 首次跟踪一个实体的时候，EF Core会创建这个实体的快照，执行SaveChanges()等方法时，EF Core将会把存储的快照中的值于实体的当前值进行比较

### 实体的状态

SaveChanges()识别为什么样的状态，就对数据库执行什么样的操作

实体的五种状态

- 已添加  Add：SaveChanges()识别为 插入	
- 未改变 Unchanged：SaveChanges()识别为 忽略	
- 已修改 Modified：SaveChanges()识别为 修改
- 已删除 Deleted：SaveChanges()识别为 删除
- 分离 Detached ：SaveChanges()识别为 忽略

### 实体状态操作

~~~C#
EntityEntry e1 = db.Entry(address); // 查询状态

string ss=e1.DebugView.LongView;// 快照信息

//AsNoTracking()  禁用跟踪  可以不让EF Core 跟踪他的改变；在只有显示数据的时候可以使用，降低内存的占用
db.Address.AsNoTracking().ToList();
~~~

### 小技巧

通过修改实体的状态，可直接SaveChanges()对数据库进行操作，不需要查询

```C#
    Address aa = new Address { Id = 36, Name = "日本" };
    var ee = db.Entry(aa);
    ee.Property("Name").IsModified = true;
    db.SaveChanges();
```



## 软删除

> 给表设置一个特殊字段 ，通过该字段的状态来进行软删除，
>
> 删除表的数据时，只需要将特殊字段的状态改为false，对数据查询时筛选一下就行了
>
> 软删除有可能会影响性能，根据需要可以创建联合索引

### 全局过滤器

- HasQueryFilter()：配置全局过滤，所有生成的查询语句后面都会自动添加该方法的条件
- IgnoreQueryFilters()：屏蔽全局过滤，使用时会将全局过滤屏蔽

~~~C#
// 软删除 全局过滤 在Fluent API中配置 
builder.HasQueryFilter(a => a.IsDelete == false);// 这样每个生成的sql查询的后面都会跟一个IsDelete=false 的条件判断，通过判断该字段来判断是否展示该数据，从而实现软删除



// 忽略全局过滤器 
db.Address.IgnoreQueryFilters().ToList();// 当使用时会将全局过滤屏蔽


~~~

## 数据库的并发操作

### 悲观并发控制

> 使用悲观锁，将正在操作的数据锁住，只允许“我”来访问，当“我”访问结束解锁后；其他用户才能访问。
>
> 开启悲观锁 ：将 `with(updlock)`  放到表后面 ；例如  `select * from 表 WITH(UPDLOCK) where id=1`
>
> 优点：简单
>
> 缺点：不适用于高并发场景；操作时间过长时，容易发生阻塞；使用不当，会造成死锁
>
> 注：尽量使用最小锁的级别（锁的级别：行锁，表锁，页锁）

示例（租房子）：当一个用户抢到后，其他用户就不能强了

~~~c#
static void Main(string[] args)
        {

            Console.WriteLine("请输入租客名称：");
            string name = Console.ReadLine();

            using (MyDBContext db=new MyDBContext())
            {
                using (var tran=db.Database.BeginTransaction())// 开启事务
                {
                    // with(updlock) 开启悲观锁 
                    var room = db.Room.FromSqlInterpolated($"select * from room WITH(UPDLOCK) where id=1").SingleOrDefault();

                    Thread.Sleep(5000);// 等待5秒

                    if (room.UserName==null)
                    {
                        Console.WriteLine($"恭喜您，抢房成功");
                        room.UserName = name;
                    }
                    else
                    {
                        
                        Console.WriteLine($"抱歉，房间已被{room.UserName}抢到");
                    }
                    db.SaveChanges(); // 关闭悲观锁

                    tran.Commit();// 提交事务
                }
            }

            Console.ReadKey();
        }
~~~

### 乐观并发控制

> 使用乐观控制并发；有两种方式（并发令牌）：控制单个字段的 新、旧 值(Owner)；控制整行的版本

#### Owner

> `控制单个字段的 新、旧 值`：通过判定行中的指定字段的 旧值 是否和“我”认为的 旧值 是否相等，若不相等，说明存在并发操作，且修改报错并返回错误（DbUpdateConcurrencyException ex）

- Owner字段并发操作就是修改判断操作；并发操作时，就会对并发字段进行筛选，如果没有找的符合筛选条件的数据，说明该字段已被修改(并发操作)，那么数据库将不会做任何操作(sql执行返回行数为0)，这时EF 会报DbUpdateConcurrencyException的错误，通过try-actch 来对报错进行处理就可以了。
- 语法(就是sql的修改语法)：update 表 set Owner='新' where id=1 and Owner=旧  ---并发修改令牌
- Fluent API配置：需要设置并发字段为 IsConcurrencyToken()；
- 通过报错的`DbUpdateConcurrencyException` 对象，可以拿到现在数据库中新的行数据(对象)

注：Owner只是别名

~~~C#
// Fluent API配置
builder.Property(r => r.Owner).IsConcurrencyToken();// 设置为并发令牌，适用于单个字段
~~~

~~~C#
static void Main(string[] args)
        {

            Console.WriteLine("请输入租客名称：");
            string name = Console.ReadLine();

            using (MyDBContext db=new MyDBContext())
            {
                using (var tran=db.Database.BeginTransaction())// 开启事务
                {
                    var room = db.Room.FromSqlInterpolated($"select * from room where id=1").SingleOrDefault();
                    
                    Thread.Sleep(5000);// 等待5秒

                    if (room.UserName==null)
                    {
                        Console.WriteLine($"恭喜您，抢房成功");
                        room.UserName = name;
                    }
                    else
                    {
                        
                        Console.WriteLine($"抱歉，房间已被{room.UserName}抢到");
                    }
					//------------乐观并发控制重点-------
                    try
                    {
                        db.SaveChanges();
                    }
                    catch (DbUpdateConcurrencyException ex)
                    {

                        var entry = ex.Entries.First();
                        var dbValues = entry.GetDatabaseValues();
                        // 可以拿到数据中最新的room对象类型的RoomName属性值(导致“我”失败的原因)
                        string newValue = dbValues.GetValue<string>(nameof(room.RoomName));
                    }
                    //-----------------------

                    tran.Commit();// 提交事务
                }
            }

            Console.ReadKey();
        }
~~~



#### RowVersion

> 控制整行的版本：通过判断行的“版本号”，来判断该行是否存在修改；若“版本号”不同，说明存在并发操作，并且对该行的操作失败，返回错误（DbUpdateConcurrencyException ex）
>
> `timestamp`类型：SQL Server 中的数据类型字段，该字段为16进制，每一次对数据进行操作时，该类型的字段就会自动改变，且该字段不会重复，一般会被当做 `行的版本号` 来控制数据并发操作

- 给存在并发的表添加一个`timestamp`类型的字段，该字段会被当做该行的版本号(并发令牌)
- 只要对行中的任何字段进行操作(修改)时，都会对`timestamp`类型的字段进行改变(自动改变)
- 操作数据时，会判断该行的版本号是否变化，若变化说明存在并发，数据库不会操作（数据库影响行数为0）；EF会返回错误`DbUpdateConcurrencyException` 对象，可以拿到现在数据库中新的行数据(对象)
- Models配置：将设置版本属性的类型设置为`byte[]`类型
- Fluent API配置：使用`IsRowVersion()`对版本属性进行设置
- 注：低版本MySQL精度不够， 可以使用guid 但是每次都需要手动更新guid

~~~C#
//Model
 public class Room
    {
        public int Id { get; set; }
        /// <summary>
        /// 房间名称
        /// </summary>
        public string RoomName { get; set; }
        /// <summary>
        /// 拥有该房间的用户名称
        /// </summary>
        public string UserName { get; set; }
        //public string Owner { get; set; } //设置并发字段 通过单个字段判断并发操作
        public byte[] RowVersion { get; set; }// 指定的行版本号  行中任何字段改变，该值都会改变
    }
~~~

~~~C#
// Fluent API配
public class RoomConfig : IEntityTypeConfiguration<Room>
    {
        public void Configure(EntityTypeBuilder<Room> builder)
        {
            builder.ToTable("Room");
            builder.Property(r => r.RoomName).HasMaxLength(20);
            builder.Property(r => r.UserName).HasMaxLength(20);
            //builder.Property(r => r.Owner).IsConcurrencyToken();// 设置为并发令牌，适用于单个字段
            builder.Property(r => r.RowVersion).IsRowVersion();//  对这个一行做任何改变，他的值就会改变，适合做并发令牌，适用于修改多个字段
            // 低版本mysql 精度不够， 可以使用guid 但是每次都需要手动更新guid
        }
    }
~~~

~~~C#
// 实现
class Program
    {
        static void Main(string[] args)
        {

            Console.WriteLine("请输入租客名称：");
            string name = Console.ReadLine();

            using (MyDBContext db=new MyDBContext())
            {
                using (var tran=db.Database.BeginTransaction())// 开启事务
                {
                    var room = db.Room.FromSqlInterpolated($"select * from room where id=1").SingleOrDefault();
                    
                    Thread.Sleep(5000);// 等待5秒

                    if (room.UserName==null)
                    {
                        Console.WriteLine($"恭喜您，抢房成功");
                        room.UserName = name;
                    }
                    else
                    {
                        
                        Console.WriteLine($"抱歉，房间已被{room.UserName}抢到");
                    }

                    try
                    {
                        db.SaveChanges();
                    }
                    catch (DbUpdateConcurrencyException ex)
                    {

                        var entry = ex.Entries.First();
                        var dbValues = entry.GetDatabaseValues();
                        // 可以拿到数据中最新的room对象类型的RoomName属性值(导致“我”失败的原因)
                        string newValue = dbValues.GetValue<string>(nameof(room.RoomName));
                    }

                    tran.Commit();// 提交事务
                }
            }

            Console.ReadKey();
        }
    }
~~~



### 总结

1. 乐观并发控制能够避免悲观锁带来的性能、死锁等问题，因此推荐使用乐观并发控制而不是悲观锁
2. 如果有一个确定的字段要被并发控制，那么使用IsConcurrencyToken()把这个字段设置为并发令牌即可
3. 如果无法确定唯一的并发令牌列，那么就可以引入一个额外的属性设置并发令牌，并且在每一次更新数据的时候，手动跟新这一列的值。若果用的是SQL Server数据库，那么也可以采用RowVersion列，这样就不用开发者手动来在每次更新数据的时候，手动更新并发令牌的值了。



充血模型   第一个是属性  第二个是回调（配置属性用的）

## 表达式树
