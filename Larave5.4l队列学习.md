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

**正式环境中需要守护进程来运行队列进程,如pm2,supervisor等,防止queue:work意外关闭.**

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

## 队列守护进程

官网使用的是supervisor,配置复杂,最后还没有配置成功,后改用pm2

pm2也同样有进程宕掉重启功能

[http://http://pm2.keymetrics.io/](http://http://pm2.keymetrics.io/ "PM2")
	
    //npm安装
	npm install pm2 -g

    //yarn安装
	yarn global add pm2
   
#### pm2使用
   
   
	pm2 start queue.config.json
    //运行需切换到artisan目录下运行    

    //queue.config.json配置如下

	{
	  "apps":[
	    {
		  "name": "laravel-queue",
		  "args": "queue:work --queue=high,default",
		  "script": "artisan",
		  "exec_interpreter": "php",
		  "error_file":"./storage/logs/laravel-queue-high.error.log",
		  "out_file":"./storage/logs/laravel-queue-high.out.log",
	    }
	  ]
	}
	
    //常用命令

	pm2 list //打印监控进程列表
    pm2 describe id   //查看进程详情
    pm2 delete id 关闭进程id
    pm2 delete all 关闭所有进程
    
    pm2 restart id 重启进程id
    pm2 restart all 重启所有进程

## 重启队列守护进程

队列进程是长生命周期进程，在重启以前，所有源码的修改并不会对其产生影响

当源码修改时,需要重启守护进程

	//错误方式
    pm2 restart //不使用这种重启方式

    //正确方式
    php artisan queue:restart   //会优雅的等任务运行结束后在重启守护进程

## 处理失败的任务


当任务失败时,会调用失败方法和触发失败事件,我们可以出在这里记录日志,发送通知等.

**调用任务失败方法**

调用任务类的failed方法,并传人异常

**触发队列失败事件**


## 队列的各种事件

在AppServiceProvider里面配置监听

	<?php
	namespace App\Providers;
	
	use Illuminate\Support\Facades\Queue;
	use Illuminate\Queue\Events\JobFailed;
	use Illuminate\Queue\Events\JobProcessed;
	use Illuminate\Queue\Events\JobProcessing;
	use Illuminate\Support\ServiceProvider;
	
	class AppServiceProvider extends ServiceProvider
	{
	    /**
	     * 启动应用服务.
	     *
	     * @return void
	     */
	    public function boot()
	    {
             //失败事件
	        Queue::failing(function (JobFailed $event) {
	            // $event->connectionName
	            // $event->job
	            // $event->exception
	        });

           //执行队列任务前事件
	       Queue::before(function (JobProcessing $event) {
	            // $event->connectionName
	            // $event->job
	            // $event->job->payload()
	        });
	         
            //执行队列任务后事件,任务失败了,是不会执行此事件
	        Queue::after(function (JobProcessed $event) {
	            // $event->connectionName
	            // $event->job
	            // $event->job->payload()
	        });
	    }

	}

## 队列相关的命令
	
	php artisan queue:restart  //重启队列
    php artisan queue:retry id|all //重新执行失败任务一个,或者重新执行全部失败任务
    php artisan queue:failed  //查看失败任务列表
    php artisan queue:flush  //删除全部失败任务
    php artisan queue:forget id //删除单个失败任务