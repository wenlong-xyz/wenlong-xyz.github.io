---
title: OSCache分布式集群配置
date: 2015-08-01 10:23:26
tags: [Cache,分布式]
category: Technology
toc: true
---
&emsp;&emsp;之前在IBM实习时，由于licence的原因，项目使用OScCache进行数据缓存。单机部署项目测试时，未出现什么问题，但多机测试时出现了问题：OSCache频繁刷新，缓存效果下降。由于OSCcache已经停止更新，OSCache集群方面资料较少，通过一周的学习和探索，终于找到了解决方案。

# OSCache简介
* 缓存任何对象
* 永久缓存，缓存持久化处理
* 缓存记录的过期
* 支持集群（需要额外配置）
<div align="center">
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/OSCache-cluster.png?raw=true" width="300" />
</div>

# OSCache 集群支持介绍
&emsp;&emsp;OScache自身不具备集群通信能力，它他需要借助第三方通讯工具来实现集群缓存数据同步。下图所示是OScache集群相关的Java类，其中

- AbstractBroadcastingListener负责缓存同步事件的下发和处理
- 类ClusterNotification为组播消息体（Bean）
- 类JavaGroupsBroadcastingListener/ JMS10BroadcastingListening/ JMSBroadcastingListening由OSCache官方提供，利用JGroups通信框架或JMS通信框架的组播能力实现OSCache的缓存数据同步需求。

<div align="center">
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-2.png?raw=true" width="300" />
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-3.png?raw=true" width="350" />
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-4.png?raw=true" width="500" />
</div>

# OOSCache集群同步解决方案
1. OSCache+JGroups（本文介绍）
2. OSCache+JMS（略）
3. OSCache+其他的通信框架（略）
<div align="center">
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-5.png?raw=true" width="500" />
</div>
下面介绍OSCache+JGroups的配置方式
<div align="center">
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-6.png?raw=true" width="600" />
</div>

JavaGroupsBroadcastingListener

- initialize(Cache cache, Config config)  -- 加载通信基本配置
- sendNotification(ClusterNotification message) -- 发送组播消息
- handleNotification(Serializable serializable)  --  处理接收到的接收组播消息

## 默认配置
1. 我使用的Jar包：jgroups-2.12.3.Final.jar(3.*以上JGroups不支持)，concurrent-1.3.4.jar，oscache-2.4.1.jar，log4j-1.2.17.jar，commons-logging-1.2.jar(OScache中使用的log)
2. 添加文件src/oscache.properties（log4j.properties也可以放在src目录下），添加以下内容
```bash
cache.event.listeners=com.opensymphony.oscache.plugins.clustersupport.JavaGroupsBroadcastingListener  -- 组播事件监听器
cache.memory=true -- 是否使用内存
cache.blocking=true -- 是否同步
cache.cluster.properties=
	UDP(mcast_addr=231.12.21.132;mcast_port=45566;ip_ttl=32;\
	mcast_send_buf_size=150000;mcast_recv_buf_size=80000):\
	PING(timeout=2000;num_initial_members=3):\
	MERGE2(min_interval=5000;max_interval=10000):\
	FD_SOCK:VERIFY_SUSPECT(timeout=1500):\
	pbcast.NAKACK(gc_lag=50;retransmit_timeout=300,600,1200,2400,4800;max_xmit_size=8192):\
	UNICAST(timeout=300,600,1200,2400):\
	pbcast.STABLE(desired_avg_gossip=20000):\
	FRAG(frag_size=8096;down_thread=false;up_thread=false):\
	pbcast.GMS(join_timeout=5000;join_retry_timeout=2000;shun=false;print_local_addr=true)
cache.cluster.multicast.ip=231.12.21.132 -- 组播IP
```
### 默认配置的问题：
刷新动作（数据库增删改）同步，但缓存数据不同步。

<div align="center">
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-7.png?raw=true" width="600" />
</div>

### 缓存数据不同步原因：
OSCache所提供的AbstractBroadcastingListener类中只有刷新消息的发送、接收和处理逻辑，无实时同步逻辑。

<div align="center">
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-8.png?raw=true" width="500" />
</div>

## 缓存数据同步配置

