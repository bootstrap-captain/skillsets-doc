## 进程查看

```bash
# 1.查看java进程
ps -ef | grep replay-service-api-1.0-SNAPSHOT.jar

# 包含两个进程： 一个是运行的jar包对应的java进程， 一个是用jps查看的进程
root      5161     1  0 00:11 ?        00:00:07 java -jar /opt/erick/replay-service-api-1.0-SNAPSHOT.jar
root      5246  4503  0 00:26 pts/0    00:00:00 grep --color=auto replay-service-api-1.0-SNAPSHOT.jar

# 2. 过滤
# 2.1 过滤掉grep的进程: -v代表过滤
ps -ef | grep replay-service-api | grep -v grep
# 2.2  筛选出java和account的进程
ps -ef | grep replay-service-api | grep 'java -jar'

root      5161     1  0 00:11 ?        00:00:07 java -jar /opt/erick/replay-service-api-1.0-SNAPSHOT.jar

# 3. 获取端口： awk， 可以将对应的第二个字符获取，也就是30634， 游标是从1开始
ps -ef | grep replay-service-api | grep 'java -jar' | awk '{printf $2}'

5161[root@iZ2ze8y9odvmultgyoflt4Z erick]#
```

