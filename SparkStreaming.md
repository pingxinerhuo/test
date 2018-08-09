<font size=4><center>目录</center>

<font size=3>

[toc]

</font>

</br>

### 一、引入SparkStreaming

<font size=3>

**SparkCore:**</br>
核心抽象：RDD</br>
程序入口：new SparkContext</br>
代码开发：</br>
&emsp; &emsp; &emsp; &emsp;      通过程序入口生成RDD</br>
&emsp; &emsp; &emsp; &emsp;        RDD.transforamt 或者action的操作</br>

**SparkSQL:**</br>
核心抽象：DataFrame/DataFrame</br>
程序入口：</br>
&emsp; &emsp; &emsp; &emsp;          Spark1.x.x:  SQLContext(sc)/HiveCotext(sc)</br>
&emsp; &emsp; &emsp; &emsp;          Spark2.x.x:  SparkSession(sc)</br>
代码开发：通过程序入口生成DataFrame/DataSet</br>
&emsp; &emsp; &emsp; &emsp;  注册成为一个张表/  使用DataFrame或者DataSet API开发</br>
&emsp; &emsp; &emsp; &emsp;  SQL </br>

**SparkStreaming:**  </br>
核心抽象：DStream（RDD）</br>
程序入口：new StreamingContext(sc,<font color=red>**Seconds(2)**</font>) </br>
代码开发：通过程序入口生成DStream  </br>
&emsp; &emsp; &emsp; &emsp;   DStream.<font color=red>**Transformation**</font>   和<font color=red>**output**</font>（类似于RDD的action操作） </br>

</font>

</br>

### 二、SparkStreaming的运行流程

![SparkStreaming的运行流程](85FA5F4ACDD347D7932CD4692DF53DC9) </br>


