# mirror 增加

```
#生成mirror 增加配置文件
gpaddmirrors -o ./addmirror   # 回车输入mirror目录 生成如下

0|iot-server3|41000|/data/greenplum/mirror/gpseg0
1|iot-server2|41000|/data/greenplum/mirror/gpseg1

#  增加mirrror
gpaddmirrors -i addmirror



```
