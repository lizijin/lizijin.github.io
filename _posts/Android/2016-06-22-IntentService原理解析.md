##IntentService 源码解析

[IntentServcie源码链接](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/app/IntentService.java)

IntentService是Service的子类。与Service不同的是 IntentService在执行完操作之后会主动关闭掉服务。Service程序执行是在主线程中，如果做耗时操作的话会阻塞主线程引起ANR,IntentService程序执行是在子线程中,不会阻塞主线程。

IntentService为什么会在子线程中执行呢流程大概是

	          	  oncreate()	
    IntentService ==========> 启动一个HandlerThread
    HandlerThread是Thread的子类,它会通过Looper一直轮询消息
    
    @Override
    public void onCreate() {
        super.onCreate();
        //创建一个线程,最终的操作onHandleIntent会在这个现场中执行
        HandlerThread thread = new
        	HandlerThread("IntentService[" + mName + "]");
        thread.start();

		//获取线程的Looper 并创建一个Handler与该线程绑定
        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    	
     
             	  oncreate()	
    IntentService ==========> 创建ServiceHandler
    
    ServiceHandler是与HandlerThread绑定在一起的。换句话说通过	ServiceHandler发送消息最终会在HandlerThread中执行  
    
     private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
        	//最终会执行onHandleIntent方法
            onHandleIntent((Intent)msg.obj);
            //执行完后会自动stopSelf
            stopSelf(msg.arg1);
        }
    }  
    
             	  onStart()	
    IntentService ==========> 通过ServiceHandler来发送消息
    
    @Override
    public void onStart(Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        //这个是在主线程中执行 通过mServiceHandler将消息分发
        给HandlerThread执行
        mServiceHandler.sendMessage(msg);
    }

