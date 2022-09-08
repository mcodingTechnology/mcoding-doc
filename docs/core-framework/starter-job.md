# 任务调度组件

任务调度组件依赖 [xxl-job](https://www.xuxueli.com/xxl-job/)，本地开发时建议在 docker 中部署 xxl-job 服务。

## 使用方法

1、在工程的 **Adapter** 模块添加依赖

```xml
<dependency>
    <groupId>cn.mcoding</groupId>
    <artifactId>mcoding-spring-boot-starter-job</artifactId>
</dependency>
```

2、在工程的 **Start** 模块增加配置内容，配置文件在 `resources/application.yml`

```yaml
# Job 相关配置
xxl:
  job:
    # 是否启用 XXL-JOB
    enabled: true
    # 2.3.1版本调度通讯默认启用accessToken
    accessToken: default_token
    admin:
      addresses: http://127.0.0.1:8080/xxl-job-admin
    # 执行器
    executor:
      appName: ${spring.application.name}
      logPath: ${user.home}/logs/xxl-job/${spring.application.name}
      # 开发模式下，当XXL-JOB部署到本机的Docker容器中时，任务调度执行的时候会从容器内部发起请求
      # 调用宿主机上的服务（本机IDEA起的应用服务），为了让调度能正常访问本机上的服务，需要在这里
      # 手动指定执行器的IP：host.docker.internal，因为在容器内部通过host.docker.internal
      # 能正常访问到宿主机。
      # 详见官方文档：https://docs.docker.com/desktop/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host
      ip: host.docker.internal
```

3、创建 JobHandler 类

!> 注意：JobHandler 的处理类需要放到 **Adapter** 模块中，在 controller 中创建 job 目录

```java
package cn.mcoding.demo.controller.job;

import com.xxl.job.core.context.XxlJobHelper;
import com.xxl.job.core.handler.annotation.XxlJob;
import org.springframework.stereotype.Component;

@Component
public class DemoJob {
    @XxlJob("demoJobHandler")
    public void demoJobHandler() {
        XxlJobHelper.log("XXL-JOB, Hello World.");
        // 业务处理
    }
}
```

## 配置说明

- `xxl.job.enabled` 配置项控制是否启用 xxl-job 配置，**默认值：false**
- `xxl.job.accessToken` 配置项在 xxl-job 版本 **2.3.1** 开始默认启用，这里可以默认填：`default_token`，也可以自定义 token
- `xxl.job.executor.ip` 配置项一般情况下不用配置，但如果在本地测试时，xxl-job 调度服务是跑在 docker 中的，就需要指定 IP：`host.docker.internal`，具体原因前面有说明