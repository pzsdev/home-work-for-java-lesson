

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

## 测试环境

**加压接口**：http://ip:9001/spu/goods/slow/10000023827800

**JVM 数据收集环境**：五台阿里云 VPS，4核 8 G配置 。

一台服务器跑 Java 应用，一台服务器跑 Jmeter 作为 Slave 节点，第三台跑 MySQL，第四台跑 Prometheus、Grafana、InfluxDB，第五台 Window Server 作为 Jmeter Master 控制和收集加压数据，全部通过局域网进行通信，内网带宽最高 10 Gbps。

**压测计划**：梯度压测，1000、2000、3000 线程组，Ramp-up peiod 为 1S，Loop Count 为 1000，总请求数 600 0000。



![](./week-three-test-plan.png)



> 说明：此处三个场景使用同一台服务器，连续收集三次数据，采集数据时间顺序和标题顺序正好相反，先跑了 G1，再跑响应时间优先，最后跑的吞吐量优先，也就是先情况 3，然后 2，1是最后收集的。



## 1、吞吐量优先 JVM 设置及 GC 情况

### 设置 JVM 参数：

```Bash
# 吞吐量优先策略： 
JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m -Xss512k " 
JAVA_OPT="${JAVA_OPT} -XX:+UseParallelGC -XX:+UseParallelOldGC "
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-ps-po.log" 
```

启动应用，参数生效：

![](test-section-one-001.png)



整个过程，Java 应用服务器资源情况：

![](test-section-one-002.png)

![](test-section-one-003.png)



### 线程数、RT、TPS、吞吐量：

![](test-section-one-004.png)

![](test-section-one-005.png)

![](test-section-one-006.png)



### GC 的统计信息、堆内存统计信息：

![](test-section-one-007.png)



![](test-section-one-008.png)



![](test-section-one-009.png)



### Arthas 工具数据：

![](test-section-one-010.png)



## 2、响应时间优先策略下  JVM 设置及 GC 情况

### 设置 JVM 参数：

```Bash
# 响应时间优先策略 
JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -Xmn125m -XX:MetaspaceSize=128m -Xss512k " 
JAVA_OPT="${JAVA_OPT} -XX:+UseParNewGC -XX:+UseConcMarkSweepGC "
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-parnew-cms.log"
```

启动应用，参数生效： 

![](test-section-two-001.png)



采集 Jmeter 数据：

![](test-section-two-002.png)



整个过程，Java 应用服务器资源情况：

![](test-section-two-003.png)



![](test-section-two-004.png)



### 线程数、RT、TPS、吞吐量：

![](test-section-two-005.png)



![](test-section-two-006.png)



![](test-section-two-007.png)



![](test-section-two-008.png)



### GC 的统计信息、堆内存统计信息：

![](test-section-two-009.png)



![](test-section-two-010.png)



### Arthas 工具数据：

![](test-section-two-011.png)



## 3、使用全功能垃圾收集器 G1 时  JVM 设置及 GC 情况

### 设置 JVM 参数：

```Bash
# 全功能垃圾收集器 
JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -XX:MetaspaceSize=128m -Xss512k "
JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:MaxGCPauseMillis=100"
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-g-one.log"
```



启动应用，参数生效：

![](test-section-three-001.png)



采集 Jmeter 数据：

![](test-section-three-002.png)



### RT、TPS、吞吐量：

![](test-section-three-003.png)



![](test-section-three-004.png)



![](test-section-three-005.png)



### GC 的统计信息、堆内存统计信息：

![](test-section-three-006.png)



![](test-section-three-007.png)