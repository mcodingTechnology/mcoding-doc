## mcoding

m·coding 是收集优质代码于一身的技术平台与架构，通过平台服务化的方式，致力于为企业级构建移动化、互联网化、柔性化的业务系统。

mcoding 基于 Spring Boot 、 Spring Cloud 、 Spring Cloud Alibaba 、 Mybatis-Plus 等主流技术栈实现的统一技术平台，其中业务中心层采用 COLA 架构以满足业务为核心、解耦外部要求、分离业务复杂度和技术复杂度等要求。

#### 目录

```
+---mcoding
|   +---demo                                    // 示例工程
|   |   +---demo-adapter                        // 示例工程-适配层
|   |   +---demo-app                            // 示例工程-应用层
|   |   +---demo-client                         // 示例工程-SDK服务对外透出的API
|   |   +---demo-domain                         // 示例工程-领域层
|   |   +---demo-infrastructure                 // 示例工程-基础设施层
|   |   \---start                               // 示例工程-启动类
|   +---mcoding-center                          // 业务中心层
|   |   \---mcoding-payment-center              // 支付中心
|   +---mcoding-core-framework                  // 核心框架
|   |   +---mcoding-common                      // 基础公共类
|   |   +---mcoding-spring-boot-starter-banner  // 自定义banner
|   |   +---mcoding-spring-boot-starter-job     // 任务调度
|   |   +---mcoding-spring-boot-starter-mq      // 消息队列
|   |   +---mcoding-spring-boot-starter-redis   // Redis操作
|   |   \---mcoding-spring-boot-starter-swagger // swagger集成
|   \---mcoding-dependencies                    // 依赖管理
```

#### Maven 的依赖结构

pom.xml(mcoding) 管理了四个 module：demo、mcoding-center、mcoding-core-framework、mcoding-dependencies

pom.xml(mcoding-dependencies) 管理了项目中所用到的所有依赖的版本

pom.xml(mcoding-core-framework) 通过 parent 方式引入了 mcoding-dependencies，内部模块通过 parent 方式依赖 mcoding-core-framework

pom.xml(demo) 通过 parent 方式引入了 mcoding-dependencies