1. 增加类
	- JavaGroupsSyncBroadcastingListener

		- 实现方法（发消息）：
			- cacheEntryAdded
			- cacheEntryRemoved
			- cacheEntryUpdated
		- 扩充方法（收消息）：
			- handleClusterNotification
	- CacheConstants  --  添加事件类型常量
	- SerialCacheEvent  -- 可序列化的事件类

2. 修改配置文件oscache.properties：

- cache.event.listeners=JavaGroupsSyncBroadcastingListener（Package完整路径）

<div align="center">
<img src="https://raw.githubusercontent.com/ZongWenlong/ZongWenlong.github.io/master/image/oscache-9.png?raw=true" width="500" />
</div>

### 代码列表

- JavaGroupsSyncBroadcastingListener
```java
import java.util.Date;  
import java.util.Set;  
  
import org.apache.commons.logging.Log;  
import org.apache.commons.logging.LogFactory;  
  
import com.opensymphony.oscache.base.events.CacheEntryEvent;  
import com.opensymphony.oscache.plugins.clustersupport.ClusterNotification;  
import com.opensymphony.oscache.plugins.clustersupport.JavaGroupsBroadcastingListener;  
  
/** 
* 自定义BroadCastingListener,重写父类方法。 
*/  
public class JavaGroupsSyncBroadcastingListener extends  
          JavaGroupsBroadcastingListener {  
     private final static Log log = LogFactory  
               .getLog(JavaGroupsBroadcastingListenerImpl.class);  
  
     @SuppressWarnings("deprecation")  
     @Override  
     public void handleClusterNotification(ClusterNotification message) {  
  
          if (cache == null) {  
               log.warn("A cluster notification ("  
                         + message  
                         + ") was received, but no cache is registered on this machine. Notification ignored.");  
  
               return;  
          }  
  
          if (log.isInfoEnabled()) {  
               log.info("Cluster notification (" + message + ") was received.");  
          }  
  
          switch (message.getType()) {  
          case ClusterNotification.FLUSH_KEY:  
            cache.flushEntry((String) message.getData(), CLUSTER_ORIGIN);  
            break;  
        case ClusterNotification.FLUSH_GROUP:  
            cache.flushGroup((String) message.getData(), CLUSTER_ORIGIN);  
            break;  
        case ClusterNotification.FLUSH_PATTERN:  
            cache.flushPattern((String) message.getData(), CLUSTER_ORIGIN);  
            break;  
        case ClusterNotification.FLUSH_CACHE:  
            cache.flushAll((Date) message.getData(), CLUSTER_ORIGIN);  
            break;      
          case CacheConstants.CLUSTER_ENTRY_ADD:  
               log.info("Cluster data add (" + message + ") ");  
               if (message.getData() instanceof SerialCacheEvent ) {  
                    SerialCacheEvent event = (SerialCacheEvent ) message.getData();  
                    cache.putInCache(event.getKey(), event.getEntry().getContent(),setToArray(event.getEntry().getGroups()), null, CLUSTER_ORIGIN);  
               }  
               break;  
          case CacheConstants.CLUSTER_ENTRY_UPDATE:  
               log.info("Cluster data update (" + message + ") ");  
               if (message.getData() instanceof SerialCacheEvent ) {  
                    SerialCacheEvent event = (SerialCacheEvent ) message.getData();  
                    cache.putInCache(event.getKey(), event.getEntry().getContent(),setToArray(event.getEntry().getGroups()), null, CLUSTER_ORIGIN);  
               }  
               break;  
          case CacheConstants.CLUSTER_ENTRY_DELETE:  
               log.info("Cluster data delete (" + message + ") ");  
               if (message.getData() instanceof SerialCacheEvent ) {  
                    SerialCacheEvent event = (SerialCacheEvent ) message.getData();  
                    cache.removeEntry(event.getKey());  
               }  
               break;  
          default:  
               log.error("The cluster notification (" + message  
                         + ") is of an unknown type. Notification ignored.");  
          }  
     }  
  
     @Override  
     public void cacheEntryAdded(CacheEntryEvent event) {  
          log.info("attribute data add (" + event.getKey() + ") ");  
          super.cacheEntryAdded(event);  
          if (!CLUSTER_ORIGIN.equals(event.getOrigin())) {  
               sendNotification(new ClusterNotification(  
                         CacheConstants.CLUSTER_ENTRY_ADD, new SerialCacheEvent (  
                                   event.getMap(), event.getEntry(), CLUSTER_ORIGIN)));  
          }  
     }  
  
     @Override  
     public void cacheEntryRemoved(CacheEntryEvent event) {  
          log.info("attribute data delete (" + event.getKey() + ") ");  
          super.cacheEntryRemoved(event);  
          if (!CLUSTER_ORIGIN.equals(event.getOrigin())) {  
               sendNotification(new ClusterNotification(  
                         CacheConstants.CLUSTER_ENTRY_DELETE, new SerialCacheEvent (  
                                   event.getMap(), event.getEntry(), CLUSTER_ORIGIN)));  
          }  
     }  
  
     @Override  
     public void cacheEntryUpdated(CacheEntryEvent event) {  
          log.info("attribute data update (" + event.getKey() + ") ");  
          super.cacheEntryUpdated(event);  
          if (!CLUSTER_ORIGIN.equals(event.getOrigin())) {  
               sendNotification(new ClusterNotification(  
                         CacheConstants.CLUSTER_ENTRY_UPDATE, new SerialCacheEvent (  
                                   event.getMap(), event.getEntry(), CLUSTER_ORIGIN)));  
          }  
     }  
      
      
     private String[] setToArray(Set set){  
          String[] strArray = new String[set.size()];  
           
          int i = 0;  
          for(Object str : set){  
               strArray[i] = (String) str;  
               i++;  
          }  
          return strArray;  
     }  
  
}  
```

