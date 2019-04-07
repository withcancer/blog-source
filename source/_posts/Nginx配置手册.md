---
title: nginx配置手册
date: 2017-12-20
categories:
- 后端
- 服务器
- nginx
tags:
- 后端
- nginx
- 配置
---
# 全局配置
全局配置参数| 解释
---|---
user | 用这个参数来配置worker进程的用户和组，如果忽略group，那么group的名字等于该参数指定用户的用户组
worker_processes| 指定worker进程启动的数量，选择一个正确的数量取决于服务器环境，一个好的经验法则就是把该参数的值与cpu的核心数量相同，并用1.5~2之间的数乘这个数作为I/O密集型负载
error_log | 制定了errorlog的位置，但是要在编译时打开--with-debug选项才可以使用
pid| 指定pid文件的位置
# event
event配置参数 | 解释
---|---
use | 指示采用什么样的连接方法
worker_connections| 制定接受并发连接的最大数量
# http
http配置参数| 解释
---|---
chunked_transfer_encoding| 允许使用http1.1发送请求
client_body_buffer_size| 如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
client_body_in_file_only| 指定是否将用户请求体存储到一个文件里。
client_body_in_single_buffer|该指令设置NGINX将完整的请求主体存储在单个缓冲区中。 默认情况下，指令值为off。 如果启用，它将优化读取$request_body变量时涉及的I/O操作。
client_body_temp_path| 此指令指定存储请求正文的临时文件的位置。 除了位置之外，指令还可以指定文件是否需要最多三个级别的文件夹层次结构。级别指定为用于生成文件夹的位数。默认情况下，NGINX在NGINX安装路径下的client_body_temp文件夹创建临时文件。
client_body_timeout|设置用户请求体的超时时间。
client_header_buffer_size|设置用户请求头所使用的buffer大小
client_header_timeout| 设置用户请求头的超时时间。
client_max_body_size| 设置所能接收的最大请求体的大小，也就是上传文件的大小
keepalive_timeout| keepalive超时时间，保持连接打开的时间
keepalive_request| 连接打开期间，用户可以发送请求的次数
large_client_header_buffers|客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
# I/O
IO配置参数 | 解释
---|---
aio| 该指令启用异步文件IO
directio| 直接I/O是文件系统的一个功能，其从应用程序到磁盘直接读取和写入，从而绕过所有操作系统缓存。 这使得更好地利用CPU周期和提高缓存效率。
directio_alignment| 该指令设置directio的算法，默认值512
open_file_cache| max=65535 inactive=30s可以设置最多缓存多少个文件，缓存多少时间
open_file_cache_errors| 按照open_file_cache的设置，启用查询错误缓存
open_file_cache_min_uses| 在30S中没有使用到这个配置的次数的话就删除
open_file_cache_valid|多少时间检查一次，如果发现30s内没有用过一次的删除
postpone_output|指定Nginx发送给客户端最小的数值，如果可能的话，没有数据会发送，直到达到此值
sendfile|当应用程序传输文件时，内核首先缓冲数据，然后将数据发送到应用程序缓冲区。 应用程序反过来将数据发送到目的地。 Sendfile方法是一种改进的数据传输方法，其中数据在操作系统内核空间内的文件描述符之间复制，而不将数据传输到应用程序缓冲区。 这使操作系统资源的利用率提高。
sendfile_max_chunk|sendfile_max_chunk可以减少阻塞调用sendfile()所花费的最长时间.因为Nginx不会尝试一次将整个文件发送出去,而是每次发送大小为256KB的块数据.

另外：
启用aio时会自动启用directio,小于directio定义的大小的文件则采用sendfile进行发送,超过或等于directio定义的大小的文件,将采用aio线程池进行发送,也就是说aio和directio适合大文件下载.因为大文件不适合进入操作系统的buffers/cache,这样会浪费内存,而且Linux AIO(异步磁盘IO)也要求使用directio的形式.

# hash

hash指令|	说明
---|---
server_names_hash_bucket_size|	指定用于保存server_name哈希表大小的“桶”
server_names_hash_max_size|	指定的server_name哈希表的最大大小
types_hash_bucket_size|	指定用于存放哈希表的“桶”的大小
types_hash_max_size|	指定哈希类型表的最大大小
variables_hash_bucket_size|	它指定用于存放保留变量“桶”的大小
variables_hash_max_size|	指定存放保留变量最大哈希值的大小

# Socket

指令|	说明
---|---
lingering_close|	指定如何保持客户端的连接，以便用于更多数据的传输
lingering_time|	在使用 lingering_close指令的连接中，使用该指令指定客户端连接为了处理更多的数据需要保持打开连接的时间
lingering_timeout|	结合lingering_close，该指令显示Nginx在关闭客户端连接之前，为获得更多数据会等待多久
reset_timedout_connection|	使用这个指令之后，超时的连接将会被立即关闭，释放相关的内 存。默认的状态是处于FIN _ WAIT1，这种状态将会一直保持连接
send_lowat|	如果非零，Nginx将会在客户端套接字尝试减少发送操作
send_timeout|	在两次成功的客户端接收响应的写操作之间设置一个超时时间
tcp_nodelay|	启用或者禁用TCP _ NODELAY选项，用于keep-alive连接
tcp_nopush|	仅依赖于 sendfile的使用。它能够使得Nginx在一个数据包中尝试发送响应头，以及在数据包中发送一个完整的文件
# server
指令|说明
---|---
server_name| 虚拟主机名，可以使用正则表达式，也可以使用字符串捕获功能
## listen
指令|说明
---|---
default_server|	定义这样一个组合： address:port默认的请求被绑定在此	
Setfib|为套接字监听设置相应的FIB	
backlog|在listen()的调用中设置backlog参数调用	
rcvbuf|在套接字监听中设置 SO_RCVBUF 参数	
sndbuf|在套接字监听中设置 SO_SNDBUF参数	
accept_filter|	设置接受的过滤器：dataready或者httpready dataready	
deferred|设置 accept()调用的TCP_DEFER_ACCEPT	
bind|为address:port套接字对打开一个单独的 bind()调用
ipv6only|设置 IPV6_V6ONLY参数的值	
ssl|表明该端口仅接受HttpS的连接 
so_keepalive|为TCP监听套接字配置keepalive
## locations

修饰符|	处理方式
---|---
=|	使用精确匹配并且终止搜索
～|	区分大小写的正则表达式匹配
～*|	不区分大小写的正则表达式匹配
^～	|只匹配uri部分
/| 通用匹配，任何请求都会匹配到。
root| 配置根目录，请求会加根目录的地址
alias| 配置uri地址更改
index| 配置访问首页，按空格分开，先找到哪个就用哪个进行响应
error_page| 错误页面，先code后页面，页面也可以被redirect到新的root目录下
deny| 禁止访问的地址
allow| 允许访问的地址