1. 首先初始化的是SparkContext程序入口，接着初始化StreamingContext
2. Driver会发送Receivers(接收器)长时间运行在Executor里面(Receiver可以是一个，也可以是多个，如果需要多个，需要我们写代码设置，当Receiver启动起来运行的时候，其实就是一个task任务)
3. 每个Receiver都会接受输入进来的数据，并且把这些数据按照一定的规则生成block块（依据是什么？）然后把数据写入到Executor的内存里面，还需要再找另外一个Executor存另外的一个副本</br>
**依据:** task的并行度：BlockInterval:200ms (spark.streaming.blockInterval) 
4. 生成的这些block信息会由Receiver发送给StreamingContext
5. SparkStreaming会根据一定的时间间隔，把一定时间内的block（依据是什么？）抽象成为一个RDD，接下来代码运行 </br>
**依据:** new StreamingContext(sc,Seconds(2)  BatchInterval




> <font size=3 color=black face="Arial Black">spark.streaming.blockInterval,设置<font color=red>**blockInterval**</font>，把一定时间内（默认是200ms）的数据生成一个block  </br> </br>
> new StreamingContext(sc,<font> color=red>**Seconds(2)**</font>)，设置<font color=red>**batchInterval**</font>，把一定时间内（这里设置的是2秒）的block抽象成一个RDD</br>
</font>

</br>
</br>

### 三、SparkStreaming的三个部分

<font size=3>

一个实时处理的应用程序，应该由三部分构成  </br>


数据的输入 | 数据的处理 | 数据的输出
---|---|---
<font color=red>**kafak：最最最重要的数据源**</font></br>Flume</br>socket</br>HDFS</br>kinesis</br>| 涉及到transformation的API | <font color=red>**HBase</br>Redis</br>kafka</br>Mysql/Oracle**</font></br>Dashboards

</font>

<img src="http://spark.apache.org/docs/latest/img/streaming-arch.png" width=70% >

</br>
</br>


### 四、Storm和SparkStreaming

<font size=3>

**1、延迟和吞吐量**</br>
&emsp;  **Storm:** </br>
&emsp; 低延迟的，一次处理一条数据，真正意义上的实时处理。吞吐量小一点。 </br>

&emsp;**SparkStreaming：** </br>
 &emsp; 微批处理，稍微有一些延迟性，一次处理的是一批数据，其实不是真正意义上的实时处理，准实时处理。</br>
  &emsp; 吞吐量要大一点。</br>

**2、容错性**</br>
&emsp; SparkStreaming: 容错性做得非常好。Driver，Executor，Receiver，Task，Block数据丢失怎么办？（基于RDD）</br>

&emsp; Storm：storm的容错是基于ack组件完成。开销要比SparkSreaming大 </br>

**3、事务性**</br>
&emsp; 实时处理的语义： </br>
&emsp; 最多处理一次：丢数据  </br>
&emsp; 至少处理一次：有可能重复处理  </br>
&emsp; 处理且只被处理一次：不会丢数据，也不会重复处理数据。  </br>

&emsp; SparkStreaming 可以实现，难度比Storm小一点 </br>
&emsp; Storm也可以实现，需要我们自己写代码去把控。</br>

**4、生态系统**</br>
&emsp;  SparkStreaming是基于Spark的，Spark本身就有非常成熟的系统。（SparkCore,SparkSQL,图计算，机器学习等等）。这才是SparkStreaming 最牛的地方。</br>
&emsp;  Storm，只是一个单独的流式计算框架而已。没有完整的生态系统。



</font>

</br>
</br>

### 五、Dstream(离散流)

#### 1、什么是Dstream
&emsp; &emsp; RDD流

![DStream](F7A5794E95C7453D95413AD1F24CAA64)



#### 2、Dstream操作
![DStream操作 ](5F24D2AFF1DA4E419638BA6CD77B5435)

<font size=4>

```scala
DStream.scala 源码
generatedRDDs就是一个DStream，包含多个RDD(一个时间点对应一个RDD)
  // RDDs generated, marked as private[streaming] so that testsuites can access it
  @transient
  private[streaming] var generatedRDDs = new HashMap[Time, RDD[T]]()

```
</font>

</br>
</br>

### 六、代码实例

#### 1、快速例子运行(wordCount)

<font size=4>

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object NetWordCount {
  def main(args: Array[String]): Unit = {

    // 1.初始化一个程序入口
    /*  如果是本地运行线程数至少要设置为2，因为：
     *  Receiver启动起来表现为一个task任务，运行的时候需要一个线程
     *  如果设置local[1],那么只有一个线程接收数据，没有线程处理数据
     *  task  另外一个线程处理数据
     */
    val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
    val ssc = new StreamingContext(conf, Seconds(2))

    // 2.通过程序入口获取DStream
    // Create a DStream that will connect to hostname:port, like localhost:9999
    val dstream : ReceiverInputDStream[String] = ssc.socketTextStream("hadoop", 9999)

    // 3.对DStream流进行操作
    val wordCountDStream : DStream[(String,Int)] = dstream.flatMap(line => line.split(",")).map((_,1)).reduceByKey(_+_)

    wordCountDStream.print()

    // 4.启动应用程序
    ssc.start()
    ssc.awaitTermination()  //等待结束
    ssc.stop()
  }
}

```
</font>

</br>

<font size=3>**在hadoop上输入nc -lk 9999，然后输入传输的数据** </font>

<font size=4>

```
[hadoop@hadoop ~]$ nc -lk 9999
hadoop,hadoop,hadoop
spark,spark
haha,haha
spark,spark,spark,spark,spark,spark

```
</font>

</br>

#### 2、HDFSWordCount 

> <font size=3 color=black>监控某个HDFS文件夹，上传文件到该目录，对该上传文件的内容进行单词统计</font>

<font size=4>

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.DStream
import org.apache.spark.streaming.{Seconds, StreamingContext}

object HDFSWordCount {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
    val ssc = new StreamingContext(conf, Seconds(2))

    // 1.数据的输入
    
    // 监控/streaming目录
    val fileDStream : DStream[String] = ssc.textFileStream("hdfs://myha01/streaming")

    // 2.数据的处理
    val wordCountDStream: DStream[(String, Int)] = fileDStream.flatMap(_.split(",")).map((_,1)).reduceByKey(_+_)

    // 3.数据的输出
    wordCountDStream.print()

    ssc.start()
    ssc.awaitTermination()
    ssc.stop()
  }
}

```
</font>

</br>

#### <font color=red>3、updateStateByKey(transformation算子)</font>

> <font size=3 color=black>updateStateByKey 实现wordCount的全局累加</font>

<font size=4>

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object UpdateStateBykeyWordCount {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
    val ssc = new StreamingContext(conf, Seconds(2))

    //这个checkpoint目录可以不必事先创建，但一定要有权限
    ssc.checkpoint("hdfs://myha01/streaming/streamingcheckpoint")

    // 数据的输入（这里从socket读数据）
    val dstream : ReceiverInputDStream[String] = ssc.socketTextStream("hadoop",9999)

    // 数据的处理

    /* updateStateByKey(),需要的一种参数：updateFunc:(Seq[V],Option[S]) => Option[S]
     * Options:
     * Some: 有值
     * None: 没有值
     *
     * 数据的输入：
     * you,1
     * you,1
     * jump,1
     *
     * ByKey:分组
     * you,[1,1]
     * jump,[1]
     *
     * (values:Seq[Int],state:Option[Int])=>{}  每个key都调用一次该匿名函数
     *
     * values:Seq[Int]
     * you调用上面的函数，对于you来说 values代表[1,1]
     *
     * state:Option[Int]  上一次这个单词出现了多少次 None Some 2
     */
     val wordCountDStream: DStream[(String, Int)] = dstream.flatMap(_.split(","))
        .map((_, 1))
        .updateStateByKey((values: Seq[Int], state: Option[Int]) => {
      val currentCount = values.sum

      /* getOrElse()
         Returns the option's value if the option is nonempty,
         otherwise return the result of evaluating `default`
       */
      // 上一次出现的次数
      val lastCount = state.getOrElse(0)

      // updateStateByKey中的匿名函数有返回值：这个单词一共出现了多少次
      Some(currentCount + lastCount)
    })


    //数据的输出
    wordCountDStream.print()

    ssc.start()
    ssc.awaitTermination()
    ssc.stop()

  }
}
```


</font>

<font size=3>**本例还是需要在hadoop主机上用 nc -lk 9999 通过socket 模拟出数据** </font>

<font size=4>


```
[hadoop@hadoop ~]$ nc -lk 9999
spark,hadoop,spark
spark,hadoop,hadoop
you,jump
i,jump

