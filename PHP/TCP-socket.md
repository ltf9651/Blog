## PHP实现TCP服务端与客户端
```php
<?php 
//客户端
set_time_limit(0); 
   
$host = "127.0.0.1"; 
$port = 3046; 
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP)or die("Could not create  socket\n"); 
    
$connection = socket_connect($socket, $host, $port) or die("Could not connet server\n");
socket_write($socket, "hello socket") or die("Write failed\n");
while ($buff = socket_read($socket, 1024, PHP_NORMAL_READ)) { 
    echo("Response was:" . $buff . "\n");
    echo("input what you want to say to the server:\n");
    $text = fgets(STDIN);
    socket_write($socket, $text);
} 
socket_close($socket);
```

```php
<?php 
//服务端

set_time_limit(0);  //确保在连接客户端时不会超时 
//设置IP和端口号  
$address = "127.0.0.1";  
$port = 3046; 
/** 
 * 创建一个SOCKET  
 * AF_INET=是ipv4 如果用ipv6，则参数为 AF_INET6 
 * SOCK_STREAM为socket的tcp类型，如果是UDP则使用SOCK_DGRAM 
*/  
$sock = socket_create(AF_INET, SOCK_STREAM, SOL_TCP) or die("socket_create() fail:" . socket_strerror(socket_last_error()) . "/n");  
//阻塞模式  
socket_set_block($sock) or die("socket_set_block() fail:" . socket_strerror(socket_last_error()) . "/n");  
//绑定到socket端口  
$result = socket_bind($sock, $address, $port) or die("socket_bind() fail:" . socket_strerror(socket_last_error()) . "/n");  
//开始监听  
$result = socket_listen($sock, 4) or die("socket_listen() fail:" . socket_strerror(socket_last_error()) . "/n");  
echo "OK\nBinding the socket on $address:$port ... ";  
echo "OK\nNow ready to accept connections.\nListening on the socket ... \n";  
do { // never stop the daemon  
    //它接收连接请求并调用一个子连接Socket来处理客户端和服务器间的信息  
    $msgsock = socket_accept($sock) or  die("socket_accept() failed: reason: " . socket_strerror(socket_last_error()) . "/n");  
    while(1){
		//读取客户端数据  
		echo "Read client data \n";  
		//socket_read函数会一直读取客户端数据,直到遇见\n,\t或者\0字符.PHP脚本把这写字符看做是输入的结束符.  
		$buf = socket_read($msgsock, 8192);  
		echo "Received msg: $buf   \n";
		if($buf == "bye"){
			//接收到结束消息，关闭连接，等待下一个连接
			socket_close($msgsock);
			continue;
		}
		  
		//数据传送 向客户端写入返回结果  
		$msg = "welcome \n";  
		socket_write($msgsock, $msg, strlen($msg)) or die("socket_write() failed: reason: " . socket_strerror(socket_last_error()) ."/n");  		
	}  
      
} while (true);  
socket_close($sock);  
```