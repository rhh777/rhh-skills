# Spring Boot 讲解包

## 触发探测
- `pom.xml` 或 `build.gradle` 含 `spring-boot`
- 存在 `@SpringBootApplication` 注解的类

## 重点看什么(检查清单)
- Bean 生命周期与注入方式(构造器注入 vs 字段注入)
- 事务边界:`@Transactional` 的传播级别、自调用失效陷阱
- AOP 切面影响了哪些方法(`@Aspect` / 代理)
- 配置来源:`application.yml` / `@ConfigurationProperties` / profile
- 请求链路:Controller → Service → Repository 分层是否清晰
- 异常处理:`@ControllerAdvice` / `@ExceptionHandler` 统一在哪

## 常见存疑点(供"③标记存疑"参考)
- `@Transactional` 加在 private 方法或被自调用 → 不生效
- 字段注入(`@Autowired` 字段)导致循环依赖被悄悄吞掉
- JPA 懒加载触发 N+1 查询
- `@Async` / 事务在同类自调用下失效(都走代理)
- 把业务逻辑写进 Controller,Service 形同虚设

## 扫盲套路(供"④扫盲")
- 依赖注入:用大白话讲"谁负责 new 对象、谁把它塞进来"
- 代理机制:Spring 在外面包了一层代理,所以"自己调自己"绕过了代理,事务/切面就失效
- Bean 作用域:singleton 默认全局一个,别在里面存请求级状态
