---
title: "Task 异步编程测试案例及基础应用说明"
catalog: true
date: 2019-09-01 19:00:00
subtitle: ""
header-img: "Demo.png"
tags:
- Hexo
- Blog
catagories:
- Hexo
---
> created by [xyzko1](https://github.com/xyzko1/xyzko1.github.io) 
> 2019年09月01日 19:00:00

对于多线程，我们经常使用的是Thread。在我们了解Task之前，如果我们要使用多核的功能可能就会自己来开线程，然而这种线程模型在.net 4.0之后被一种称为基于“任务的编程模型”所冲击，因为task会比thread具有更小的性能开销，不过大家肯定会有疑惑，任务和线程到底有什么区别呢？
 任务和线程的区别：
1、任务是架构在线程之上的，也就是说任务最终还是要抛给线程去执行。
2、任务跟线程不是一对一的关系，比如开10个任务并不是说会开10个线程，这一点任务有点类似线程池，但是任务相比线程池有很小的开销和精确的控制。

# 认识Task和Task的基本使用
---
## 认识Task
首先来看一下Task的继承结构。Task标识一个异步操作。
![avatar](1.png)
可以看到Task和Thread一样，位于System.Threading命名空间下，这也就是说他们直接有密不可分的联系。下面我们来仔细看一下吧！

## 创建Task
创建Task的方法有两种，一种是直接创建——new一个出来，一种是通过工厂创建。下面来看一下这两种创建方法：
```
        //第一种创建方式，直接实例化
         var task1 = new Task(() =>
         {
            //TODO you code
         });
```
这是最简单的创建方法，可以看到其构造函数是一个Action，其构造函数有如下几种，比较常用的是前两种。
![avatar](2.png)
```
        //第二种创建方式，工厂创建
         var task2 = Task.Factory.StartNew(() =>
         {
            //TODO you code
         });
```
这种方式通过静态工厂，创建以个Task并运行。下面我们来建一个控制台项目，演示一下，代码如下：
要添加System.Threading.Tasks命名控件引用。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace TaskDemo
{
   class Program
   {
      static void Main(string[] args)
      {
         var task1 = new Task(() =>
         {
            Console.WriteLine("Hello,task");
         });
         task1.Start();

         var task2 = Task.Factory.StartNew(() =>
         {
            Console.WriteLine("Hello,task started by task factory");
         });

         Console.Read();
      }
   }
}
```

 这里我分别用两种方式创建两个task,并让他们运行。可以看到通过构造函数创建的task,必须手动Start,而通过工厂创建的Task直接就启动了。
下面我们来看一下Task的声明周期，编写如下代码：

```
      var task1 = new Task(() =>
         {
            Console.WriteLine("Begin");
            System.Threading.Thread.Sleep(2000);
            Console.WriteLine("Finish");
         });
         Console.WriteLine("Before start:" + task1.Status);
         task1.Start();
         Console.WriteLine("After start:" + task1.Status);
         task1.Wait();
         Console.WriteLine("After Finish:" + task1.Status);

         Console.Read();

```
　　task1.Status就是输出task的当前状态，其输出结果如下：
![avatar](3.png)
可以看到调用Start前的状态是Created，然后等待分配线程去执行，到最后执行完成。
从我们可以得出Task的简略生命周期：
Created：表示默认初始化任务，但是“工厂创建的”实例直接跳过。
WaitingToRun: 这种状态表示等待任务调度器分配线程给任务执行。
RanToCompletion：任务执行完毕。

 # Task的任务控制
---
　　Task最吸引人的地方就是他的任务控制了，你可以很好的控制task的执行顺序，让多个task有序的工作。下面来详细说一下：
##  Task.Wait
在上个例子中，我们已经使用过了，task1.Wait();就是等待任务执行完成，我们可以看到最后task1的状态变为Completed。

##  Task.WaitAll
看字面意思就知道，就是等待所有的任务都执行完成，下面我们来写一段代码演示一下：

```
    static void Main(string[] args)
      {
         var task1 = new Task(() =>
         {
            Console.WriteLine("Task 1 Begin");
            System.Threading.Thread.Sleep(2000);
            Console.WriteLine("Task 1 Finish");
         });
         var task2 = new Task(() =>
         {
            Console.WriteLine("Task 2 Begin");
            System.Threading.Thread.Sleep(3000);
            Console.WriteLine("Task 2 Finish");
         });
         
         task1.Start();
         task2.Start();
         Task.WaitAll(task1, task2);
         Console.WriteLine("All task finished!");

         Console.Read();
      }
```

其输出结果如下：
![avatar](4.png)
可以看到，任务一和任务二都完成以后，才输出All task finished！

##  Task.WaitAny
这个用法同Task.WaitAll，就是等待任何一个任务完成就继续向下执行，将上面的代码WaitAll替换为WaitAny，输出结果如下：
![avatar](5.png)

##  Task.ContinueWith
就是在第一个Task完成后自动启动下一个Task，实现Task的延续，下面我们来看下他的用法，编写如下代码：

```
static void Main(string[] args)
      {
         var task1 = new Task(() =>
         {
            Console.WriteLine("Task 1 Begin");
            System.Threading.Thread.Sleep(2000);
            Console.WriteLine("Task 1 Finish");
         });
         var task2 = new Task(() =>
         {
            Console.WriteLine("Task 2 Begin");
            System.Threading.Thread.Sleep(3000);
            Console.WriteLine("Task 2 Finish");
         });

         
         task1.Start();
         task2.Start();
         var result = task1.ContinueWith<string>(task =>
         {
            Console.WriteLine("task1 finished!");
            return "This is task result!";
         });

         Console.WriteLine(result.Result.ToString());


         Console.Read();
      }

```
执行结果如下：
![avatar](6.png)
可以看到，task1完成之后，开始执行后面的内容，并且这里我们取得task的返回值。
在每次调用ContinueWith方法时，每次会把上次Task的引用传入进来，以便检测上次Task的状态，比如我们可以使用上次Task的Result属性来获取返回值。我们还可以这么写：

```
var SendFeedBackTask = Task.Factory.StartNew(() => { Console.WriteLine("Get some Data!"); })
                            .ContinueWith<bool>(s => { return true; })
                            .ContinueWith<string>(r => 
                              {
                                 if (r.Result)
                                 {
                                    return "Finished";
                                 }
                                 else
                                 {
                                    return "Error";
                                 }
                              });
         Console.WriteLine(SendFeedBackTask.Result);
```
首先输出Get some data,然后执行第二个获得返回值true,最后根据判断返回Finished或error。输出结果:
Get some Data!
Finished
其实上面的写法简化一下，可以这样写：
```
Task.Factory.StartNew<string>(() => {return "One";}).ContinueWith(ss => { Console.WriteLine(ss.Result);});
```
输出One，这个可以看明白了吧~
 更多ContinueWith用法参见：

##  Task的取消
前面说了那么多Task的用法，下面来说下Task的取消，比如我们启动了一个task,出现异常或者用户点击取消等等，我们可以取消这个任务。
如何取消一个Task呢，我们通过cancellation的tokens来取消一个Task。在很多Task的Body里面包含循环，我们可以在轮询的时候判断IsCancellationRequested属性是否为True，如果是True的话就return或者抛出异常，抛出异常后面再说，因为还没有说异常处理的东西。
下面在代码中看下如何实现任务的取消，代码如下：

```
var tokenSource = new CancellationTokenSource();
         var token = tokenSource.Token;
         var task = Task.Factory.StartNew(() =>
         {
            for (var i = 0; i < 1000; i++)
            {
               System.Threading.Thread.Sleep(1000);
               if (token.IsCancellationRequested)
               {
                  Console.WriteLine("Abort mission success!");
                  return;
               }
            }
         }, token);
         token.Register(() =>
         {
            Console.WriteLine("Canceled");
         });
         Console.WriteLine("Press enter to cancel task...");
         Console.ReadKey();
         tokenSource.Cancel();
         Console.ReadKey();//这句忘了加，程序退出了，看不到“Abort mission success!“这个提示
```

这里开启了一个Task,并给token注册了一个方法，输出一条信息，然后执行ReadKey开始等待用户输入，用户点击回车后，执行tokenSource.Cancel方法，取消任务。其输出结果如下：
![avatar](7.png)

好了，今天先说道这里，明天继续讲task，接下来该说说task的异常处理和其他的一些用法，如果喜欢可以关注我，一有更新会马上通知你。
异步编程能充分利用多核计算机，提升程序性能。
请看如下程序：
```
static object o = new object();
        static void Main(string[] args)
        {

            var tf = new TaskFactory();//线程池对象
            Task t1 = tf.StartNew(TaskMenthod, "1");//创建并启用线程方法1
            Task t2 = Task.Factory.StartNew(TaskMenthod, "2");//创建并启用线程方法2
            Task t3 = new System.Threading.Tasks.Task(TaskMenthod, "3");
            t3.Start();//创建并启用线程方法3
            Task t4 = Task.Run(() => TaskMenthod("4"));//创建并启用线程方法4
           
            //
            Task t5 = Task.Run(() => TaskMenthod("5"));
            Task t6 = Task.Run(() => TaskMenthod("6"));
            Task t7 = Task.Run(() => TaskMenthod("7"));
            Task t8 = Task.Run(() => TaskMenthod("8"));
            Task t9 = Task.Run(() => TaskMenthod("9"));
            Task t10 = Task.Run(() => TaskMenthod("10"));
            Console.ReadKey();
        }

        static void TaskMenthod(object title)
        {
            lock (o)//确保一个线程执行完方法后 其他线程才能进入
            {
                //连续输出四次
                Console.WriteLine(title.ToString());
                Console.WriteLine(title.ToString());
                Console.WriteLine(title.ToString());
                Console.WriteLine(title.ToString());
            }
        }
```
运行结果
![avatar](8.png)
因为Task是异步的，所以输出的结果不是按照1~10的顺序输出的，相信这点大家都能理解。
但是：
我们发现本应在一个方法体内连续输出的四个9却被分成了两部分，为何会这样呢？
这是因为多个Task同时访问TaskMenthod方法，他们之间是无序的，各个Task之间是没有优先级别的，因此会造成上述问题。
如何解决呢？
通过C# LOCK 可以解决，我们对TaskMenthod方法进行加锁即可，代码修正如下：
##  Task的Wait方法
![avatar](9.png)
运行代码如下：
```
class Program
    {
        static object o = new object();
        static void Main(string[] args)
        {
            var tf = new TaskFactory();//
            Task t1 = tf.StartNew(TaskMenthod, "1");
            t1.Wait();//等待线程1 执行完毕后 继续执行
            Task t2 = Task.Factory.StartNew(TaskMenthod, "2");
            t2.Wait();
            Task t3 = new System.Threading.Tasks.Task(() => TaskMenthod("3"));
            t3.Start();
            t3.Wait();
            Task t4 = Task.Run(() => TaskMenthod("4"));
            t4.Wait();

            //
            Task t5 = Task.Run(() => TaskMenthod("5"));
            Task t6 = Task.Run(() => TaskMenthod("6"));
            Task t7 = Task.Run(() => TaskMenthod("7"));
            Task t8 = Task.Run(() => TaskMenthod("8"));
            Task t9 = Task.Run(() => TaskMenthod("9"));
            Task t10 = Task.Run(() => TaskMenthod("10"));
            Console.ReadKey();
        }

        static void TaskMenthod(object title)
        {
            lock (o)//确保一个线程执行完方法后 其他线程才能进入
            {
                Console.WriteLine(title);
            }
        }
    }
```
以上程序中t1 t2 t3 t4 这四个Task进行了Wait操作，也就是说：其他程序必须等待线程 1 2 3 4 执行完毕后，才能执行，而且线程 1 2 3 4 完全按照代码编写顺序进行输出。
三次执行结果如下：
![avatar](10.png)
![avatar](11.png)
![avatar](12.png)
由上图可知，线程1 2 3 4 的输出牢牢的把控了前四的位置，且是严格按照顺序输出的。
##  Task的WaitAll方法
![avatar](13.png)
将上述代码修改为如下：
```
class Program
    {
        static object o = new object();
        static void Main(string[] args)
        {
            var tf = new TaskFactory();//线程池对象
            Task t1 = tf.StartNew(TaskMenthod, "1");
            Task t2 = Task.Factory.StartNew(TaskMenthod, "2");
            Task t3 = new System.Threading.Tasks.Task(() => TaskMenthod("3"));
            t3.Start();
            Task t4 = Task.Run(() => TaskMenthod("4"));
            Task.WaitAll(t1, t2, t3, t4); //优先执行线程1 2 3 4    线程 1 2 3 4 之间是异步的，也就是说执行顺序是无序的   待线程 1 2 3 4 执行完毕后  才会执行后续的线程

            //
            Task t5 = Task.Run(() => TaskMenthod("5"));
            Task t6 = Task.Run(() => TaskMenthod("6"));
            Task t7 = Task.Run(() => TaskMenthod("7"));
            Task t8 = Task.Run(() => TaskMenthod("8"));
            Task t9 = Task.Run(() => TaskMenthod("9"));
            Task t10 = Task.Run(() => TaskMenthod("10"));
            Console.ReadKey();
        }

        static void TaskMenthod(object title)
        {
            lock (o)//确保一个线程执行完方法后 其他线程才能进入
            {
                Console.WriteLine(title);
            }
        }
    }
```
上面有句注释：Task.WaitAll(t1, t2, t3, t4); //优先执行线程1 2 3 4 线程 1 2 3 4 之间是异步的，也就是说执行顺序是无序的 待线程 1 2 3 4 执行完毕后 才会执行后续的线程
三次执行结果：
![avatar](14.png)
![avatar](15.png)
![avatar](16.png)
由图可知，t1 t2 t3 t4 这四个线程优先执行，但是这四个线程之间的执行是异步的。理解这一点很重要。
继续探讨WaitAll方法
由代码 Task.WaitAll(t1, t2, t3, t4); 可知，必须等待 t1 t2 t3 t4 这四个线程执行完毕后，才会执行下面的操作。、
在此假设名字为t3的线程，我们将 t3.Start(); 去掉，看看会是社么结果：
代码如下：
```
  class Program
    {
        static object o = new object();
        static void Main(string[] args)
        {
            var tf = new TaskFactory();//线程池对象
            Task t1 = tf.StartNew(TaskMenthod, "1");
            Task t2 = Task.Factory.StartNew(TaskMenthod, "2");
            Task t3 = new System.Threading.Tasks.Task(() => TaskMenthod("3"));
            //t3.Start();  //t3 不执行了
            Task t4 = Task.Run(() => TaskMenthod("4"));
            Task.WaitAll(t1, t2, t3, t4); //优先执行线程1 2 3 4    线程 1 2 3 4 之间是异步的，也就是说执行顺序是无序的   待线程 1 2 3 4 执行完毕后  才会执行后续的线程

            //
            Task t5 = Task.Run(() => TaskMenthod("5"));
            Task t6 = Task.Run(() => TaskMenthod("6"));
            Task t7 = Task.Run(() => TaskMenthod("7"));
            Task t8 = Task.Run(() => TaskMenthod("8"));
            Task t9 = Task.Run(() => TaskMenthod("9"));
            Task t10 = Task.Run(() => TaskMenthod("10"));
            Console.ReadKey();
        }

        static void TaskMenthod(object title)
        {
            lock (o)//确保一个线程执行完方法后 其他线程才能进入
            {
                Console.WriteLine(title);
            }
        }
    }
```
执行结果：
![avatar]17.png)
由此可知：因为 t3 的罢工，导致其他线程一直处于等待状态(得不到执行)，如何解决这个问题呢？
还好，WaitAll方法提供的重载方法中有一个和时间结合的，如下：
![avatar](18.png)
将代码修改如下：
```
 class Program
    {
        static object o = new object();
        static void Main(string[] args)
        {
            var tf = new TaskFactory();//线程池对象
            Task t1 = tf.StartNew(TaskMenthod, "1");
            Task t2 = Task.Factory.StartNew(TaskMenthod, "2");
            Task t3 = new System.Threading.Tasks.Task(() => TaskMenthod("3"));
            //t3.Start();  //t3 不执行了  罢工了  咋的吧
            Task t4 = Task.Run(() => TaskMenthod("4"));

            Task[] tskArray = new Task[] { t1, t2, t3, t4 };
            Task.WaitAll(tskArray,3000); //等待三秒后，执行后续线程

            //
            Task t5 = Task.Run(() => TaskMenthod("5"));
            Task t6 = Task.Run(() => TaskMenthod("6"));
            Task t7 = Task.Run(() => TaskMenthod("7"));
            Task t8 = Task.Run(() => TaskMenthod("8"));
            Task t9 = Task.Run(() => TaskMenthod("9"));
            Task t10 = Task.Run(() => TaskMenthod("10"));
            Console.ReadKey();
        }

        static void TaskMenthod(object title)
        {
            lock (o)//确保一个线程执行完方法后 其他线程才能进入
            {
                Console.WriteLine(title);
            }
        }
    }
```
这样，在程序等待三秒后，会执行后续操作！
##  WaitAny 方法，既是等待任意线程执行完毕后，执行后续方法
将上述程序中的WaitAll 修改成WaitAny 试一下
```
class Program
    {
        static object o = new object();
        static void Main(string[] args)
        {
            var tf = new TaskFactory();//线程池对象
            Task t1 = tf.StartNew(TaskMenthod, "1");
            Task t2 = Task.Factory.StartNew(TaskMenthod, "2");
            Task t3 = new System.Threading.Tasks.Task(() => TaskMenthod("3"));
            //t3.Start();  //t3 不执行了  罢工了  咋的吧
            Task t4 = Task.Run(() => TaskMenthod("4"));

            Task.WaitAny(t1, t2, t3, t4); //

            //
            Task t5 = Task.Run(() => TaskMenthod("5"));
            Task t6 = Task.Run(() => TaskMenthod("6"));
            Task t7 = Task.Run(() => TaskMenthod("7"));
            Task t8 = Task.Run(() => TaskMenthod("8"));
            Task t9 = Task.Run(() => TaskMenthod("9"));
            Task t10 = Task.Run(() => TaskMenthod("10"));
            Console.ReadKey();
        }

        static void TaskMenthod(object title)
        {
            lock (o)//确保一个线程执行完方法后 其他线程才能进入
            {
                Console.WriteLine(title);
            }
        }
    }
```

---
<!-- Place this tag in your head or just before your close body tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>
<!-- Place this tag where you want the button to render. -->

Please <a class="github-button" href="https://github.com/xyzko1/myblog" data-icon="octicon-star" aria-label="Star xyzko1/myblog on GitHub">Star</a> this Project if you like it! <a class="github-button" href="https://github.com/xyzko1" aria-label="Follow @xyzko1 on GitHub">Follow</a> would also be appreciated!
Peace!
