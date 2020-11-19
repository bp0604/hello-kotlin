# 翻译使用：https://translate.google.cn/

## 스프링 부트 애플리케이션 실행 运行Spring Boot应用程序
애플리케이션 패키징 하고 나면, 대상 폴더에 <code>spring-kotlin-0.0.1-SNAPSHOT.jar</code> 파일이 생성된다. 스프링 부트에서는
내장 톰캣을 갖고 있기 때문에 <code>java -jar</code> 명령어로 쉽게 실행할 수 있다.

打包应用程序后，将在目标文件夹中创建<code> spring-kotlin-0.0.1-SNAPSHOT.jar </ code>文件。
由于它具有内置的Tomcat，因此可以使用<code> java -jar </ code>命令轻松运行它。

```shell script
mvnw package
java -jar target/spring-kotlin-0.0.1-SNAPSHOT.jar
``` 

### 실행 가능 JAR 만들기 创建可执行JAR
아래와 같이 <code>executable</code> 옵션을 활성화하면 <code>java -jar</code> 명령어 없이 jar 파일을 실행 가능한 파일로 생성할 수 있다.

如果激活如下所示的<code> executable </ code>选项，则可以在不使用<code> java -jar </ code>命令的情况下将jar文件创建为可执行文件。
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

```shell script
./target/spring-kotlin-0.0.1-SNAPSHOT.jar
```

## 애플리케이션 설정 값 설정 방법 如何设置应用程序设置
1. application.properties
2. application.yml
3. 명령줄 인수命令行参数

## 프로파일 个人资料
프로파일 값에 따라 설정 값을 변경할 수 있다. 아래 프로파일 기본값은 <code>development</code>로 설정되어 있으며,
다른 프로파일로 실행하려면 <code>--spring.profiles.active="production"</code> 인수를 추가하면 된다. 

您可以根据配置文件值更改设置值。下面的默认配置文件设置为<code> development </ code>，
如果要使用其他配置文件运行，只需添加<code>-spring.profiles.active =“ production” </ code>参数。

```yaml
spring:
  profiles:
    active: "development"

---

spring:
  profiles: "development"

---

spring:
  profiles: "production"
```

## 경로 변수 路径变量
<code>@Pathvariable</code> 어노테이션을 사용해서 URL로 넘어오는 값을 얻을 수 있다.
您可以使用<code> @Pathvariable </ code>批注获取传递到URL的值。

```kotlin
@GetMapping("/customer/{id}")
fun getCustomer(@PathVariable id: Int) = customers[id]
```
 
## 요청 매개 변수 请求参数
<code>@RequestParam</code> 어노테이션을 사용해서 매개 변수 값을 얻을 수 있다.
可以使用<code> @RequestParam </ code>批注获取参数值。

```kotlin
@GetMapping("/customers")
fun getCustomers(@RequestParam(required = false, defaultValue = "") nameFilter: String)
        = customers.filter {
            it.value.name.contains(nameFilter, true)
        }.map(Map.Entry<Int, Customer>::value).toList();
```

## Controller 에러 처리
控制器错误处理

```kotlin
@ControllerAdvice
class ErrorHandler {

    @ExceptionHandler(JsonParseException::class)
    fun jsonParseExceptionHandler(servletRequest: HttpServletRequest, exception: Exception): ResponseEntity<ErrorResponse> {
        return ResponseEntity(ErrorResponse("JSON Error", exception.message ?: "invalid json"), BAD_REQUEST)
    }
}
```

## Reactor Publisher 구현체
Reactor Publisher实施

- Mono : 0-1개의 데이터를 전달 Mono：传输0-1数据
```kotlin
val customer = Customer(1, "Mono").toMono()
```

- Flux : 0-N개의 데이터를 전달 Flux：传输0-N数据
```kotlin
val customerFlux = listOf(Customer(1, "Customer1"), Customer(2, "Customer2")).toFlux()
```

## RouterFunction
<code>Controller</code> 대신 <code>RouterFunction</code>를 사용해서 서버로 들어오는 요청을 처리할 수 있다.
可以使用<code> RouterFunction </ code>代替<code> Controller </ code>来处理进入服务器的请求。

