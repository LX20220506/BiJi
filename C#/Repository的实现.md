# 仓储接口的定义（IRepository）

1.IRepositoryBase接口：定义基础接口，适用于所有仓库的操作（增删改查），所有的实体类的仓储接口都要继承这个接口

~~~C#
using System;
using System.Linq;
using System.Linq.Expressions;

namespace Demo.Redis.IRepository
{
    public interface IRepositoryBase<T> where T :class
    {
        IQueryable<T> QueryAll();
        IQueryable<T> Find(Expression<Func<T,bool>> expression);

        void Create(T entity);
        
        void Edit(T entity);

        void Remove(T entity);
        
    }
}

~~~

2.IRepositoryWrapper接口：定义仓储的集合。所有的仓储定义在里面，这个接口就相当于所有接口的集合

~~~C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Demo.Redis.IRepository
{
    public interface IRepositoryWrapper
    {
        public IUserRepository UserRepository { get; }

        Task Save();
    }
}
~~~

3.IUserRepository接口：定义可以操作实体类的接口。实体类的仓储接口要实现

~~~c#
using Demo.Redis.Entity;

namespace Demo.Redis.IRepository
{
    public interface IUserRepository:IRepositoryBase<Account> 
    {
        Account FindAccountById(int id);
    }
}

~~~

# 实现仓储层（Repository）

1.实现IRepositoryBase接口，创建所有接口的基类，所有的实体类的仓储类都要继承这个类。

~~~C#
using Demo.Redis.EF;
using Demo.Redis.IRepository;
using Microsoft.EntityFrameworkCore;
using System;
using System.Linq;
using System.Linq.Expressions;

namespace Demo.Redis.Repository
{
    public class RepositoryBase<T> : IRepositoryBase<T> where T : class
    {
        private readonly RedisTestDbContext _dbContext;
		
        // 通过子类传入构造函数（注意：没有使用依赖注入）
        public RepositoryBase(RedisTestDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public  void Create(T entity)
        {
            _dbContext.Set<T>().Add(entity);
            //EntityEntry e1 = _dbContext.Entry(entity); // 查询状态

            //string ss = e1.DebugView.LongView;// 快照信息
        }

        public void Edit(T entity)
        {
            _dbContext.Set<T>().Update(entity);
        }

        public IQueryable<T> Find(Expression<Func<T, bool>> expression)
        {
            return _dbContext.Set<T>().Where(expression);
        }

        public IQueryable<T> QueryAll()
        {
            return _dbContext.Set<T>().AsNoTracking();
        }

        public void Remove(T entity)
        {
            _dbContext.Set<T>().Remove(entity);
        }
    }
}

~~~

2.实现IRepositoryWrapper接口，这个类相当于将所有的仓促接口都整合到这个类中，这样在使用仓储时，只要注入这个一个类，就可以使用全部的仓储接口了。

> 需要什么类型的仓储，就定义一个对应的私有变量和只读的属性

~~~C#
using Demo.Redis.EF;
using Demo.Redis.IRepository;
using System.Threading.Tasks;

namespace Demo.Redis.Repository
{
    // 一个对外的类，将所有的仓储接口全部放到这里，到时候只用注入这个类，那么所有的仓储接口都可以使用
    public class RepositoryWrapper : IRepositoryWrapper
    {
        private readonly RedisTestDbContext _dbContext;
        // 添加一个私有字段，每次访问的对象都是这个私有字段
        private IUserRepository _userRepository;

		// 添加只能读取公共IUserRepository属性，通过这个只读的属性来访问私有字段
        public IUserRepository UserRepository // 判断_userRepository是否初始化，若没有则进行初始化
        {
            get
            {
                return _userRepository ??= new UserRepository(_dbContext);// 将数据库上下文传给实体类的仓储
            }
        }
		
        // 这个地方的数据库上下文对象，是通过依赖注入添加进来的
        public RepositoryWrapper(RedisTestDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task Save()
        {

             await _dbContext.SaveChangesAsync();
        }
    }
}
~~~

3.定义实体类的仓促，需要继承RepositoryBase基础类和对应的接口

> 因为实体类的仓储接口继承IRepositoryBase接口，而RepositoryBase类又实现了IRepositoryBase接口，所以，实体的仓储不需要再实现IRepositoryBase接口（公共操作）了，只需要实现自己对应的实体仓储接口就行了

~~~C#
using Demo.Redis.EF;
using Demo.Redis.Entity;
using Demo.Redis.IRepository;
using System.Linq;

namespace Demo.Redis.Repository
{
    public class UserRepository:RepositoryBase<Account>,IUserRepository
    {
        private readonly RedisTestDbContext _dbContext;
		
        // 获取从RepositoryWrapper类中传入的数据库上下文，然后再传递给父类（RepositoryBase）的构造函数
        public UserRepository(RedisTestDbContext dbContext) : base(dbContext)
        {
            _dbContext = dbContext;
        }

        /// <summary>
        /// 通过id查找
        /// </summary>
        /// <param name="id"></param>
        /// <returns></returns>
        public Account FindAccountById(int id) {
            return _dbContext.Accounts.FirstOrDefault(a => a.Id == id);
        }
    }
}

~~~

# Repository的使用

只需要在Startup中将RepositoryWrapper注入到IOC容器中，之后所有的地方都能用了

~~~C#
services.AddScoped<IRepositoryWrapper, RepositoryWrapper>();
~~~

