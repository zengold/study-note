Linux

关闭防火墙 重启失效 service iptables stop



service firewalld stop



ps -ef | grep java



sudo -i



当前目录下的文件占用磁盘情况

du -sh *



解压

tar -xzvf xxx



压缩

tar -zcvf xxx



推送

rsync -avz xxx root@192.1.8.109:/home/linxl



sftp

sftp linxl@192.1.8.111

put，就可以将东西传过去



查看指令的映射地址

ls -al [指令]



查看进程中CPU占用较高的线程，记录对应的TID（即shift+h后显示的PID）

top -p [PID]，然后 shift + h



获取线程信息，并找到占用CPU高的线程

ps -mp [pid] -o THREAD,tid,time | sort -rn 



将线程对应PID转为 16进制数（TID16）

printf "%x\n" TID



查看该线程的堆栈信息；

jstack [PID] | grep -A 30 "nid=0x[TID16]"





./jstatd -J-Djava.security.policy=/home/linxl/jstatd.all.policy 



-Djava.rmi.server.hostname=192.1.8.111 \

-Dcom.sun.management.jmxremote \
-Dcom.sun.management.jmxremote.port=9998 \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.ssl=false \



查看某个端口的连接数
netstat -nat | grep -iw "9090" | wc -l

查看连接状况

netstat -nat | grep -iw "9090"