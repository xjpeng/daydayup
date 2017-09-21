# Laravel5.4事件学习


## 注册事件和监听器



### 使用事件类和监听器类方式

在EventServicePrivoder里面配置$listen属性

    protected $listen = [

       //在这里面添加你的事件名和监听器

       //事件名
       'App\Events\Event1'=>[
          //监听器1
	      'App\Listeners\Notice1',
          //监听器2
		  'App\Listeners\Notice2'
	   ]
    ];
生成事件类和监听类

    //自动生成事件类和监听类
	php artisan event:generate

### 使用闭包方式

在EventServicePrivoder里面配置boot方法

    public function boot()
    {
        parent::boot();
		Event::listen('event.event1',function($a,$b){
			//处理事件
		});
		
        //*号,匹配0-n个字符
		Event::listen('event.*',function($a,$b){
			//处理事件
		});
		
    }
    //闭包参数,event函数第二个参数使用数组传递 event('event.event1',[1,2]);

### 把事件放到队列里执行

***闭包方式不能使用队列***

只需要吧监听器类继承ShouldQueue接口即可,前提是队列进程queue:work在运行

	use Illuminate\Contracts\Queue\ShouldQueue;
	
	class Notice1 implements ShouldQueue{

## 指派事件

	//使用全局函数event指派
    //OrderShipped是事件类
	event(new OrderShipped($order));