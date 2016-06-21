### 1.AsyncTask的基本使用
Android开发中我们经常需要把耗时的操作放到子线程中,避免出现"程序未响应"的对话框。我们经常使用到的方法有使用AsyncTask来处理耗时操作比如以下代码

    public class MyAsyncTask extends AsyncTask<Void,Void,Void> {
        
        /**
        **必须实现doInBackground方法表示把耗时操作放在子线程中执行
        **/
        @Override
        protected Void doInBackground(Void... params) {
            try {
                //模拟耗时操作
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return null;
        }
    }
### 2.AsyncTask的几个泛型参数
上面例子中的三个Void Void Void 泛型参数可能会比较费解。本节我们将对三个参数一一讲解并以一个根据姓名查询学生成绩的案例来讲解

第一个Void参数表示传入给AsyncTask的查询参数

第二个Void参数表示任务执行的进度

第三个Void参数表示操作返回的结果类型

假如我们需要根据用户输入的学生姓名查询学生的年龄。并且显示查询进度。那么三个参数类型分别是 String Double Integer

    public class QueryStudentAgeAsyncTask extends AsyncTask<String, Double, Integer> {

      public String[] mStudents = {"赵一", "钱二", "孙三", "李四", "周五", "吴六", "郑七", "王八"};
      public int[] mAges = {20, 21, 22, 23, 24, 25, 26, 27};

      @Override
      protected Integer doInBackground(String... params) {

        String paramName = params[0];
        Log.d("jiangbin", "输入的参数为" + paramName);
        for (int i = 0; i < mStudents.length; i++) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (paramName.equals(mStudents[i])) {
                publishProgress(100.0);
                return mAges[i];
            } else {
                publishProgress(i * 1.0 / mStudents.length * 100);
            }
        }
        return -1;
      }

      @Override
       protected void onProgressUpdate(Double... values) {
        super.onProgressUpdate(values);
        Log.d("jiangbin", "查询进度" + values[0] + "%");
      }

      @Override
      protected void onPostExecute(Integer integer) {
        super.onPostExecute(integer);
        Log.d("jiangbin", "查询结果为" + integer);
      }
    }
  
  调用如下
  
    new QueryStudentAgeAsyncTask().execute("王八");
  
  执行结果如下
  
    06-21 22:59:39.658 2539-2939/com.peter.myapplication D/jiangbin: 输入的参数为王八
    06-21 22:59:40.692 2539-2539/com.peter.myapplication D/jiangbin: 查询进度0.0%
    06-21 22:59:41.732 2539-2539/com.peter.myapplication D/jiangbin: 查询进度12.5%
    06-21 22:59:42.743 2539-2539/com.peter.myapplication D/jiangbin: 查询进度25.0%
    06-21 22:59:43.780 2539-2539/com.peter.myapplication D/jiangbin: 查询进度37.5%
    06-21 22:59:44.821 2539-2539/com.peter.myapplication D/jiangbin: 查询进度50.0%
    06-21 22:59:45.861 2539-2539/com.peter.myapplication D/jiangbin: 查询进度62.5%
    06-21 22:59:46.902 2539-2539/com.peter.myapplication D/jiangbin: 查询进度75.0%
    06-21 22:59:47.941 2539-2539/com.peter.myapplication D/jiangbin: 查询进度100.0%
    06-21 22:59:47.941 2539-2539/com.peter.myapplication D/jiangbin: 查询结果为27  

总结:

AsyncTask 有几个重要的函数

1.void onPreExecute() 表示在执行子线程任务之前的操作,在主线程中执行。可以在该方法中弹出对话框

2.Result doInBackground(Params... params)表示真正在子线程中完成的操作

3.void onPostExecute(Result result) 操作完成后将结果作为参数返回,在主线程中执行。可以在该方法中更新UI

4.void onProgressUpdate(Progress... values)在主线程中更新操作完成的进度
  
### 3.深入AsyncTask源码
[AsyncTask源码链接](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/AsyncTask.java)

首先我们从AsyncTask<Params, Progress, Result> execute(Params... params)方法入手

    @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);//1
    }
//1处sDefaultExecutor的值为 public static final Executor SERIAL_EXECUTOR = new SerialExecutor(); 这是一个串行的线程执行器,每次只能执行一个Runnable后面我们会详细讲解

紧接着查看 AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,Params... params)

    @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();// 2

        mWorker.mParams = params;
        exec.execute(mFuture);//3

        return this;
    }

