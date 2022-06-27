## 1.大数据:
### 1-1.定义:
    是指利用常用软件工具捕获,管理和处理数据所耗时间超过可容忍时间的数据集(维基百科).
    Volume体量巨大,Velocity处理速度快,Variety类型繁多,Value价值密度低.
### 1-2.应用:
    金融,教育,政府公共安全,交通规划,清洁能源
### 1-3.主要计算模式:
    批处理计算(大规模数据的批量),流计算(流数据的实时计算),图计算(大规模图结构数据),
    查询分析计算(大规模数据的存储管理和查询分析)
## 2.HDFS
### 2-1概述:
    在商品硬件上运行的分布式文件系统,具有高度的容错能力,高通吐量访问,实现流式访问
### 2-2相关概念:
    NameNode:  文件名,存储元数据,元数据存在内存中,block与datanode的映射关系(唯一性)
               =Editlog(对元数据的修改信息)+Fsimage(文件系统数,文件夹,文件元数据信息)
    DataNode:  数据节点,存储文件内容,文件内容保存在磁盘,block id与datanode与本地文件的映射关系
    Client:  客户端,主要包含HDFS的一些接口,用于访问HDFS上的文件
#### Block-块:
    HDFS默认一个块128MB,一个文件可被分成多个块,以快作为存储单位
    块的大小远远小于普通文件系统,可以最小化寻址开销
#### 抽象块概念的好处:
    支持大规模文件存储,  简化系统设计,  适合数据备份
### 2-3 体系架构:
#### 概述:
   ![](https://user-images.githubusercontent.com/81810940/175805852-3e39fc9f-eff6-4fdb-8b2e-7aa90e85db0e.png)
#### 命名空间管理:
    命名空间包含目录,文件和块
    NameNode维护文件系统命名空间,对文件系统命名空间或其属性的任何更改均由NameNode记录
#### 单名称节点体系结构的局限性:
    1.命名空间的限制:名称节点式保存在内存中的,因此,名称节点能够容纳的对象(文件,块)的个数会受到内存空间大小的限制
    2.性能的瓶颈:整个分布式文件系统的吞吐量,受限于单个名称节点的吞吐量
    3.隔离问题:由于集群中只有一个名称节点,只有一个命名空间,因此,无法对不同应用程序进行隔离
    4.集群的可用性:一旦这个唯一的名称节点发生故障,会导致整个集群变得不可用
### 2-4 关键特性介绍:
#### 高可用性(HA):
    双节点:NameNode(active)主节点,NameNode(stand by)二备份节点
##### 图形概述:
    ![](https://user-images.githubusercontent.com/81810940/175807308-895b2245-cec4-44d7-b0cc-e242baf3a785.png)
##### 文字概述:
    同时运行两个Namenode，一个作为活动的Namenode（Active），一个作为备份的Namenode（Standby）。备份的Namenode的命名空间与活动的Namenode是实时同步的，所以当活动的Namenode发生故障而停止服务时，备份Namenode可以立即切换为活动状态，而不影响HDFS集群服务。
    ![](https://user-images.githubusercontent.com/81810940/175806673-e8c7e6c0-83df-45de-8481-ecb9c20884f5.png)
#### 元数据持久化:
    SecondaryNamenode.
##### 图形概述:
    ![](https://user-images.githubusercontent.com/81810940/175807053-f2103f4c-14df-450c-a8f9-799514dbdf99.png)
##### 文字概述:
    1、主namenode接收文件系统操作请求，生成editlog，并回滚日志，向editlog.new中记录日子
    2、备用namenode从主namenode上下载FSImage，并从共享存储中读取editlog
    3、备用namenode将日志和旧的元数据合并，生成新的元数据FSImage.ckpt
    4、备用namenode将元数据上传到主用namenode
    5、主namenode将上传的元数据进行回滚
    6、循环操作
#### HDFS联邦(Federation):
##### 图形概述:
    ![image](https://user-images.githubusercontent.com/81810940/175807405-697bf7b2-5407-4e5d-9a60-79ff466894fe.png)
##### 文字概述::
    Federation使用多组独立的Namenodes/Namespaces。所有的Namenodes是联邦的，也就是说，他们之间相互独立且不需要互相协调，各自分工，管理自己的区域。Datanode被用作通用的数据块存储设备，每个DataNode要向集群中所有的Namenode注册，且周期性的向所有Namenode发送心跳和块报告，并执行来自所有Namenode的命令。
    1.这些namenode直接相互独立，各自分工管理自己的区域，且不需要互相协调，一个namenode挂掉了不会影响其他的namenode
    2.datanode被用作通用的数据存储设备，每个datanode要向集群中所有的namenode注册，且周期性的向所有namenode发送心跳和报告，并执行来自所有namenode的命令
    3.一个block pool由属于同一个namespace的数据块组成，每个datanode可能会存储集群中所有block pool 数据块每个block pool内部自治，各自管理各自的block，不会与其他block pool交流
    4.namenode和block pool一起被称作namespace volume，它是管理的基本单位，当一个namespace被删除后，所有datanode上与其对应的block pool也会被删除。当集群升级时，每个namespace volume作为一个基本单元进行升级 
#### 


    
    