<code>/functional/customer</code> 경로에 GET 요청을 처리하는 엔드포인트 하나가 선언되어 있다. 응답에 대한 코드는 핸들러
클래스로 따로 빼내서 정의한다.
在路径<code> / functional / customer </ code>中，有一个端点处理GET请求。响应的代码是一个处理程序
它被单独定义为一个类。

### CustomerRouter

```kotlin
@Component
class CustomerRouter(private val customerHandler: CustomerHandler) {

    @Bean
    fun customerRoutes() = router {
        "/functional".nest {
            "/customer".nest {
                GET("/", customerHandler::get)
            }
        }
    }
}
``` 

### CustomerHandler

```kotlin
@Component
class CustomerHandler {

    fun get(serverRequest: ServerRequest) =
        ok().body(Customer(1, "Hello World").toMono(), Customer::class.java)

}
```

## 데이터베이스 설치 数据库安装
논블로킹 마이크로서비스가 블로킹 오퍼레이션을 사용해서 데이터를 쿼리를 하게 되면, 리액티브 프로그래밍의 이점을 잃어버린다. 스프링 데이터를 이용해서 
리액티브 작업을 진행하기 전에 먼저 데이터베이스가 설치되어 있어야 한다. 이번 예제 프로젝트에서는 NoSQL 데이터베이스인 몽고 DB를 활용한다. 

当非阻塞微服务使用阻塞操作查询数据时，它们将失去反应式编程的优势。带有弹簧数据
必须先安装数据库，然后才能进行响应式。在此示例项目中，使用了Nogo数据库Mongo DB。

### Docker Mongo DB 설치 安装
```shell script
docker run --name mongo -p 27017:27017 -d mongo
```

### Docker Container 확인 检查Docker容器
```shell script
docker ps
```

### Docker Container 접속 Docker容器连接
```shell script
docker exec -it mongo bash
```

### Mongo DB 버전 확인 Mongo DB版本检查
```shell script
mongo
db.version()
```
目前显示4.4.1


![mongo-version](https://user-images.githubusercontent.com/43853352/73738727-5a0ac100-4788-11ea-875a-a3a15f60dba8.png)

### Mongo Auth 접속 Mongo Auth连接
```shell script
mongo admin -u 'username' -p 'password'
```
此时报错，需要先建帐号。

```
docker exec -it mongo bash
mongo
use admin;
db.createUser({ user: "username", pwd: "password", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })


认证进入：mongo admin -u username -p password
```


### MongoDB Tool
[Compass 설치](https://www.mongodb.com/products/compass)

默认无密码时：
mongodb://192.168.123.110:27017

有密码时：
mongodb://username:password@192.168.123.110:27017


![mongodb-compass](https://user-images.githubusercontent.com/43853352/73739702-16b15200-478a-11ea-9b58-a724181ed212.png)]

# MongoDB 添加用户 
https://www.cnblogs.com/jinjiangongzuoshi/p/9305748.html

https://docs.mongodb.com/manual/tutorial/enable-authentication/



# docker mongodb新建用户并授权
https://my.oschina.net/matt0614/blog/3067063

### 启动mongodb 容器

```
[root@vultr ~]# docker run -d --name mymongo -p 27017:27017 --privileged=true docker.io/mongo --auth
```

### 创建用户并授权

```
[root@vultr ~]# docker exec -it mymongo mongo admin
> use admin
> db.createUser( {user: "admin",pwd: "admin",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]})
> db.auth("admin","admin")
> use nxxq
> db.createUser( {user: "test",pwd: "test",roles: [ { role: "readWrite", db: "testdb" } ]})
```

### springboot 连接testdb数据库地址

#### 方法1

```
spring.data.mongodb.database=testdb
spring.data.mongodb.host=192.168.10.111
spring.data.mongodb.port=27017
spring.data.mongodb.username=test
spring.data.mongodb.password=test
```

#### 方法2

```
spring.data.mongodb.uri=mongodb://test:test@192.168.10.111:27017/testdb
```
