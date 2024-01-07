# Marketing API Scheduler

## 设计
![Marketing API Scheduler](https://github.com/denghj2023/marketing-api-scheduler/assets/129133885/6dd9c63f-d9a7-4893-abb0-9f2b61801b09)

* ApiEndpoint：定义了ApiEndpoint的标准行为，包括创建请求、限流、断言、解析、结果处理、请求失败、请求已取消。
* ApiScheduling：继承ApiEndpoint，定义了调度的标准行为，包括开始、结束、配置线程池（资源隔离）。
* ApiSchedulingContainer：负责管理ApiScheduling的生命周期，使用Http Client5来进行HTTP调用，支持请求重试、限流、统计。

## 调度开发

_示例_
```JAVA
@Component
@ApiScheduled(delay = @Delay(second = 300, variables = DateRange.LAST_TWO_DAY))
public class OceanengineReportAdGet implements ApiScheduling {

    @Override
    public List start() {
        // 返回活跃的广告账号列表
        return null;
    }

    @Override
    public SimpleHttpRequest create(Object access, Object variables) {
        // 创建请求
        return null;
    }
    
    @Override
    public boolean assertion(Object access, SimpleHttpResponse response) {
        // 断言请求
        return response.getCode() == 200;
    }

    @Override
    public Object parse(Object access, SimpleResponse simpleResponse) {
        // 解析请求
        return JSON.parseObject(simpleResponse.getBody(), Object.class);
    }

    @Override
    public int accept(Object access, Object resp, Object variables) {
        // 保存数据
        return 0;
    }
    
    // ...
}
```

## API

* 列举所有调度：`GET /scheduling/list`
* 执行调度：`GET /scheduling/start?scheduling=OceanengineReportAdGet&start=2024-01-01&end=2023-01-05`

## 限流

_示例_
```JAVA
@Component
public class OceanengineRateLimiter implements IRateLimiter {

    private final RateLimiter rateLimiter = RateLimiter.of("Oceanengine", RateLimiterConfig.custom()
            .limitForPeriod(1)
            .limitRefreshPeriod(Duration.ofMillis(150))
            .timeoutDuration(Duration.ofMinutes(1L))
            .build());

    @Override
    public boolean support(Class<?> apiEndpointClass) {
        // 判断是否需要限流
    }

    @Override
    public boolean tryAcquire(ApiEndpoint<?, ?> apiEndpoint, Object access) {
        boolean acquirePermission = rateLimiter.acquirePermission();
        log.debug("acquirePermission = {}", acquirePermission);
        return acquirePermission;
    }
}
```

## Monitoring(Metrics)
(待补充)


