# Canvas跨域问题

### 问题描述

在使用cropit截图时出现The operation is insecure问题

### 解决办法

1,服务器端对图片资源和视频资源添加 Access-Control-Allow-Origin头


    #nginx服务器
	location ~*  \.(jpg|jpeg|png|gif|ico)$ {
	   expires 365d; 
	   add_header 'Access-Control-Allow-Origin' '*';
	}


2,对Img和Video添加crossOrigin属性.

    
    var img = new Image,
    canvas = document.createElement("canvas"),
    ctx = canvas.getContext("2d"),
    src = "http://example.com/image"; // insert image url here

	img.crossOrigin = "anonymous";
	
	img.onload = function() {
	    canvas.width = img.width;
	    canvas.height = img.height;
	    ctx.drawImage( img, 0, 0 );
	    localStorage.setItem( "savedImageData", canvas.toDataURL("image/png") );
	}
	img.src = src;

## 其他问题

添加crossOrigin属性后,ios部分手机不支持onload事件.

目前我的解决办法是,不采用onload事件,直接延迟执行代码.


	setTimeout(function(){
	  //代码
	},1000);


> 启用了 CORS 的图片
> [https://developer.mozilla.org/zh-CN/docs/Web/HTML/CORS_enabled_image](https://developer.mozilla.org/zh-CN/docs/Web/HTML/CORS_enabled_image) 
