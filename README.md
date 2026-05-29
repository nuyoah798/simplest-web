# rest-api-spring-boot-starter

一个面向 Spring Boot Web API 的快速开发脚手架，目标是把统一返回、分页包装、异常处理、日志、缓存、OpenAPI 文档、代码生成等重复工作收敛起来，让业务代码更干净。

## 特性

- 统一 REST API 返回
- 分页返回包装与字段自定义
- 局部关闭统一返回
- 全局异常处理与参数校验
- 错误国际化
- 第三方请求封装（RestTemplate）
- Redis 工具与 Spring Cache 封装
- OpenAPI 3 / Swagger 文档
- MyBatis Plus 代码生成
- 日志链路追踪、IP 城市记录
- HttpUserAgent、RequestUtil 等工具封装

## 快速开始

### 1. 引入依赖

```xml
<dependency>
  <groupId>cn.soboys</groupId>
  <artifactId>rest-api-spring-boot-starter</artifactId>
  <version>1.5.0</version>
</dependency>
```

### 2. 开启功能

在 Spring Boot 启动类或配置类上添加 `@EnableRestFullApi`。

```java
@SpringBootApplication
@EnableRestFullApi
public class SuperaideApplication {

    public static void main(String[] args) {
        SpringApplication.run(SuperaideApplication.class, args);
    }
}
```

启用后，项目中的接口会自动走统一返回格式。

## 统一返回

普通 `Controller` 方法返回 `Map`、实体对象或集合时，会自动包装成统一响应。

```java
@PostMapping("/chat")
public HashMap chatDialogue() {
    HashMap m = new HashMap();
    m.put("age", 26);
    m.put("name", "Judy");
    return m;
}
```

统一响应示例：

```json
{
  "success": true,
  "code": "OK",
  "msg": "操作成功",
  "requestId": "IPbHLE5SZ1fqI0lgNXlB",
  "timestamp": "2023-07-09 02:39:40",
  "data": {
    "name": "judy",
    "hobby": "swing",
    "age": 18
  }
}
```

也可以直接返回 `Result`：

```java
@PostMapping("/chat")
public Result chatDialogue(@Validated EntityParam s) {
    return Result.buildSuccess(s);
}
```

## 分页支持

分页场景可以使用 `ResultPage`：

```java
@PostMapping("/page")
@Log("分页查询用户数据")
public Result page(@Validated EntityParam s) {
    ResultPage<List<EntityParam>> resultPage = new ResultPage<>();
    List a = new ArrayList();
    a.add(s);
    resultPage.setPageData(a);
    return ResultPage.buildSuccess(resultPage);
}
```

分页返回默认会携带 `previousPage`、`nextPage`、`pageSize`、`totalPageSize`、`hasNext` 等信息。
如果项目希望把分页对象继续放到 `data` 中，可以开启 `pageWrap`，并把分页列表字段映射成自己的 key，例如 `records`。

### 自定义分页字段

```yaml
rest-api:
  enabled: true
  msg: msg1
  code: code1
  code-success-value: 200
  success: success1
  previousPage: previousPage1
  nextPage: nextPage1
  pageSize: pageSize1
  hasNext: hasNext1
  totalPageSize: totalPageSize1
  data: info
```

## 局部关闭统一返回

某些接口不想被包装时，可以使用 `@NoRestFulApi`：

```java
@GetMapping("/test")
@NoRestFulApi
public Map chatDialogue() {
    Map m = new HashMap<>();
    m.put("name", "judy");
    m.put("age", 26);
    return m;
}
```

也可以通过包扫描配置统一控制：

```yaml
include-packages: cn.soboys.superaide.controller
exclude-packages: xx.xxx.xxx
```

默认会过滤 `String` 类型，认为它是页面路径。

## 错误处理

项目封装了常见 HTTP 错误和参数校验错误，并支持错误国际化。

```json
{
  "success": false,
  "code": "405",
  "msg": "方法不被允许",
  "timestamp": "2023-07-03 22:36:47",
  "data": "Request method 'GET' not supported"
}
```

```json
{
  "success": false,
  "code": "404",
  "msg": "请求资源不存在",
  "timestamp": "2023-07-03 22:42:35",
  "data": "/api"
}
```

如果需要让 Spring Boot 把错误抛给全局异常处理器，可以加上：

```yaml
spring:
  mvc:
    throw-exception-if-no-handler-found: true
  web:
    resources:
      add-mappings: false
```

### 错误国际化

支持按 `i18n-header` 选择语言，默认中文，也可以扩展更多语言。

## 第三方请求

项目封装了 `RestTemplate`，用于常见的第三方 HTTP 请求：

- GET
- POST
- DELETE
- PUT

## 日志与链路追踪

统一响应中包含 `requestId`，便于日志链路查询。
`Log` 注解可以标记需要记录的接口，日志支持异步写入，并可自定义日志数据源。

## IP 城市记录

日志可记录 IP 对应的城市信息。默认通过 `ip2region` 获取，也支持通过工具类直接查询。

## OpenAPI 文档

- Swagger UI：`/swagger-ui.html`
- springdoc 增强 UI：`/doc.html`
- OpenAPI JSON：`/v3/api-docs`
- OpenAPI YAML：`/v3/api-docs.yaml`

如果不需要 Swagger UI，可以关闭：

```yaml
springdoc:
  swagger-ui:
    enabled: false
```

## 代码生成

项目封装了 MyBatis Plus 代码生成工具，适合单表 CRUD 快速生成。

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-generator</artifactId>
  <version>3.4.1</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.28</version>
</dependency>
<dependency>
  <groupId>org.freemarker</groupId>
  <artifactId>freemarker</artifactId>
  <version>2.3.31</version>
</dependency>
```

### 代码生成配置

```java
@Data
public class GenerateCodeConfig {
    private String driverName;
    private String username;
    private String password;
    private String url;
    private String projectPath;
    private String packages;
}
```

## Redis 与缓存

项目提供 Redis key/value 封装，同时封装了 Spring Cache。默认不引入 Redis 依赖时使用内存缓存；引入 Redis 依赖后会自动切换为 Redis 实现。

```yaml
redis:
  key-prefix: rest
```

## 其他工具

- `Assert`：业务断言
- `JwtUtil`：JWT 工具
- `ValidatorUtil`：校验工具
- `RequestUtil`：请求参数解析
- `HttpUserAgent`：请求设备识别
- `Limit`：限流相关能力
