#多线程 GCD

###队列
* serialQueue 串行队列
* concurrentQueue 并发队列
在gcd中,开发者,不去操作具体的线程,而是通过队列和任务关联的方式去进行,多线程的编码
多线程是异步执行和并发执行的解决方式,GCD回归到了,问题的本质,去关注要进行处理的任务和执行的方式

### 任务关联队列
在GCD中,任务关联了队列之后,就开始进行多线程的操作了
在GCD中有两种任务关联方式,官方把这种关联方式称之为"分发" - dispatch;
分发分为两种

* dispatch_sync 	```同步分发``` 

```
使用同步分发的任务,关联队列之后,会同步的处理,
所谓同步的处理,就是前一个任务执行完成之后,再执行下一个任务
```

* dispatch_async ```异步分发```

```
使用异步分发的任务,关联队列之后,会异步的处理,
所谓异步的处理,就是前一个任务开始执行后,立即执行下个队列中的任务
,任务执行不依赖前一个任务的执行结果
```

在GCD之中,对于队列的描述也不相同

* 串行队列 serial
* 并发队列 concurrent

```
串行队列执行任务的时候,会依赖前一个任务的执行结果,顺序的执行队列中的任务
并发队列执行任务的时候,不会依赖前一个任务的执行结果(前提是,不是同步分发形式分发来的任务)
```
多种分发方式,和多种队列组合之后的结果如下:

* async + serial (异步分发,串行队列)  开启一条新线程,队列中的任务串行执行
* async + concurrent (同步分发,并发队列) 开启多条线程,队列中的任务并发执行
* sync + serial (同步分发,串行队列) 不开启线程,在当前线程下顺序执行任务
* sync + concurrent (同步分发,并发队列) 不开启线程,在当前线程下顺序执行任务

其他特殊队列

* 主线程队列
* 全局线程队列

```
全局线程队列,就是系统维护的一套线程池,其中都是并发队列,处理结果和方式,可以参照concurrent_Queue形的队列
需要注意的是主线程队列
主线程队列是GCD中唯一一个表明了执行线程的队列,在使用 同步分发的方式,去把任务分发到主线程中时,会造成死锁
(由于主线程一直接受系统的runloop的相应,所以一直有任务执行,会出现,要执行的任务一直无法执行,导致死锁的发生)
```

### 线程同步

线程同步,是发生在多线程编程中经常需要处理的问题

线程同步
```
场景: 例如,小明吃完饭需要喝热水,吃饭为一个任务,烧热水为另一个任务,我们多线程执行这两个任务,
为了实现"小明吃完饭需要喝热水"的需求,在小明吃完饭和烧好热水这两个任务,都完成了需要同步一下,
让小明刚刚吃好饭就可以喝热水.
所以,多线程编程中进行,线程同步是一个常见的操作
```
在GCD中有多种方式进行线程同步

* barrier 栅栏方法
栏杆方法会把,异步的多线程任务从中间分隔开,就如同在多线程之间,用栅栏隔开一样
```
栅栏方法特点如下:
1,栅栏方法之前的,异步的多线程任务执行完成之后,才会调用栏杆方法中的任务
2,栅栏方法之后的,多线程任务(异步,或,同步),都必须在前面的栏杆方法执行完成之后才会执行
3,栅栏方法内的任务,是在一条线程中串行执行的
```

* dispatch_group 分发任务到队列并关联线程组

```
dispatch_group,可以用来关联一个队列中的多个任务,也可以用来关联多个队列中的多个任务,关联到一个组的任务,可以使用notiy方法来相应,任务处理结果
```

```
group-notiy
一般是和group配对使用,达到线程同步的目的,可以关注多个队列中的多个任务的完成状态,进行监听同步,处理后续事务
可以理解为,一个关联了任务队列的队列组,当队列中的分发任务完成之后,将会执行notiy关联的队列中的分发任务(CallBack只是其中的一种处理方式)
```

```
group-wait
等待group中的任务,执行,后续配置的超时时间,可以制作类似网络请求超时的处理
注意,group_wait会阻塞调用线程,所以不要在主线程中调用
```

* dispatch_enter/leave 关联分组进入/退出块

```
enter和leave成对出现,也是结合dispatch_group处理同步问题
目的解决,异步调用的多线程的同步问题,和notiy的区别是,当关联线程中又异步的执行了任务,那么notiy会直接执行响应如:
dsipatch_group_async(^ {
	afnetwork 网络请求
})
dsipatch_group_async(^ {
	afnetwork 网络请求2
})
dispatch_notiy(^{
	// 由于afnetwork是一个异步请求,所以当网络请求还没有回来的时候,就会执行
	(如果不用afnetwork而是阻塞,两个异步线程,那么notiy就可以达到目的)
})
```

为了解决以上问题
可以这样处理:

```

dispatch_group_t group = dispatch_group_create();
dispatch_group_enter(group);
dispatch_async(dispatch_get_global_queue(0, 0), ^{
   
    AFnetwork(^ success {
    	// 执行完不leave,不会响应notiy
    dispatch_group_leave(group);
    });
});
dispatch_group_enter(group);
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    
    AFnetwork(^ success {
    	// 执行完不leave,不会响应notiy
    dispatch_group_leave(group);
    });
});
dispatch_group_notify(group, dispatch_get_global_queue(0, 0), ^{
    
    NSLog(@"任务完成");
});

使用create_group的方式,手动阻塞group的相应,放置异步执行时,不关注执行结果的问题
```

* dispatch_semaphore 信号量处理

```
这里的处理逻辑和 group块的方式类似,但是使用信号量的方式,
是更加的直接的把异步的多线程处理,转化为同步的逻辑处理
而是直接去处理信号量
还是以上面的例子
在网络请求之中,去设置线程的阻塞,以达到线程的同步
dispatch_semaphore_t sema = dispatch_semaphore_create(0);
[网络请求:{
        成功：dispatch_semaphore_signal(sema);
        失败：dispatch_semaphore_signal(sema);
}];
dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
```

* dispatch_semaphore和dispatch_enter/leave

```
对于这个的区别,个人的感觉是, enter/leave 是对关联的线程组的一个标记,影响的是关联到 group 的 notiy方法
而,对于信号量,他是直接使用阻塞线程方式,去让异步的线程执行方式,变为,同步的执行方式
```