```

</font>

</br>

#### <font color=red>4、DriverHAWordCount（checkpoint的应用）</font>

> <font size=3 color=black>**上一个代码的问题是：** 如果关闭程序（手动或出异常），再次开启程序，上一次处理的结果就没有了，所有的数据都将重新计数</br>
**问题的原因在于**，我们重新启动程序，会创建一个新的程序入口（new StreamingContext），即两次启动的是不同的Driver服务</br>
**解决思路：** 再次启动的时候恢复上一次Driver服务的信息/数据，而checkpoint的目录文件中存储了driver服务的信息，下一次启动的时候可以通过checkpoint中的数据恢复上一次的Driver服务，就可以对上一次运行的结果进行操作
</font>



</font>

<font size=4>

```scala
和上一个代码pdateStateBykeyWordCount的逻辑基本一样，可以全部复制到方法functionToCreateContext()中，在该方法外
主要是以下代码：
    val checkpointDirectory:String = "hdfs://myha01/streaming/streamingcheckpoint"
    
    再次获取 StreamingContext
    val ssc = StreamingContext.getOrCreate(checkpointDirectory, functionToCreateContext _)
 
    再次启动关闭 StreamingContext
    ssc.start()
    ssc.awaitTermination()
    ssc.stop()
    
==============================================================================

import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object DriverHAWordCount {
  def main(args: Array[String]): Unit = {
  
    val checkpointDirectory:String = "hdfs://myha01/streaming/streamingcheckpoint"

    // Function to create and setup a new StreamingContext
    def functionToCreateContext(): StreamingContext = {
      val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
      val ssc = new StreamingContext(conf, Seconds(2))

      ssc.checkpoint(checkpointDirectory)

      // 数据的输入（这里从socket读数据）
      val dstream : ReceiverInputDStream[String] = ssc.socketTextStream("hadoop",9999)

      val wordCountDStream: DStream[(String, Int)] = dstream.flatMap(_.split(","))
        .map((_, 1))
        .updateStateByKey((values: Seq[Int], state: Option[Int]) => {
       
        val currentCount = values.sum
        val lastCount = state.getOrElse(0)
        Some(currentCount + lastCount)
        
      })

      //数据的输出
      wordCountDStream.print()

      ssc.start()
      ssc.awaitTermination()
      ssc.stop()

      // 注意要返回ssc
      ssc
    }
   
    // 从checkpoint获得程序入口StreamingContext或者重新获得一个
    val ssc = StreamingContext.getOrCreate(checkpointDirectory, functionToCreateContext _)

    // Do additional setup on context that needs to be done,
    // irrespective of whether it is being started or restarted
    ssc.start()
    ssc.awaitTermination()
    ssc.stop()

  }
}
```

</font>

</br>


#### <font color=red>5、transform（transformation算子）</font>

<font size=4>

```scala
transform源码

