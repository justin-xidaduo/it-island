# 🏈 常用命令

### awk

<pre class="language-bash" data-overflow="wrap" data-full-width="false"><code class="lang-bash"><strong># 统计tcp 状态
</strong><strong>netstat -n|awk '/^tcp/ {++state[$NF]} END {for(key in state) print key," \t" ,state[key]}'
</strong><strong>
</strong></code></pre>

### netstat

<pre class="language-sh"><code class="lang-sh"><strong># -l或--listening 显示监控中的服务器的Socket。
</strong><strong># -n或--numeric 直接使用IP地址，而不通过域名服务器。
</strong><strong># -t或--tcp 显示TCP传输协议的连线状况。
</strong># -u或--udp 显示UDP传输协议的连线状况。
# -p或--programs 显示正在使用Socket的程序识别码和程序名称。
<strong># -a或--all 显示所有连线中的Socket。
</strong><strong># -i或--interfaces 显示网络界面信息表单。
</strong>
</code></pre>

### vmstat

{% code overflow="wrap" %}
```bash
[root@i-q88hdhfe ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0 138752 560168 196960 19454352    0    0    34  1286    0    1  2  1 97  0  0
 
# proce r 展示了正在执行和等待cpu资源的任务个数。当这个值超过了cpu个数，就会出现cpu瓶颈。
# proce b 等待IO的进程数量

# memory swpd 正在使用虚拟的内存大小，单位k
# memory free 空闲内存大小
# memory buffer 已用的buff大小，对块设备的读写进行缓冲
# memory cache 已用的cache大小，文件系统的cache

# swap si 每秒从交换区写入内存的大小（单位：kb/s）
# swap so 每秒从内存写到交换区的大小

# io bi 每秒读取的块数（读磁盘）块设备每秒接收的块数量，单位是block，这里的块设备是指系统上所有的磁盘和其他块设备，现在的Linux版本块的大小为1024bytes
# io bo 每秒写入的块数（写磁盘）块设备每秒发送的块数量，单位是block

# system in 每秒中断数，包括时钟中断 这两个值越大，会看到由内核消耗的cpu时间sy会越多
# system cs 每秒上下文切换数 秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目

# cpu us 用户进程执行消耗cpu时间(user time)
# cpu sy 系统进程消耗cpu时间(system time)
# cpu wa wa过高时，说明io等待比较严重，这可能是由于磁盘大量随机访问造成的，也有可能是磁盘的带宽出现瓶颈。
```
{% endcode %}
