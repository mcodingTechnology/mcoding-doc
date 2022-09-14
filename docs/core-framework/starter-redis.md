# Redis 组件

自定义 Redis 组件除了封装 RedisTemplate 提供常用操作方法外，还启用了 Spring Boot Cache （基于Redis） 

## 使用方法

1、在工程中对应的模块添加依赖

```xml
<!-- mcoding 自定义Redis组件 starter -->
<dependency>
    <groupId>cn.mcoding</groupId>
    <artifactId>mcoding-spring-boot-starter-redis</artifactId>
</dependency>
```

2、在工程的 **Start** 模块增加配置内容， 配置文件在 `resources/application.yml`

```yaml
spring:
  # Redis 相关配置
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
  # Spring Boot Cache 配置项 - 基于 Redis
  cache:
    type: REDIS
    redis:
      cache-null-values: false
      # 设置过期时间为 1 小时
      time-to-live: 1h
```

!> 这里的配置项为两个部分，一个是 Redis 的配置，一个是启用 Spring Boot Cache 的配置

## Redis 工具类的使用

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

移除缓存，在方法上增加 `@CacheEvict` 注解

```java
@CacheEvict(cacheNames = "UserProfile", key = "#cmd.userProfileCO.userId")
public Response execute(UserProfileUpdateCmd cmd) {
    // update UserProfile info
    return Response.buildSuccess();
}
```

!> 设置缓存之后，如果数据有更新或删除动作，要注意在 update or delete 方法上增加移除缓存的注解，并且要保证缓存的 key 一致