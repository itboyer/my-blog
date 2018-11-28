title: Redis的事务和消息机制
categories: Redis
top_img: /img/905584.jpg
tags:
  - redis
  - 事务
date: 2018-11-21 14:35:27
---
# Redis的事务和消息机制


![upload successful](/images/pasted-2.png)
<!-- more -->
## Java操作事务

  ```Java
  @Test
  	public void testTransaction(){
  		Jedis jedis = new Jedis("192.168.131.111", 6379);
  		
  		//开启事务
  		Transaction tc = null;
  		
  		try {
  			tc = jedis.multi();
  			tc.decrBy("tom",100);
  			tc.incrBy("mike", 100);
  			//提交事务
  			tc.exec();
  		} catch (Exception e) {
  			e.printStackTrace();
  			//回滚事务
  			tc.discard();
  		}
  		jedis.disconnect();
  	}
  ```

## Java操作锁

  ```Java
  @Test
  	public void testLock(){
  		
  		Jedis jedis = new Jedis("192.168.131.111", 6379);
  		//对ticket加锁，入伙在事务执行的过程中，该值有变化，则抛出异常
  		jedis.watch("ticket");
  		
  		Transaction tc = null;
  		try {
  			tc = jedis.multi();
  			tc.decr("ticket"); //车票刷量减1
  			
  			//如果在线程睡眠的过程中 ticket监控的值有变化，则抛出异常
  			Thread.sleep(10000);
  			tc.decrBy("tom", 100);//扣取买票钱
  			
  			//提交事务
  			tc.exec();
  			
  		} catch (Exception e) {
  			e.printStackTrace();
  			tc.discard();
  		}
  		
  	}
  ```


![upload successful](/images/pasted-3.png)

![redis Message](/images/pasted-4.png)

## Demo

* 使用Java程序实现消息的发布与订阅，需要继承JedisPubSub 类

  ```Java
  	@Test
  	public void testMessage(){
  		Jedis jedis = new Jedis("192.168.131.111", 6379);
  		
  		//subscribe 和 psubcribe 不能同时订阅
  		jedis.subscribe(new MyRedisMessageConsumer(), "channel");
  		//jedis.pshbscribe(new MyRedisMessageConsumer(),"channel");
  		
  	}	
  ```

  ```Java
  //实现Redis消息的接受者
  class MyRedisMessageConsumer extends JedisPubSub{

  	@Override
  	public void onMessage(String channel, String message) {
  		// 订阅某个频道的
  		System.out.println(channel+ "\t" + message);
  	}

  	@Override
  	public void onPMessage(String pattern, String channel, String message) {
  		// 用通配符匹配频道
  		System.out.println("pattern:"+pattern);
  		System.out.println("channel:"+channel);
  		System.out.println("message:"+message);
  	}
  	@Override
  	public void onPSubscribe(String arg0, int arg1) {}
  	@Override
  	public void onPUnsubscribe(String arg0, int arg1) {}
  	@Override
  	public void onSubscribe(String arg0, int arg1) {}
  	@Override
  	public void onUnsubscribe(String arg0, int arg1) {}
  	
  }
  ```