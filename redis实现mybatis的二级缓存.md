## Mybatis中的二级缓存

#### Myabtis 的二级缓存是如何实现的

~~~markdown
# 在Mybatis中开启二级缓存
在Mapper.xml文件中加标签<cache />  
开启二级缓存需要将实体类序列化，否则会出现错误
	Cause: java.io.NotSerializableException: com.jiang.travels.entity.User
~~~

~~~markdown
# Mybatis的二级缓存的实现
mybatis中有一个接口Cache。
mybatis二级缓存的实现类是PerpetualCache.java,实现的是接口Cache
~~~

![image-20210409091444602](https://gitee.com/Akihij/PicGo/raw/master/img/20210409091451.png)

![image-20210409091649035](https://gitee.com/Akihij/PicGo/raw/master/img/20210409091649.png)

~~~markdown
我们可以通过设置断点来证明PerpetualCache.java是二级缓存的实现类
在PerpetualCache.java中的getObject()方法设置一个断点
~~~

![image-20210409091938771](https://gitee.com/Akihij/PicGo/raw/master/img/20210409091938.png)

![image-20210409092142465](https://gitee.com/Akihij/PicGo/raw/master/img/20210409092142.png)

~~~markdown
我们发现在程序停在了getObject()方法中。
此时我们可以知道key的值，和cacahe的size。
因此我们知道了Mybatis的二级缓存的实现类确实是PerpetualCache.java
~~~

#### 解析PerpetualCache.java

```java
//我们可以看到PerpetualCache非常的简单，实现的核心就是一个Map的数据结构
//但其实每个方法的执行是比较的复杂的，这个暂不分析，我们现在只需了解每个方法
public class PerpetualCache implements Cache {

  private final String id;
    
  private final Map<Object, Object> cache = new HashMap<>();

  public PerpetualCache(String id) {
    this.id = id;
  }

  @Override
  public String getId() {
    return id;
  }

  @Override
  public int getSize() {
    return cache.size();
  }

  @Override
  public void putObject(Object key, Object value) {
    cache.put(key, value);
  }

  @Override
  public Object getObject(Object key) {
    return cache.get(key);
  }

  @Override
  public Object removeObject(Object key) {
    return cache.remove(key);
  }

  @Override
  public void clear() {
    cache.clear();
  }

  @Override
  public boolean equals(Object o) {
    if (getId() == null) {
      throw new CacheException("Cache instances require an ID.");
    }
    if (this == o) {
      return true;
    }
    if (!(o instanceof Cache)) {
      return false;
    }

    Cache otherCache = (Cache) o;
    return getId().equals(otherCache.getId());
  }

  @Override
  public int hashCode() {
    if (getId() == null) {
      throw new CacheException("Cache instances require an ID.");
    }
    return getId().hashCode();
  }

}
```

~~~markdown
我们了解完PerpetualCache.java的实现类，知道了Mybatis的二级缓存的原理。那么我们通过redis来实现Mybatis的二级缓存就很简单了。
~~~

## redis实现Mybatis二级缓存

~~~markdown
我们已经知道了Mybatis二级缓存的原理，实现的是PerpetualCache.java。

我们可以自己来创建一个类，实现Cache接口。重写put，get方法。
但是我们怎么让Mybatis知道? 就像我们如何让Spring容器知道我们注册了一个类。

我们说过，只有在Mapper.xml文件中添加<Cache />标签才能使用二级缓存。而在<Cache />标签中有几个属性
~~~

![image-20210409094246879](https://gitee.com/Akihij/PicGo/raw/master/img/20210409094254.png)

~~~mark
其中type属性就是决定实现类。
默认    <cache type="org.apache.ibatis.cache.impl.PerpetualCache"/>
~~~

#### RedisCache

~~~markdown
到现在，我们基本上可以知道RedisCache的原理和实现

1. 创建一个RedisCache类，并implements Cache接口
2. 实现RedisCache中的方法
3. 在Mapper.xml文件中 type="XXXXX.RedisCache"
~~~

```java
/***
 * 自定义Redis缓存的实现
 */
public class RedisCache implements Cache {
    @Override
    public String getId() {
        return null;
    }

    @Override
    public void putObject(Object key, Object value) {

    }

    @Override
    public Object getObject(Object key) {
        return null;
    }

    @Override
    public Object removeObject(Object key) {
        return null;
    }

    @Override
    public void clear() {

    }

    @Override
    public int getSize() {
        return 0;
    }
}
```

#### id的作用

~~~markdown
RedisCache的基本框架就是这样。
我们与PerpetualCache.java进行对比，我们发现少了许多的东西。
1. private final String id;
2. private final Map<Object, Object> cache = new HashMap<>();

对于Map<> cache很容易理解
我们来分析id的含义
~~~

~~~markdown
# private final String id;
我们并不知道id的作用是什么，我们首先来使用这个Cache，来判断这个id是否是必须的。
注意更改实现类    <cache type="com.jiang.travels.cache.RedisCache"/>
~~~

![image-20210409100123846](https://gitee.com/Akihij/PicGo/raw/master/img/20210409100123.png)

~~~markdown
# 出现了错误：
错误中由这样一句话:
Base cache implementations must have a constructor that takes a String id as a parameter.  
必须有一个构造器带id。
此时我们依旧不知道这个id是什么用。
不过，既然出现了错误，我们就改:为RedisCache实现类添加常量id，并添加构造函数
private final String id;

public RedisCache(String id) {
   System.out.println("id = " + id);
   this.id = id;
}

@Override
public String getId() {
   return id;
}

我们打印这个id到底是什么。
此时并没有报错，并且数据已经从数据库中查到
~~~

![image-20210409100752912](https://gitee.com/Akihij/PicGo/raw/master/img/20210409100752.png)

![image-20210409100833215](https://gitee.com/Akihij/PicGo/raw/master/img/20210409100833.png)

~~~markdown
我们终于知道了这个id 的含义:这就是Mapper.xml文件中的namespace
~~~

![image-20210409101026749](https://gitee.com/Akihij/PicGo/raw/master/img/20210409101026.png)



#### 实现方法get put

```java
//我们先来判断这两个方法是否执行
//放入
@Override
public void putObject(Object key, Object value) {
    System.out.println("key = " + key);
    System.out.println("value = " + value);
}

//读取
@Override
public Object getObject(Object key) {
    System.out.println("key = " + key);
    return null;
}
```

~~~markdown
同时，我们为了让结果更加的简单明了，我们更改Test方法。
~~~

```java
@Test
public void findUser(){
    User user = new User();
    user.setPassword("123456");
    user.setUsername("admin");
    User byUser1 = userService.findByUser(user);
    System.out.println(byUser1);
    
    System.out.println("====================================");

    user.setPassword("123456");
    user.setUsername("admin");
    User byUser2 = userService.findByUser(user);
    System.out.println(byUser2);
}
```

![image-20210409102404794](https://gitee.com/Akihij/PicGo/raw/master/img/20210409102404.png)

~~~markdown
很明显，方法执行了。
我们根据方法分析，
我们执行了一个方法，userService.findByUser(user);
这个方法去读取缓存，即执行getObject()方法。但是缓存为命中。
所以执行putObject()方法，放入缓存，因此在日志中才打印出这么多的数据。
~~~



~~~markdown
# redis实现方法
我们知道如何通过java来操作redis。
--- RedisTemplate
如何不了解的可以查看一些文章
https://www.cnblogs.com/smartsmile/p/11633844.html

但是我们需要创建一个工厂，来获取这个redisTRedisTemplate
~~~

```java
//用来获取Springboot创建好的工厂
@Component
public class ApplicationContextUtils implements ApplicationContextAware {

    //工厂
    private static ApplicationContext applicationContext;

    //将创建好的工厂以参数形式传递给这个类
    @Override
    public void  setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    //提供在工厂中获取对象的方法
    public static Object getBean(String beanName){
        return applicationContext.getBean(beanName);
    }
}
```



~~~markdown
# put方法的实现
~~~

```java
//放入
@Override
public void putObject(Object key, Object value) {
    System.out.println("key = " + key);
    System.out.println("value = " + value);
    //通过application工具类获取redisTemplate
    RedisTemplate redisTemplate = (RedisTemplate) ApplicationContextUtils.getBean("redisTemplate");

    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setHashKeySerializer(new StringRedisSerializer());

    //将redishash类型作为缓存存储模型  key  hashkey  value
    redisTemplate.opsForHash().put(id.toString(),key.toString(),value);
}
```

~~~markdown
# 写完put方法之后，我们对它进行测试
首先我们可以看到在redis中我们没有任何数据
~~~

![image-20210409110346364](https://gitee.com/Akihij/PicGo/raw/master/img/20210409110353.png)

~~~markdown
# 执行单元测试
再次观察redis，我们发现redis中有了数据，并且key为id值，
但是我们发现，在日志中，缓存依旧没有命中，这是因为我们没有重写getObject()方法
~~~

![image-20210409110448039](https://gitee.com/Akihij/PicGo/raw/master/img/20210409110448.png)





~~~markdown
# get方法的实现
~~~

```java
//读取
@Override
public Object getObject(Object key) {
    System.out.println("key = " + key);

    //通过application工具类获取redisTemplate
    RedisTemplate redisTemplate = (RedisTemplate) ApplicationContextUtils.getBean("redisTemplate");

    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setHashKeySerializer(new StringRedisSerializer());

    Object obj = redisTemplate.opsForHash().get(id.toString(), key.toString());

    return obj;
}
```

~~~markdown
我们将redis中的数据删除，重新运行单元测试
明显看出，缓存命中了
~~~

![image-20210409111052865](https://gitee.com/Akihij/PicGo/raw/master/img/20210409111052.png)

![image-20210409111119362](https://gitee.com/Akihij/PicGo/raw/master/img/20210409111119.png)



#### 缓存带来的问题

~~~markdown
我们都知道缓存会带来一些问题，比如我们修改后，再次查询，我们依旧会查询到redis中的缓存。

针对这个问题:
在redis中的解决方案是从redis中删除掉。
~~~

~~~markdown
# clear()方法和removeObject()方法
删除时使用哪一个方法，我们暂时没有对方法进行具体操作。
~~~

```java
@Override
public Object removeObject(Object key) {
    System.out.println("removeObject ================  key = " + key);
    return null;
}

@Override
public void clear() {
    System.out.println("clear ================  ");
}
```

~~~markdown
# 同时编写一个修改方法
~~~

```
<update id="updata" parameterType="com.jiang.travels.entity.User">
    update t_user set email = #{email}
    where id = #{id}
</update>

@Test
public void Updata(){
    User user = new User();
    user.setId("9");
    user.setEmail("11111@qq.com");
    userService.updata(user);
}
```

![image-20210409113712974](https://gitee.com/Akihij/PicGo/raw/master/img/20210409113713.png)

~~~markdown
我们可以发现最后执行的时clear方法
~~~

~~~java
//编写clear方法
@Override
public void clear() {
   System.out.println("clear ================  ");
   //通过application工具类获取redisTemplate
   RedisTemplate redisTemplate = (RedisTemplate) 	ApplicationContextUtils.getBean("redisTemplate");

   redisTemplate.setKeySerializer(new StringRedisSerializer());
   redisTemplate.setHashKeySerializer(new StringRedisSerializer());

   redisTemplate.delete(id.toString());
}
~~~

~~~markdown
运行之后，我们再去redis中查看是否还有缓存。
~~~

![image-20210409114150467](https://gitee.com/Akihij/PicGo/raw/master/img/20210409114150.png)

~~~markdown
证明redis中的缓存已经被删除了
~~~



## 代码

#### RedisCache.java

```java
package com.jiang.travels.cache;

import com.jiang.travels.utils.ApplicationContextUtils;
import org.apache.ibatis.cache.Cache;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/***
 * 自定义Redis缓存的实现
 */
public class RedisCache implements Cache {

    private final String id;

    public RedisCache(String id) {
        System.out.println("id = " + id);
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }


    //代码冗余
    public RedisTemplate getRedisTemplate(){

        //通过application工具类获取redisTemplate
        RedisTemplate redisTemplate = (RedisTemplate) ApplicationContextUtils.getBean("redisTemplate");

        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());

        return redisTemplate;
    }


    //放入
    @Override
    public void putObject(Object key, Object value) {
        System.out.println("key = " + key);
        System.out.println("value = " + value);

        //将redishash类型作为缓存存储模型  key  hashkey  value
        getRedisTemplate().opsForHash().put(id.toString(),key.toString(),value);

    }

    //读取
    @Override
    public Object getObject(Object key) {
        System.out.println("key = " + key);
        return getRedisTemplate().opsForHash().get(id.toString(), key.toString());

    }

    @Override
    public Object removeObject(Object key) {
        System.out.println("removeObject ================  key = " + key);
        return null;
    }

    @Override
    public void clear() {
        System.out.println("clear ================  ");
        getRedisTemplate().delete(id.toString());

    }

    @Override
    public int getSize() {
        //获取hash中key value的数量
        return getRedisTemplate().opsForHash().size(id.toString()).intValue();
    }
}
```

#### ApplicationContextUtils.java

```java
package com.jiang.travels.utils;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

//用来获取Springboot创建好的工厂
@Component
public class ApplicationContextUtils implements ApplicationContextAware {

    //工厂
    private static ApplicationContext applicationContext;

    //将创建好的工厂以参数形式传递给这个类
    @Override
    public void  setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    //提供在工厂中获取对象的方法
    public static Object getBean(String beanName){
        return applicationContext.getBean(beanName);
    }
}
```



`参考资料:https://www.bilibili.com/video/BV1jD4y1Q7tU?p=17`