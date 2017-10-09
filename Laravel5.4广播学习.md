# Laravel5.4广播学习


### 广播理解

Laravel广播只能实现服务器到客户端单方面的实时通讯,客户端如需和服务端通讯,需要额外的通讯,如ajax提交数据通知服务器

## *切记修改任何后端代码,需要重启队列* ##
    php artisan queue:restart
##前期准备

我测试队列和广播都是使用redis

- Redis服务器
- Redis队列进程(php artisan queue:work 这个是用守护进程,如前面所讲的PM2)
- larave-mix环境配置安装
- laravel-echo-server (安装,并后台运行)
- laravel-echo(安装,前端引入使用)

##前端配置

###npm镜像配置
	//配置淘宝镜像源
	npm config set registry https://registry.npm.taobao.org

###laravel-mix安装与使用

	//三种方式安装,总有一种可以安装成功
    npm install
	cnpm install
	yarn intall
	
	//配置webpack.mix.js
	mix.js('resources/assets/js/echo.js', 'public/js');

    //webpack打包生成
    //开发模式
    npm run dev
    //生产模式,会删除console.log相关信息,当时以为出问题了,研究很久
    npm run prod

###laravel-echo-server安装与运行
	
	//三种方式安装,总有一种可以安装成功
    npm install -g laravel-echo-server
	cnpm install -g laravel-echo-server
	yarn global add laravel-echo-server

    //初始laravel-echo-server,会生成laravel-echo-server.json文件
    laravel-echo-server init
    //在json里面配置认证服务器,和调试模式,数据库等参数
	
	//运行laravel-echo-server
    larave-echo-server start

###laravel-echo安装与运行

	//三种方式安装,总有一种可以安装成功
    npm install laravel-echo
	cnpm install laravel-echo
	yarn add laravel-echo

###前端调用

    //resources/assets/js/echo.js

    import EchoClass from "laravel-echo"

	let Echo = new EchoClass({
	    broadcaster: 'socket.io',
	    host: window.location.hostname + ':6001'
	});
	
	window.echo = Echo.join('channel1')
	.listen('WebSocketEvent', function(e){
		console.log(e);
		document.querySelector('#showbox').innerHTML=e.o.time;
	})
	.here(function(users){
		console.log('here',users);
	})
	.joining(function(users){
		console.log('joining',users);
	})
	.leaving(function(users){
		console.log('leaving',users);
	})  
	.whisper('typing', {
	   name: 'list1'
	}) 
	.listenForWhisper('typing', (e) => {
	    console.log(e.name);
	});
	
	const ele = document.createElement('div');
	ele.id = 'showbox';
	document.body.appendChild(ele);

###打包前端

    //webpack打包生成
    //开发模式
    npm run dev

###模板


    <!--index.blade.php -->
	<!DOCTYPE html>
	<html>
	  <head>
	    <title>index</title>
		<meta charset="utf-8">
		<meta name="csrf-token" content="{{ csrf_token() }}">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
		<style></style>
	  </head>
	  <body>
	  <script src="//{{ Request::getHost() }}:6001/socket.io/socket.io.js"></script>
	  <script src="{{mix('/js/echo.js')}}"></script>
	  </body>
	</html>


## 事件类配置

    //app/Events/WebSocketEvent.php

	namespace App\Events;
	
	use Illuminate\Broadcasting\Channel;
	use Illuminate\Queue\SerializesModels;
	use Illuminate\Broadcasting\PrivateChannel;
	use Illuminate\Broadcasting\PresenceChannel;
	use Illuminate\Foundation\Events\Dispatchable;
	use Illuminate\Broadcasting\InteractsWithSockets;
	use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
	use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;
	

    //需要广播的话,需要继承ShouldBroadcast接口
	class WebSocketEvent implements ShouldBroadcast
	{
	    use Dispatchable, InteractsWithSockets, SerializesModels;
		
		public $o;
		
	    /**
	     * Create a new event instance.
	     *
	     * @return void
	     */
		 
		
	    public function __construct()
	    {
	        //
			
			$this->o = new \stdClass;
			$this->o->time = date('Y-m-d H:i:s');
	    }
	
	    /**
	     * Get the channels the event should broadcast on.
	     *
	     * @return Channel|array
	     */
	    public function broadcastOn()
	    {
            //需要广播的话,配置频道类型和频道名称
			//通道有三种 Channel(公开),PrivateChannel(私人需登录认证),PresenceChannel(存在频道需登录认证)
	        return new PresenceChannel('channel1');
	    }
	}

    //routes/channels.php
    //配置认证条件
	Broadcast::channel('channel1', function () {
	    return Request::user();
	});
	


## 发送广播

    //使用全局函数event
	event(new WebSocketEvent());
    //使用全局函数broadcast
    broadcast(new WebSocketEvent());