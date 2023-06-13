# nginx

nginx配置文件

```
user  nginx;
worker_processes  4;

error_log  /var/log/nginx/error.log warn ;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;  #指定最大可以同时接收的连接数量，这里一定要注意，最大连接数量是和worker processes共同决定的
    multi_accept on;   #配置指定nginx在收到一个新连接通知后尽可能多的接受更多的连接
    use epoll;   #配置指定了线程轮询的方法
}




http {
    include       /etc/nginx/mime.types;  #指定在当前文件中包含另一个文件的指令, mime type和文件扩展名的对应关系一般放在mime.types里mime.types 文件里其实就是用 types 指令来定义
    default_type  application/octet-stream;  #指定默认处理的文件类型,若未定的文件类型多是软件或者图像，建议为application/octet-stream,浏览器会提示用户以“另存为”的方式保存文件！

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      'upstream_response_time "$upstream_response_time" request_time "$request_time" ';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;   #配置on让sendfile发挥作用，将文件的回写过程交给数据缓冲去去完成，而不是放在应用中完成，这样的话在性能提升有有好处
    #tcp_nopush     on;
    tcp_nopush on;  #让nginx在一个数据包中发送所有的头文件，而不是一个一个单独发
	tcp_nodelay on;  #让nginx不要缓存数据，而是一段一段发送，如果数据的传输有实时性的要求的话可以配置它，发送完一小段数据就立刻能得到返回值
	keepalive_timeout 3000000; #给客户端分配连接超时时间，服务器会在这个时间过后关闭连接。一般设置时间较短，可以让nginx工作持续性更好
	client_header_timeout 3000000; #设置请求头的超时时间
    client_body_timeout 3000000;  #设置请求body 的超时时间
    send_timeout 3000000;  #指定客户端响应超时时间，如果客户端两次操作间隔超过这个时间，服务器就会关闭这个链接
#    request_terminate_timeout 300000;
    fastcgi_connect_timeout 3000000;
    fastcgi_send_timeout 3000000;
    fastcgi_read_timeout 3000000;
    fastcgi_buffers 8 128k;
    fastcgi_buffer_size 128k;
    fastcgi_busy_buffers_size 256k;
    fastcgi_temp_file_write_size 256k;
#    max_children 30;
    #keepalive_timeout  65;
    client_max_body_size 100m;
    gzip  on;          #告诉nginx采用gzip压缩的形式发送数据。这将会减少我们发送的数据量
    gzip_min_length 1024; #设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于1k的字节数，小于1k可能会越压越大。 即: gzip_min_length 1024
    gzip_comp_level 5;   #设置数据的压缩等级。这个等级可以是1-9之间的任意数值，9是最慢但是压缩比最大的。我们设置为5，这是一个比较折中的设置
    gzip_buffers 16 8k; #设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流, 按照原始数据大小以8k为单位的16倍申请内存
    gzip_types  text/plain application/x-javascript application/javascript application/css  text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/json;
    gzip_disable "msie6";   #为指定的客户端禁用gzip功能。我们设置成IE6或者更低版本以使我们的方案能够广泛兼容
    gzip_proxied any;   #允许或者禁止压缩基于请求和响应的响应流。我们设置为any，意味着将会压缩所有的请求

    include /etc/nginx/conf.d/*.conf;
}
```