/**
   * Return a new DStream in which each RDD is generated by applying a function
   * on each RDD of 'this' DStream.
   */
  def transform[U: ClassTag](transformFunc: RDD[T] => RDD[U]): DStream[U] = ssc.withScope {
    // because the DStream is reachable from the outer object here, and because
    // DStreams can't be serialized with closures, we can't proactively check
    // it for serializability and so we pass the optional false to SparkContext.clean
    val cleanedF = context.sparkContext.clean(transformFunc, false)
    transform((r: RDD[T], _: Time) => cleanedF(r))
  }
  
  transform操作（以及它的变化形式如transformWith）允许在DStream运行任何RDD-to-RDD函数。
  它能够被用来应用任何没在DStream API中提供的RDD操作（It can be used to apply any RDD operation that is not
  exposed in the DStream API）。
  
```
</font>

> <font size=3 color=black>通过transform，实现黑名单效果（过滤无效字符）的wordcount</font>


<font size=4>

```scala
import org.apache.spark.SparkConf
import org.apache.spark.broadcast.Broadcast
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}

/*
 * transform 把DStream转换成一个RDD，可以用sparkcore和sparkSQL（把RDD转换成Dataframe）编程
 */


object WordBlack {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
    val ssc = new StreamingContext(conf, Seconds(2))


    // 数据的输入
    val dstream : ReceiverInputDStream[String] = ssc.socketTextStream("hadoop",9999)


    /*自己模拟一个黑名单：
      不过注意：
      这个黑名单，一般情况下，不是我们自己模拟出来的，应该是从mysql数据库
      或者是Redis数据库，或者是HBase数据库里面读取出来的
     */
    val wordBlackList: RDD[(String, Boolean)] = ssc.sparkContext.parallelize(List("?", "!", "*"))
      .map(param => (param, true))

    // 不能直接把RDD设为广播变量，可以通过collect()方法获取RDD的结果，并将该结果广播出去
    // collect(),在Driver中使用，以数组的形式返回数据集RDD的所有元素（结果）
    val blackList: Array[(String, Boolean)] = wordBlackList.collect()

    // 考虑到黑名单是从数据库中获取，可以设置为广播变量
    val blackListBroadcast: Broadcast[Array[(String, Boolean)]] = ssc.sparkContext.broadcast(blackList)

    // 数据的处理
    val wordOneDStream: DStream[(String, Int)] = dstream.flatMap(_.split(",")).map((_,1))

    // transform需要有返回值,并且类型是RDD,会被转换为DStream，即返回RDD[string]，会转换成DStream[String]
    val wordCountDStream: DStream[(String, Int)] = wordOneDStream.transform(rdd => {

        //把拿到的广播变量还原，并通过parallelize创建RDD
        val filterRDD: RDD[(String, Boolean)] = rdd.sparkContext.parallelize(blackListBroadcast.value)
        
        // join data stream with spam information(垃圾信息) to do data cleaning
        val resultRDD: RDD[(String, (Int, Option[Boolean]))] = rdd.leftOuterJoin(filterRDD)

        /* (String, (Int, Option[Boolean])
         * String：word
         * Int：1
         * Option: 左外连接以左边为准，有可能join上，有可能join不上
         *         join上是some(true)，join不上是None
         *
         *  思路：
         *     我们应该要是join不上的，说白了要的是Option[Boolean]=None
         *     因为join上的Option[Boolean]=true ,这是我们对黑名单中的符号的设置
         *
         *     filter: true -代表我们要 ，如 filter(t => t>3)
         */

        // transform需要有返回值,返回值类型要是RDD
        val value: RDD[String] = resultRDD.filter(tuple => {
          tuple._2._2.isEmpty
        }).map(_._1)

        value  // 把单词返回  返回RDD[String],会转换成DStream[String]

    }).map((_, 1)).reduceByKey(_ + _)   DStream[String] => DStream[String,Int]



    // 数据的输出
    /*  def print(): Unit = ssc.withScope {
          print(10)
        }
     *  默认打印DStream里RDD里的前是个元素
     *
     *  也可以自定义打印个数，print(5)
     */
    wordCountDStream.print()

    ssc.start()
    ssc.awaitTermination()
    ssc.stop()

  }
}

主机hadoop通过nc -lk 9999 模拟数据

