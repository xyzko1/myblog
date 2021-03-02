---
title: "ASP.NET MVC+EF 项目架构搭建"
catalog: true
date: 2018-02-01 14:00:10
subtitle: ""
header-img: "Demo.png"
tags:
- Hexo
- Blog
catagories:
- Hexo
---
> created by [xyzko1](https://github.com/xyzko1/xyzko1.github.io) 
> 2018年02月01日 14:00:10
# 正文
---
新建MVC项目UI
![avatar](1.png)

---
然后分别建立类库，Model,IDAL,DALFactory,DAL,IBLL,BLL,Common
![avatar](2.png)

---
Model里面添加EF实体 User生成数据库
![avatar](3.png)

Model里面添加EF实体 User生成数据库
![avatar](4.png)

---
IDAL层
IBasedal.cs
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace IDAL
{
   public  interface IBaseDal<T>
    {
        /// <summary>
        /// 
        /// </summary>
        /// <param name="whereLambda"></param>
        /// <returns></returns>
        IQueryable<T> LoadEntities(System.Linq.Expressions.Expression<Func<T, bool>> whereLambda);
        /// <summary>
        /// 
        /// </summary>
        /// <typeparam name="s"></typeparam>
        /// <param name="pageIndex"></param>
        /// <param name="pageSize"></param>
        /// <param name="totalCount"></param>
        /// <param name="wherelambda"></param>
        /// <param name="orderByLambda"></param>
        /// <param name="isasc"></param>
        /// <returns></returns>
        IQueryable<T> LoadPageEntities<s>(int pageIndex, int pageSize, out int totalCount, System.Linq.Expressions.Expression<Func<T, bool>> wherelambda, System.Linq.Expressions.Expression<Func<T, s>> orderByLambda, bool isasc);
        bool DeleteEntity(T entity);
        bool EditEntity(T entity);
        T AddEntity(T entity);
    }
}
```

IUserDal.cs实现IBaseDal接口：
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Model;
namespace IDAL
{
   public  interface IUserDal:IBaseDal<User>
    {
    }
}
IDbSession.cs接口(回话层接口DALFactory)
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace IDAL
{
    public interface  IDbSession
    {
        DbContext db { get; }
        IUserDal userdal { get; set; }
        bool SaveChanges();
    }
}
```
IDbSession.cs接口(回话层接口DALFactory)
```
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace IDAL
{
    public interface  IDbSession
    {
        DbContext db { get; }
        IUserDal userdal { get; set; }
        bool SaveChanges();
    }
}
```
![avatar](5.png)

DALFactory(数据回话层)
DBSession.cs实现IDBSession

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using IDAL;
using DAL;
namespace DALFactory
{
    public class DBSession : IDbSession
    {
        public System.Data.Entity.DbContext db
        {
            get {
                return DBContextFactory.CReateDbContext();
            }
        }
        private IUserDal _UserDAl;
       public IUserDal userdal { get =>AbstractFactorycs.createIUserDal(); set => _UserDAl=value; }
 
        public bool SaveChanges()
        {
            return db.SaveChanges() > 0;
        }
    }
}
```
DBSessionFactory.cs DBsession的工厂
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.Remoting.Messaging;
using System.Text;
using System.Threading.Tasks;
using IDAL;
namespace DALFactory
{
    public   class DBSessionFactory
    {
        public static IDbSession CreateDBSession()
        {
            //面向接口编程 返回接口
            IDbSession   dbsession =(IDbSession) CallContext.GetData("dbsession");
            if (dbsession == null)
            {
               dbsession = new DBSession();
                CallContext.SetData("dbsession", dbsession);
            }
            return dbsession;
        }
    }
}
```
AbstractFactory.cs(DAL的抽象工厂 利用反射创建DAL)

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Configuration;
using System.Reflection;
using IDAL;
using DAL;
namespace DALFactory
{
    public class AbstractFactorycs
    {
        public static readonly string AssemblyPath = ConfigurationManager.AppSettings["AssemblyPath"];
        public static readonly string NameSpace = ConfigurationManager.AppSettings["NameSpace"];
        public static IUserDal createIUserDal()
        {
            string fullclassname = NameSpace + ".UserDal";
            return (IUserDal)CreateInstace(fullclassname);
        }
        private static object CreateInstace(string classname)
        {
            var assembly = Assembly.Load(AssemblyPath);//这里加载的是程序集名称 如DAL
            return assembly.CreateInstance(classname);//这里需要填写 命名空间.类名
        }
    }
}
```
![avatar](6.png)

DAL层
DBContextFactory.cs (EF Context线程内唯一对象)
```
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Runtime.Remoting.Messaging;
using System.Text;
using System.Threading.Tasks;
using Model;
namespace DAL
{
    public  class DBContextFactory
    {
        public static DbContext CReateDbContext()
        {
 
            DbContext dbcontext = (DbContext)CallContext.GetData("dbContext");
            if (dbcontext == null)
            {
                dbcontext = new MYMVCOAContainer();
                CallContext.SetData("dbContext", dbcontext);
            }
            return dbcontext;
        }
    }
}
```
UserDal(具体的dal类，实现IUserDAL接口)
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Model;
using IDAL;
using System.Linq.Expressions;
using System.Data.Entity;
namespace DAL
{
    public class UserDal :IUserDal
    {
        MYMVCOAContainer db =(MYMVCOAContainer) DBContextFactory.CReateDbContext();
        public User AddEntity(User entity)
        {
            db.UserSet.Add(entity);
            db.SaveChanges();
            return entity;
        }
 
        public bool DeleteEntity(User entity)
        {
            db.UserSet.Remove(entity);
           return  db.SaveChanges()>0;
        }
 
        public bool EditEntity(User entity)
        {
            db.Entry<User>(entity).State = System.Data.Entity.EntityState.Modified;
            return db.SaveChanges() > 0;
        }
 
        public IQueryable<User> LoadEntities(Expression<Func<User, bool>> whereLambda)
        {
            return db.UserSet.Where(whereLambda);
        }
 
        public IQueryable<User> LoadPageEntities<s>(int pageIndex, int pageSize, out int totalCount, Expression<Func<User, bool>> wherelambda, Expression<Func<User, s>> orderByLambda, bool isasc)
        {
            var temp = db.UserSet.Where(wherelambda);
            totalCount = temp.Count();
            if (isasc)
            {
                temp = temp.OrderBy<User, s>(orderByLambda).Skip<User>(pageIndex - 1).Skip<User>(pageSize);
            }
            else {
                temp = temp.OrderByDescending<User, s>(orderByLambda).Skip<User>(pageIndex - 1).Skip<User>(pageSize);
            }
            return temp;
        }
    }
}
```
![avatar](7.png)

IBLL层
IBaseService.cs (业务接口)

```
using IDAL;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace IBLL
{
     public  interface IBaseService<T> where T:class ,new()
    {
        IDbSession CurrentDBSession { get; }
       IBaseDal<T> CurrentDal { get; set; }
        IQueryable<T> LoadEntities(System.Linq.Expressions.Expression<Func<T, bool>> wherelambda);
        IQueryable<T> LoadPageEntities<s>(int pageIndex, int pageSize, out int totalCount, System.Linq.Expressions.Expression<Func<T, bool>> wherelambda, System.Linq.Expressions.Expression<Func<T, s>> orderByLambda, bool isasc);
        bool DeleteEntity(T entity);
        bool EditEntity(T entity);
        T AddEntity(T entity);
    }
}
```
IUserService.cs用户业务接口
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Model;
namespace IBLL
{
     public  interface IUserService:IBaseService<User>
    {
       
    }
}
```
BLL层
BaseService.cs(业务基类 实现IBaseService接口）
```
using IDAL;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Model;
using DALFactory;
namespace BLL
{
    //接口的成员不用声明为 public,private ,protected
    //接口的实现需要设置为public
    public abstract  class BaseService<T> where T:class ,new()
    {
       public IDbSession CurrentDBSession
        {
            get
            {
                return DBSessionFactory.CreateDBSession();
            }
        }
        public IBaseDal<T> CurrentDal { get; set; }
        public abstract void SetCurrentDal();//这个抽象方法用于(子类)实现， 当前Service是哪一类，需要 哪个DAL。（CurrentDal）（多态）
        public BaseService()
        {
            SetCurrentDal();
        }
        public IQueryable<T> LoadEntities(System.Linq.Expressions.Expression<Func<T, bool>> wherelambda)
        {
            //return  CurrentDBSession.userdal.LoadEntities(wherelambda);
            return CurrentDal.LoadEntities(wherelambda);
        }
       public   IQueryable<T> LoadPageEntities<s>(int pageIndex, int pageSize, out int totalCount, System.Linq.Expressions.Expression<Func<T, bool>> wherelambda, System.Linq.Expressions.Expression<Func<T, s>> orderByLambda, bool isasc)
        {
          return  this.CurrentDal.LoadPageEntities<s>(pageIndex, pageSize,out totalCount, wherelambda, orderByLambda, isasc);
        }
       public  bool DeleteEntity(T entity)
        {
          this.CurrentDal.DeleteEntity(entity);
            return CurrentDBSession.SaveChanges();
                
        }
        public bool EditEntity(T entity)
        {
             CurrentDal.EditEntity(entity);
            return CurrentDBSession.SaveChanges();
        }
       public T AddEntity(T entity)
        {
             CurrentDal.AddEntity(entity);
             CurrentDBSession.SaveChanges();
            return entity;
        }
 
    }
}
```
UserService.cs(继承BaseService,实现IBaseService)
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Model;
using IBLL;
namespace BLL
{
    //BaseService<User>实现了 IUserService
    //UserService继承了BaseService<User>,实现了IUserSerivce
   public class UserService : BaseService<User>,IUserService
    {
        public override void SetCurrentDal()
        {
            this.CurrentDal = this.CurrentDBSession.userdal;//多态
        }
        //public void add(User u)
        //{
        //    this.CurrentDal.AddEntity(u);
        //    this.CurrentDBSession.SaveChanges();
        //}
    }
}
```
到此项目搭建基本完成。
由于让业务层和UI层解耦，还需要使用Spring.Net进行依赖注入和控制反转。
这里需要注意Expressions是表达式 Func 是Lambda.
IQuerable有延时加载。一般DAL返回IQuerable给业务层。业务层再使用IEmuerable.

---
<!-- Place this tag in your head or just before your close body tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>
<!-- Place this tag where you want the button to render. -->

Please <a class="github-button" href="https://github.com/xyzko1/myblog" data-icon="octicon-star" aria-label="Star xyzko1/myblog on GitHub">Star</a> this Project if you like it! <a class="github-button" href="https://github.com/xyzko1" aria-label="Follow @xyzko1 on GitHub">Follow</a> would also be appreciated!
Peace!