//2 处表示在正在执行(//3)之前调用的方法 是在主线程中调用的方法

//3是用线程执行器调用Ruannable 这里默认的线程执行器为SerialExecutor。SerialExecutor的功能是每次只能执行一个Runnable,其它放进线程执行器
的任务只能进入队列中等待调度执行。当正在执行的Ruannable执行完成后才从队列中取出一个执行。这样会导致一个很严重的问题，那就是如果
一个Runnable阻塞了,那么其他等待的Ruannable将永远无法得到调度的机会。这个问题的解决方案后面将给出

    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;//表示当前执行的任务,每次只能执行一个

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {//将任务放到队列中
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();//如果任务执行完毕.取下一个任务执行
                    }
                }
            });
            if (mActive == null) {//如果当前没有任务在执行 直接执行
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);//4 真正的用多线程去执行
            }
        }
    }
  
//4 这才是真正的用多线程去执行Ruannable.我们看下THREAD_POOL_EXECUTOR为何物

    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE = 1;

    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        public Thread newThread(Runnable r) {
            return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
        }
    };

    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * An {@link Executor} that can be used to execute tasks in parallel.
     */
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);//5
  
  //5 假设手机处理器个数为2 CORE_POOL_SIZE=3, MAXIMUM_POOL_SIZE=5,sPoolWorkQueue队列长度为128
  
  CORE_POOL_SIZE=3的意思是假如 AsyncTask1,AsyncTask2,AsyncTask3 同时执行那么将开启3个线程同时执行。AsyncTask4~AsyncTask232
  将会放入到队列中等待。如果还有AsyncTask233任务执行那么将会开辟第四个线程执行,最多开辟5个线程
  
  讲了这么多AsyncTask 真正的执行主体是什么呢。是mWorker
  
     mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                Result result = doInBackground(mParams);
                Binder.flushPendingCommands();
                return postResult(result);//6
            }
        };

//6 postResult(result)是通过绑定在Looper.getMainLooper()的handler来通信的。换句话说就是会在主线程通知执行结果

    发送通知
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
    
    接收通知
    private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    //在这里将执行结果告知给finish方法
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
    
    finish方法 调用onPostExecute方法
    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
  
  ### 4.解决3.0版本后 AsyncTask可能会阻塞死的问题
  1.通过反射执行 void setDefaultExecutor(Executor exec)将默认的串行执行器改成并行的
  
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            try {
                Class clazz;
                clazz = Class.forName("android.os.AsyncTask");
                Method m = clazz.getMethod("setDefaultExecutor", new Class[]{Executor.class});

                int CPU_COUNT = Runtime.getRuntime().availableProcessors();
                int CORE_POOL_SIZE = CPU_COUNT + 1;
                int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
                int KEEP_ALIVE = 1;

                ThreadFactory sThreadFactory = new ThreadFactory() {
                    private final AtomicInteger mCount = new AtomicInteger(1);

                    public Thread newThread(Runnable r) {
                        return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
                    }
                };

                BlockingQueue<Runnable> sPoolWorkQueue =
                        new LinkedBlockingQueue<Runnable>(128);


                Executor THREAD_POOL_EXECUTOR
                        = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                        TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);

                m.invoke(null, new Object[]{THREAD_POOL_EXECUTOR});

            } catch (Exception e) {

            }
        }
  
  2.AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,Params... params)替代AsyncTask<Params, Progress, Result> execute(Params... params)
  
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            try {
                Class clazz;
                clazz = Class.forName("android.os.AsyncTask");
                Method m = clazz.getMethod("setDefaultExecutor", new Class[]{Executor.class});

                int CPU_COUNT = Runtime.getRuntime().availableProcessors();
                int CORE_POOL_SIZE = CPU_COUNT + 1;
                int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
                int KEEP_ALIVE = 1;

                ThreadFactory sThreadFactory = new ThreadFactory() {
                    private final AtomicInteger mCount = new AtomicInteger(1);

                    public Thread newThread(Runnable r) {
                        return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
                    }
                };

                BlockingQueue<Runnable> sPoolWorkQueue =
                        new LinkedBlockingQueue<Runnable>(128);


                Executor THREAD_POOL_EXECUTOR
                        = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                        TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
                      
                new QueryStudentAgeAsyncTask().executeOnExecutor(THREAD_POOL_EXECUTOR,"王八");

            } catch (Exception e) {

            }
        }
