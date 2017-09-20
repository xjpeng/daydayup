# Laravel队列学习


## 配置

配置文件在config/queue.php,配置default驱动,默认从.env文件里面导入配置

### 驱动列表

常用驱动

- null
- sync
- database
- redis

#### null驱动

放弃队列任务

#### sync驱动

会立即执行,不支持延时执行,并可以调试打印job结果

#### databse驱动

首先建立数据表
php artisan queue:table

按队列顺序执行,支持延时执行

#### redis驱动

会在 redis里建立 queues:(queuename?):delayed?

按队列顺序执行,支持延时执行

默认queuename是default

## 启动队列进程

php artisan queue:work

这个只会监听默认队列default

如想监听特定队列,使用--queue选项

php partisan queue:work --queue=myqueue

如想监听多个队列,使用--queue选项,并会安装queue里面优先级执行,myqueue优先级高于default

php partisan queue:work --queue=myqueue,default

### 运行选项
- --queue=default 
- --tries=0  
- --timeout=60  



## 建立Job任务

php artisan make:job JobName

JobName 在app/Jobs/JobName

如下面建立一个MyJob任务,结果如下

    class MyJob implements ShouldQueue
    {
	    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        private $o;

	    /**
	     * Create a new job instance.
	     *
	     * @return void
	     */
		
	    public function __construct($o)
	    {
	        // 在dispatch时使用构造函数
			$this->o = $o;
	    }
	
	    /**
	     * Execute the job.
	     *
	     * @return void
	     */
	    public function handle()
	    {
	        // 参数都可以依赖注入
            //做一些用时长的工作,不如发邮件,短信,图片处理等等
			dump($this->o);
	    }
  	 }

## 指派Job任务

使用全局函数dispatch指派

语法 
    
    job对象是php artisan make:job出来的类的实列
    dispatch(job);
  
    //可以在任何地方指派任务,这里在路由器里面指派
    Route::get('/job', function () {

		$o = new stdClass;
		$o->date=date('H:i:s');

		$job = new MyJob($o);
		dispatch($job);

        //指派到特定的队列,比如myqueue
	    $job = (new MyJob($o))->onQueue('myqueue');
	    dispatch($job);

        //指派到特定的队列,比如myqueue
	    $job = (new MyJob($o))->onQueue('myqueue');
	    dispatch($job);

        //指派到特定链接驱动,如database,redis,sync
	    $job = (new MyJob($o))->onConnection('database');
	    dispatch($job);

        //延时执行任务,下面是延迟10分钟执行
	    $job = (new MyJob($o))->delay(Carbon::now()->addMinutes(10));
	    dispatch($job);


    });