```

</font>

#### <font color=red>6、window操作</font>

![window操作更正 - 副本](02560E0DA6E64B669F5190F4BFA58A2B)

> <font size=3 color=black>实现每隔4秒,统计最近6秒的单词计数的情况</font>

> <font size=3 color=black>window操作最重要的API：<font color=red>**reduceByKeyAndWindow()**</font></font> </br>
 <font size=3 color=black><font color=red>**注意：窗口的大小和滑动的大小一定要是**</font> val ssc = new StreamingContext(conf, Seconds(2))
        中Seconds(2)，即<font color=red>**BatchInterval的倍数**</font>，因为它决定了RDD的个数</font>
        
<font size=4>

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.ReceiverInputDStream
import org.apache.spark.streaming.{Seconds, StreamingContext}


/*
   需求：每隔4秒,统计最近6秒的单词计数的情况

   最重要的API：reduceByKeyAndWindow(func, invFunc, windowLength,
                slideInterval, [numTasks])
 */

object WindowOperatorTest {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
    val ssc = new StreamingContext(conf, Seconds(2))


    // 数据的输入，到目前为止这个地方还没有跟生产进行对接（应该跟kafka对接）
    val dstream : ReceiverInputDStream[String] = ssc.socketTextStream("hadoop",9999)


    /* 数据的处理
     * 我们一直讲的是数据处理的算子（数据输入没有跟生产对接）
     * 这个地方的算子就是生产时候使用的算子
     *
     *
      def reduceByKeyAndWindow(
          reduceFunc: (V, V) => V,
          windowDuration: Duration,   窗口的大小  6
          slideDuration: Duration,    滑动的大小  4   每隔4秒统计最近6秒的数据
           numPartitions: Int      指定分区数  可以不写，不写就是spark默认的分区数
        ): DStream[(K, V)] = ssc.withScope {
        reduceByKeyAndWindow(reduceFunc, windowDuration, slideDuration,
        defaultPartitioner(numPartitions))

        注意：窗口的大小和滑动的大小一定要是 val ssc = new StreamingContext(conf, Seconds(2))
        中Seconds(2)，即BatchInterval的倍数，因为它决定了RDD的个数
      }
     */
     
    val resultWordCountDStream = dstream.flatMap(_.split(","))
      .map((_, 1))
      .reduceByKeyAndWindow((x: Int, y: Int) => x + y, Seconds(6), Seconds(4))


    // 数据的输出,这个地方我们还是没有跟生产对接起来，我们下一节课讲这个事
    resultWordCountDStream.print()

    ssc.start()
    ssc.awaitTermination()
    ssc.stop()

  }
}

主机hadoop通过nc -lk 9999 模拟数据

```

</font>


#### <font color=red>7、foreachRDD(func)（output算子(相当于action)）</font>

<font size=3>

print()这个output算子仅仅限于测试的时候使用</br>
**重点：Spark Streaming的foreachRDD运行在Driver端，而foreach和foreachPartion运行在Worker节点。**</br>
备注：对数据的向外输出，还是用foreach算子好，不要用Map算子，因为Map还要返回一个RDD。

另：为什么调用foreachRDD，用的是大括号？具体见本目录下md:Scala之小括号和花括号
</font>


##### <font size=3>误区一：在driver上创建连接对象（比如网络连接或数据库连接）</font>



> <font size=3 color=black>如果在driver上创建连接对象，然后在RDD的算子函数内使用连接对象，</br>
> <font color=red>那么就意味着需要将连接对象序列化后从driver传递到worker上。
> 而连接对象（比如Connection对象）通常来说是不支持序列化的</font>，</br>此时通常会报序列化的异常
> （serialization  errors）。<font color=red>因此连接对象必须在worker上创建，不要在driver上创建</font>。</font>



<font size=4>

```
dstream.foreachRDD { rdd=>
  val connection = createNewConnection()  // 在driver上执行
  rdd.foreach { record =>
    connection.send(record) // 在worker上执行
  }
}
```

</font>

##### <font size=3>误区二：为每一条记录都创建一个连接对象</font>


> <font size=3 color=black>通常来说，连接对象的创建和销毁都是很消耗时间的。因此频繁地创建和销毁连接对象，可能会导致降低spark作业的整体性能和吞吐量。</font>

<font size=4>

```
dstream.foreachRDD { rdd=>
  rdd.foreach { record =>
    val connection = createNewConnection()
    connection.send(record)
    connection.close()
  }
}
```
</font>

##### <font size=3>正确做法一：为每个RDD分区创建一个连接对象</font>

> <font size=3 color=black>比较正确的做法是：对DStream中的RDD，调用foreachPartition，对RDD中每个分区创建一个连接对象，使用一个连接对象将一个分区内的数据都写入底层MySQL中。这样可以大大减少创建的连接对象的数量</font>