- CacheConstants 
```java
 public final static int FLUSH_PATTERN = 3;
      /**
       * 刷新缓存对象
       */
      public final static int FLUSH_CACHE = 4;

      /**
       * 集群entry add处理
       */
      public final static int CLUSTER_ENTRY_ADD = 20;

      /**
       * 集群entry update处理
       */
      public final static int CLUSTER_ENTRY_UPDATE = 21;

      /**
       * 集群entry delete处理
       */
      public final static int CLUSTER_ENTRY_DELETE = 22;
      
}
```

- SerialCacheEvent 
```java
import java.io.Serializable;

import com.opensymphony.oscache.base.Cache;
import com.opensymphony.oscache.base.CacheEntry;

public class SerialCacheEvent  implements Serializable{

      /**
       *
       */
      private static final long serialVersionUID = -649226025117113267L;

      /**
       * The cache where the entry resides.
       */
      private Cache map = null;

      /**
       * The entry that the event applies to.
       */
      private CacheEntry entry = null;
      
      private String origin = null;
      /**
       * Constructs a cache entry event object with no specified origin
       *
       * @param map
       *            The cache map of the cache entry
       * @param entry
       *            The cache entry that the event applies to
       */
      public  SerialCacheEvent  (Cache map, CacheEntry entry) {
             this( map, entry, null);
      }

      /**
       * Constructs a cache entry event object
       *
       * @param map
       *            The cache map of the cache entry
       * @param entry
       *            The cache entry that the event applies to
       * @param origin
       *            The origin of this event
       */
      public  SerialCacheEvent  (Cache map, CacheEntry entry, String origin) {
             this. map = map;
             this. entry = entry;
             this. origin = origin;
      }

      /**
       * Retrieve the cache entry that the event applies to.
       */
      public CacheEntry getEntry() {
             return entry;
      }

      /**
       * Retrieve the cache entry key
       */
      public String getKey() {
             return entry.getKey();
      }

      /**
       * Retrieve the cache map where the entry resides.
       */
      public Cache getMap() {
             return map;
      }

      public String getOrigin() {
             return origin;
      }

      public void setOrigin(String origin) {
             this. origin = origin;
      }
      
      
}
```

# 遗留问题

1. OSCache：已停止更新，集群方面存在问题（集群规模不能太大等），是否可以考虑使用其他缓存框架。
2. JGroups：目前使用的JGroups版本较低，其中一些类已经废弃。高版本有更优、更稳定的性能，是否有必要根据高版本JGroups重写JavaGroupsBroadcastingListener还需要考量。如果想引入其他通信框架，重写JavaGroupsBroadcastingListener即可。

# 参考

1. [oscache分布式缓存](http://blog.csdn.net/laven90/article/details/9567499)JavaGroupsBroadcastingListenerImpl源码存在错误，修改见本文JavaGroupsSyncBroadcastingListener
2. [Oscache分布式集群配置总结](http://3001448.blog.51cto.com/2991448/1202879)


