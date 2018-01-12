# Nginx socket.io转发 #



## 问题列表 ##

### nginx unknown directive "proxy_ http_version" ###

解决办法:
nginx版本太低,升级到1.3以上版本

### “nginx dead but pid file exists” ###

解决办法:
重启电脑

### nginx: [emerg] socket() [::]:80 failed (97: Address family not supported by protocol) ###

解决办法：

    vim /etc/nginx/conf.d/default.conf
    将
    listen   80 default_server;
    listen   [::]:80 default_server;
    
    
    改为：
    listen   80;
    #listen   [::]:80 default_server;
	重新启动nginx即可


### “nginx socket.io链接不上” ###

    location ^~ /node/{
	    proxy_pass http://127.0.0.1:3000/;
	    proxy_http_version 1.1;
	    proxy_set_header Upgrade $http_upgrade;
	    proxy_set_header Connection "upgrade";
    }

	//客户端连接时,记得带上转发前的路径
    <script src="/node/socket.io/socket.io.js"></script>
	<script>
		var socket = io('http://ibc.cqleju.net',{path: '/node/socket.io/'});
	</script>