<font size=4>

```
dstream.foreachRDD { rdd=>
  rdd.foreachPartition { partitionOfRecords=>
    val connection = createNewConnection()
    partitionOfRecords.foreach(record =>connection.send(record))
    connection.close()
  }
}
```
</font>

##### <font size=3>正确做法二：为每个RDD分区使用一个连接池中的连接对象</font>

<font size=4>

```scala
dstream.foreachRDD { rdd=>
  rdd.foreachPartition { partitionOfRecords=>
    // 静态连接池，同时连接是懒创建的
    val connection =ConnectionPool.getConnection()
    partitionOfRecords.foreach(record =>connection.send(record))
   ConnectionPool.returnConnection(connection)  // 用完以后将连接返回给连接池，进行复用
  }
}
```
</font>



### 七、SparkStreaming的容错性

####  1、Executor失败容错

<font size=3>

executor失败了，其上的数据会丢失。会自动再找另外一个executor再启那个task任务，并且找的是数据副本所在的那台机器。找到后还会再另外一台机器再次生成这个数据的副本</br>
  如果是包含Receiver的Executor失败了，会找到另外一个executor启动Receiver
  
</font> 

![SparkStrming容错性](0FE700D2CAEB4D85985735BF67232EC7)
  
#### 2、Driver失败容错

<font size=3>
失败了手动重启：见DriverHA代码</br>
失败了自动重启：在用spark-submit提交程序时，加参数 --supervise 并且在程序中使用DriverHA的代码（checkpoint）</br>
注意：如果是sparkcore，只需要加--supervise即可，但是问题是可能kill -9 不了这个应用程序，需要在UI界面关闭该程序</br>
&emsp;&emsp;&emsp;但sparkStreaming需要DriverHA的代码，因为不仅要启动Driver，更是要恢复以前的Driver，使得可以在以前的计算结果上继续计算
 </br></br>
另外还要再考虑一个问题，当Driver挂了，所有的Executor都要挂，那么内存中的数据也会跟着挂，即以前接收的数据会丢失</br></br>
解决办法：开启WAL（WriteAheadLog）机制，设置spark.streaming.receiver.writeAheadLog.enable=true，默认是false，因为写日志需要耗费一点性能。这个日志会写到checkpoint的目录里面。设置成功后，下次再启动会帮你恢复数据</br></br>

```
 # Run on a Mesos cluster in cluster deploy mode with supervise
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master mesos://207.184.161.138:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  http://path/to/examples.jar \
  1000
 ``` 
  
</font> 



#### 3、Recevier失败容错
<font size=3 >会找到另外一个executor启动Receiver



#### 4、当一个task任务运行的很慢，SparkStreaming怎么办？

<font size=3>
设置spark.speculation（推测机制）为true，它默认是false</br>
假设现在有10个task运行，其中有task运行的比较慢</br>
&emsp;&emsp;成功的task数 > 0.75 * 10 (75%的task已经运行成功了)    &emsp;&emsp; 0.75这个值是可以配的 spark,speculation.quantile=0.75 </br>
&emsp;&emsp;正在运行的task的运行时间 > 1.5 * 成功运行的task的平均时间  &emsp;&emsp;spark,speculation.multiplier=1.5</br>
它会认为当前机器有问题或者正在GC，会在其他Excutor重启该task任务</br>
为什么默认是false,因为如果是task发生了数据倾斜，无论怎么弄都会比较慢，这样来回切换反而是帮倒忙，所以要用这个配置，必须要确定没有发生数据倾斜</br></br>
 
</font> 

![当一个task运行比较慢](492DD7349FA64D058C861D6046DA2190)
			



 
### 八、流式计算的三种语义

<font size=3>

流计算语义的定义：</br>
    针对的是每一条记录被处理的多少次 </br>
有三种语义：</br>
#### 1、At most once

一条记录要么被处理一次，要么就没处理，说白了就是会丢数据

#### 2、At least once
一条记录有可能被处理一次，也有可能被处理多次。
       有可能会重复处理数据
        
#### 3、Exactly once
一条记录处理且只被处理一次，非常完美的一个状态
        

</font>

偏移量不会实时记录，而是每隔一段时间记录偏移量，比如200ms记录一次。假如当100ms的时候出问题了，这100ms的数据已经被消费了，但是还没有到更新偏移量的时间，相当于回执还停留在200ms以前

</br>
</br>
</br>





