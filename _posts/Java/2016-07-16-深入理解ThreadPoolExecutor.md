## 1.简单了解ThreadPoolExecutor

###1.1 从构造函数开始了解

	public ThreadPoolExecutor(int corePoolSize,
	                         int maximumPoolSize,
	                         long keepAliveTime,
	                         TimeUnit unit,
                             BlockingQueue<Runnable> workQueue,
	                         ThreadFactory threadFactory,
	                         RejectedExecutionHandler handler)

下面我们将解释这个构造函数的七个形参

 1.  corePoolSize 表示线程池的核心线程数

 2. maximumPoolSize 表示线程池的最大线程数

 3. keepAliveTime 表示某个线程在多长时间内没有获取到任务时结束线程

 4. unit 表示keepAliveTime的时间单位，会转化成纳秒

 5. workQueue 表示存放Runnable的任务队列。队列分为有界队列和无界队列，不同类型的队列会影响到maximumPoolSize是否起作用

 6. threadFactory 表示生成线程池中工作线程的线程工厂

 7. handler 表示当队列满了并且工作线程大于等于线程池的最大线程数（maximumPoolSize）时如何来拒绝请求执行的runnable的策略

###1.2举例说明

	//1 第一步创建线程池执行器
	ThreadPoolExecutor threadPoolExecutor =  
			new ThreadPoolExecutor(
					3,//corePoolSize为3
					5,//maximumPoolSize为5
					60,//超时时间为60s
					TimeUnit.SECONDS, 
					//使用有界队列，队列大小为10  
					new ArrayBlockingQueue<Runnable>(10),
					//使用默认的线程工厂
					Executors.defaultThreadFactory(),
					//当任务队列
					new ThreadPoolExecutor.AbortPolicy());

	//2.第二步创建任务。假设每个任务都是睡眠10s种
	Runnable runnable = new Runnable() {
	    @Override
        public void run() {
	        try {
		        TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
	            e.printStackTrace();
            }
        }
		//3.第三步，线程池执行任务
	  threadPoolExecutor.execute(runnable)
###1.3任务执行的流程
####1.3.1执行流程描述和示例图
任务执行的流程主要分为三大步

 1. 执行一个任务时。首先判断当前线程池中的线程数是否小于corePoolSize。如果是则创建新的工作线程并执行任务。如果不是或工作线程创建失败则执行步骤2。
 2. 判断任务队列是否填满了任务。如果没有则将任务放进任务队列中等待线程执行。如果满了则执行步骤3。
 3. 判断当前线程池中的线程数是否小于maximumPoolSize，如果等于maximumPoolSize则拒绝该任务的执行。如果不是则创建新的工作线程来执行该任务。如果创建工作线程失败，也拒绝该任务

示例图如下
![线程池执行流程图](http://img.blog.csdn.net/20160716000225017)


####1.3.2举例说明线程池的执行过程

**从创建threadPoolExecutor的时候计时t=0s**

 **1. t=0s执行threadPoolExecutor.execute(runnable),此时线程池中的线程为0，0小于corePoolSize 3，则创建一个工作线程名为pool-1-thread-1执行runnable。示例图如下**
![当加入第一个任务的时候](http://img.blog.csdn.net/20160716010929044)
 **2. t=1s时执行threadPoolExecutor.execute(runnable),由于t=0时已经创建了pool-1-thread-1的线程，线程池中的线程数为1，1小于corePoolSize,则创建名为pool-1-thread-1执行runnable。示例图如下**
![当加入第二个任务的时候](http://img.blog.csdn.net/20160716011831111)
**3. t=2s时分别执行 threadPoolExecutor.execute(runnable3)和**
 **threadPoolExecutor.execute(runnable4)。执行runnable3时由于线程池线程个数为2小于corePoolSize，会创建pool-1-thread-2线程来执行runnable3.当执行runnable4的时候由于线程池个数已经大于等于corePoolSize了。所以不会再创建新的工作线程，而是把runnable4放入任务队列中。示例图如下**
 ![这里写图片描述](http://img.blog.csdn.net/20160716012830870)
 **4. t=3s时分别执行任务runnable4—runnable13。由于线程池线程数量已经等于corePoolSize，而任务队列没有满，所以把runnable放入到任务队列中。示例图如下**
 ![这里写图片描述](http://img.blog.csdn.net/20160716014124340)
**5. t=4s时执行runnable14。由于线程池中的线程执行的任务的时间为10s，而此刻才过去了4s。所以线程池中的线程都在忙，而任务队列都已经满了。此刻只能在保证线程池中的总线程数不超过maximumPoolSize也就是5的情况下再创建新的线程pool-1-thread-4去执行runnable14。示例图如下**
 ![这里写图片描述](http://img.blog.csdn.net/20160716014628608)
 **6.  t=5s时执行runnable15。由于线程池中的线程执行的任务的时间为10s，而此刻才过去了5s。所以线程池中的线程都在忙，而任务队列都已经满了。此刻只能在保证线程池中的总线程数不超过maximumPoolSize也就是5的情况下再创建新的线程pool-1-thread-5去执行runnable15。示例图如下**
![这里写图片描述](http://img.blog.csdn.net/20160716014930765)
 **7. t=6s时执行runnable16。由于此刻线程池中的线程已经等于最大线程数，任务队列也已经装满了任务。那么runnable16将被相应的拒绝策略来拒绝。默认的是ThreadPoolExecutor.AbortPolicy。表示runnable16将被抛弃。示例图如下**
 ![这里写图片描述](http://img.blog.csdn.net/20160716015423096)
**8.t=11s时,由于pool-1-thread-1和pool-1-thread-2已经将runnable执行完毕。它们将去任务队列中取任务(workQueue.take()方法).示例图如下**
![这里写图片描述](http://img.blog.csdn.net/20160716020547945)
**9.t=30s时，线程池中线程还是5个。任务队列中已经没有任务了。线程池中的线程都在等待往任务队列加入任务**

##2. TODO

 1. execute(Runnable runnable)是如何start一个线程的，并执行任务的
 2. keepAliveTime是如何终止掉多于corePoolSize个数的线程池中的线程。例如corePoolSize为3，keepAliveTime为60s，而线程池中的线程个数为5。如果有一个线程60s后没有从任务队列中获取到任务。那么该线程将会被终止掉

