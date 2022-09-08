# 基础组件
主要是常用的工具类使用说明，其中也依赖了 [hutool](https://hutool.cn/docs/#/)，除了 hutool 之外还有实体转换工具、全局异常处理等等，后面也会慢慢补充。

## 数据模型转换

现在微服务架构和 DDD 领域设计模型思想的指导下，我们的应用和应用之间，模块和模块之前会有很多的数据模型，同一个工程下也会分数据传输层、领域对象层、持久化层等，当这些对象需要相互转换时，如果还手写 Get、Set 方法，开发人员可能会崩溃。

这里我们引入 [MapStruct](https://mapstruct.org/) 框架来处理这些重复的工作量，MapStruct 是一个代码生成器，它基于约定高于配置的方法，大大简化了 Java Bean 类型之间的映射的实现。

在核心框架中提供了三个转换器接口，在项目中根据实际的需求来实现对应的接口。

1、IConvertor 提供数据传输对象（C）、领域对象（E）、数据库对象（D）之间互相转换的能力，包括支持对象集合的转换

<details>
<summary>点我查看代码</summary>

```java
/**
 * 数据传输对象转领域对象
 * @param c Client Object
 * @return Domain Entity
 */
E co2Entity(C c);

/**
 * 数据传输对象转数据库对象
 * @param c Client Object
 * @return Data Object
 */
D co2Do(C c);

/**
 * 领域对象转数据传输对象
 * @param e Domain Entity
 * @return Client Object
 */
C entity2Co(E e);

/**
 * 领域对象转数据库对象
 * @param e Domain Entity
 * @return Data Object
 */
D entity2Do(E e);

/**
 * 数据库对象转数据传输对象
 * @param d Data Object
 * @return Client Object
 */
C do2Co(D d);

/**
 * 数据库对象转领域对象
 * @param d Data Object
 * @return Domain Entity
 */
E do2Entity(D d);

/**
 * 数据传输对象集合转领域对象集合
 * @param cList List<Client Object>
 * @return List<Domain Entity>
 */
List<E> coList2EntityList(List<C> cList);

/**
 * 数据传输对象集合转数据库对象集合
 * @param cList List<Client Object>
 * @return List<Data Object>
 */
List<D> coList2DoList(List<C> cList);

/**
 * 领域对象集合转数据传输对象集合
 * @param eList List<Domain Entity>
 * @return List<Client Object>
 */
List<C> entityList2CoList(List<E> eList);

/**
 * 领域对象集合转数据库对象集合
 * @param eList List<Domain Entity>
 * @return List<Data Object>
 */
List<D> entityList2DoList(List<E> eList);

/**
 * 数据库对象集合转数据传输对象集合
 * @param dList List<Data Object>
 * @return List<Client Object>
 */
List<C> doList2CoList(List<D> dList);

/**
 * 数据库对象集合转领域对象集合
 * @param dList List<Data Object>
 * @return List<Domain Entity>
 */
List<E> doList2EntityList(List<D> dList);
```

</details>

2、IDtoConvertor 提供数据传输对象（C）、领域对象（E）的转换，包括支持对象集合的转换

<details>
<summary>点我查看代码</summary>

```java
/**
 * 数据传输对象转领域对象
 * @param c Client Object
 * @return Domain Entity
 */
E co2Entity(C c);

/**
 * 领域对象转数据传输对象
 * @param e Domain Entity
 * @return Client Object
 */
C entity2Co(E e);

/**
 * 数据传输对象集合转领域对象集合
 * @param cList List<Client Object>
 * @return List<Domain Entity>
 */
List<E> coList2EntityList(List<C> cList);

/**
 * 领域对象集合转数据传输对象集合
 * @param eList List<Domain Entity>
 * @return List<Client Object>
 */
List<C> entityList2CoList(List<E> eList);
```

</details>

3、IEntityConvertor 提供领域对象（E）、持久化对象（D）的转换，包括支持对象集合的转换

<details>
<summary>点我查看代码</summary>

```java
/**
 * 领域对象转数据库对象
 * @param e Domain Entity
 * @return Data Object
 */
D entity2Do(E e);

/**
 * 数据库对象转领域对象
 * @param d Data Object
 * @return Domain Entity
 */
E do2Entity(D d);

/**
 * 领域对象集合转数据库对象集合
 * @param eList List<Domain Entity>
 * @return List<Data Object>
 */
List<D> entityList2DoList(List<E> eList);

/**
 * 数据库对象集合转领域对象集合
 * @param dList List<Data Object>
 * @return List<Domain Entity>
 */
List<E> doList2EntityList(List<D> dList);
```

</details>

#### 使用方法

新建抽象类并实现对应的接口，类上增加 mapstruct 的注解 `@Mapper(componentModel = "spring")`

```java
@Mapper(componentModel = "spring")
public abstract class AbstractUserProfileDoConvertor implements IEntityConvertor<UserProfile, UserProfileDO> {
}
```

在 service 或其他 spring 组件中注入抽象类

```java
@Resource
private AbstractUserProfileDoConvertor userProfileDoConvertor;

// 调用对应的方法
userProfileDoConvertor.entity2Do(userProfile);
```

## 全局异常处理

全局异常处理 `GlobalExceptionHandler` 将异常信息封装成标准格式返回给前端，其中 `appName` 属性为应用服务的名称

#### GlobalExceptionHandler

处理的异常信息如下：

| 异常类 | 备注 |
| ----- | --- |
| MissingServletRequestParameterException | SpringMVC 请求参数缺失 |
| MethodArgumentTypeMismatchException | SpringMVC 请求参数类型错误 |
| MethodArgumentNotValidException | SpringMVC 方法参数无效异常 |
| BindException | SpringMVC 参数绑定不正确，本质上也是通过 Validator 校验 |
| ValidationException | 参数校验异常 |
| IllegalArgumentException | 非法参数异常 |
| HttpMessageNotReadableException | Http消息不可读异常（请求消息序列化异常） |
| ConstraintViolationException | 处理请求单个参数不满足校验规则的异常信息 |
| NoHandlerFoundException | 没有监测到资源异常 |
| HttpRequestMethodNotSupportedException | SpringMVC 请求方法不正确 |
| AccessDeniedException | Spring Security 权限不足的异常 |
| BizException | 框架层面的 BizException |
| SysException | 框架层面的 SysException |
| Exception | 未知异常 |

#### GlobalErrorCode

全局错误码类，基于 HTTP 响应状态定义，200 表示成功，400-499 表示客户端错误，500-599 表示服务端错误

| Enum | Code (String) | Desc (String) |
| ---- | ---- | ---- |
| SUCCESS | 200 | 成功 |
| BAD_REQUEST | 400 | 请求参数不正确 |
| UNAUTHORIZED | 401 | 账号未登录 |
| FORBIDDEN | 403 | 没有该操作权限 |
| NOT_FOUND | 404 | 请求未找到 |
| METHOD_NOT_ALLOWED | 405 | 请求方法不正确 |
| LOCKED | 423 | 请求失败，请稍后重试 |
| TOO_MANY_REQUESTS | 429 | 请求过于频繁，请稍后重试 |
| INTERNAL_SERVER_ERROR | 500 | 系统异常 |

#### ValidMsg

基于 `javax.validation` 框架的验证消息实体，主要是为了将后端的验证消息转换成 JSON String 返回给前端，方便前端处理。主要是在 `MethodArgumentNotValidException` 和 `BindException` 异常处理中使用到。

示例格式：

```json
{
  "success": false,
  "errCode": "400",
  "errMessage": "[{\"field\":\"dep\",\"msg\":\"部门参数不能为空\",\"object\":\"userProfilePageQry\"},{\"field\":\"userId\",\"msg\":\"用户ID不能为空\",\"object\":\"userProfilePageQry\"}]"
}
```

#### 使用方法

在应用架构的 **Adapter** 层创建一个异常处理类 `DemoExceptionHandler` （命名自定义），同时需要继承 `GlobalExceptionHandler` 类，创建构造函数传递应用名称，记得加上注解：`@RestControllerAdvice`

```java
@RestControllerAdvice
public class DemoExceptionHandler extends GlobalExceptionHandler {

    public DemoExceptionHandler() {
        super("Demo");
    }
}
```

?> 传递 appName 是为了在异常日志中输出当前应用系统的名称，方便后续的定位和分析

## 工具类

#### DruidUtil

Druid 数据源数据库密码加密工具类