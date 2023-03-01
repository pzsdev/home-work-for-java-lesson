home-work-for-week-three

# 作业题：JVM 虚拟机实践题

## 题目 - 请模拟服务高负载场景并观察 JVM 内部变化

主要关注：①Footprint 内存、②Throughput 吞吐量、③Latency 延迟

- Footprint：Eden、S0、S1，Old 内存区域占用及变化趋势
- Throughput：吞吐量
- Latency：GC 的 STW 引起的延迟



要求：

1. 要求：使用压力测试工具，给项目施加最大压力【低延时接口，高延时接口】
2. 要求：搭建 Grafana 的 JVM 监测的 Dashboard，采集各个指标信息
3. 要求：截图须附上的指标：RT、TPS、GC 的统计信息、堆内存统计信息、吞吐量
4. 要求：配置三种类型的 GC 组合：1. 吞吐量优先，2. 响应时间优先，3. 全功能垃圾收集器 G1 **注**：监测工具可以自己选用，推荐使用 GCEasy、JDK 自带工具，Arthas，Grafana，Prometheus



**JVM 配置参数的素材：**

```Bash
#JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256 -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
#JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${BASE_DIR}/logs/java_heapdump.hprof"
#JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages"

# 吞吐量优先策略： 
JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m - Xss512k"
JAVA_OPT="${JAVA_OPT} -XX:+UseParallelGC -XX:+UseParallelOldGC "
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps - XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-ps- po.log"

# 响应时间优先策略 
#JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m - Xss512k" 
#JAVA_OPT="${JAVA_OPT} -XX:+UseParNewGC -XX:+UseConcMarkSweepGC "
#JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps - XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-parnew- cms.log" 

# 全功能垃圾收集器 
#JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -XX:MetaspaceSize=128m -Xss512k"
#JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:MaxGCPauseMillis=100"
#JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps - XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-g- one.log"
```



## 答：



加压接口：http://ip:9001/spu/goods/slow/10000023827800

加压计划：

