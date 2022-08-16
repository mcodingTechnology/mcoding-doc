## Redis 工具类

该工具类封装了常用的 Redis 操作方法

#### 使用方法

在应用架构中对应的模块添加依赖

```xml

```

在 application 配置文件增加 Redis 相关配置（Redisson 相关配置待测试）

```
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.database=0
```

注入 RedisService Bean，调用对应的函数

```java
@Resource
private RedisService redisService;

public Response execute() {
  // Object
  boolean b = redisService.setObject("object-key-1", "abc");
  System.out.println("Object 数据存入缓存：" + b);
  String str = redisService.getObject("object-key-1");
  System.out.println("Object 取：" + str);

  // List
  UserProfileGetQry qry1 = new UserProfileGetQry();
  qry1.setId("1");
  qry1.setUserId("12");
  qry1.setOperater("管理员");
  UserProfileGetQry qry2 = new UserProfileGetQry();
  qry2.setId("1");
  qry2.setUserId("12");
  qry2.setOperater("管理员");
  UserProfileGetQry qry3 = new UserProfileGetQry();
  qry3.setId("1");
  qry3.setUserId("12");
  qry3.setOperater("管理员");
  List<UserProfileGetQry> list = new ArrayList<>();
  list.add(qry1);
  list.add(qry2);
  list.add(qry3);
  b = redisService.setList("list-key-1", list);
  System.out.println("List 数据存入缓存：" + b);
  List<UserProfileGetQry> qryList = redisService.getList("list-key-1");
  System.out.println("List 取："+qryList);

  // HashMap
  HashMap<String, String> map = new HashMap<>();
  map.put("1", "a1");
  map.put("2", "a2");
  map.put("3", "a3");
  b = redisService.setHash("hash-key-1", map);
  System.out.println("HashMap 数据存入缓存：" + b);
  Map<String, Object> hash = redisService.getHash("hash-key-1");
  System.out.println("HashMap 取："+hash);

  // Object方法存List
  UserProfileGetQry qry4 = new UserProfileGetQry();
  qry4.setId("1");
  qry4.setUserId("12");
  qry4.setOperater("管理员");
  list.add(qry4);
  b = redisService.setObject("object-list-key-1", list);
  System.out.println("Object 方法中存List："+b);
  List<UserProfileGetQry> l = redisService.getObject("object-list-key-1");
  System.out.println("Object 方法中存List取："+l);
  System.out.println(l.get(2));

  // 递增、递减
  long incr = redisService.incr("incr-1");
  System.out.println("递增："+incr);
  long decr = redisService.decr("decr-1");
  System.out.println("递减："+decr);

  return userProfileGateway.hello();
}
```

## 基于 Redis 实现的 Cache 配置类

mcoding-spring-boot-starter-redis 除了封装常用的Redis操作函数外，还启用了 Spring Boot Cache （基于Redis）

#### 使用方式

在应用架构中对应的模块添加依赖

```xml

```

在 application 配置文件增加缓存的配置项，缓存过期时间可以根据项目实际情况做调整

```
# Cache 配置项
spring.cache.type=redis
spring.cache.redis.cache-null-values=false
# 设置过期时间为 1 小时
spring.cache.redis.time-to-live=1h
```

设置缓存，在方法上增加 `@Cacheable` 注解

```java
@Cacheable(cacheNames = "UserProfile", key = "#qry.userId")
public SingleResponse<UserProfileCO> execute(UserProfileGetQry qry) {

    QueryWrapper<UserProfileDO> queryWrapper = new QueryWrapper<>();
    queryWrapper.lambda().eq(UserProfileDO::getUserId, qry.getUserId());
    UserProfileDO userProfileDO = userProfileMapper.selectOne(queryWrapper);

    UserProfileCO userProfileCO = userProfileConvertor.do2Co(userProfileDO);

    return SingleResponse.of(userProfileCO);
}
```

!> 设置缓存之后，如果数据有更新或删除动作，请注意及时更新缓存

移除缓存，在方法上增加 `@CacheEvict` 注解

```java
@CacheEvict(cacheNames = "UserProfile", key = "#cmd.userProfileCO.userId")
public Response execute(UserProfileUpdateCmd cmd) {
    // update UserProfile info
    return Response.buildSuccess();
}
```