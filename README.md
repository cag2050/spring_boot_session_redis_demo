* Redis 在 Spring Boot 中两个典型的应用场景：数据缓存、共享Session。此项目是 共享Session 的例子。

### 步骤一：创建项目
一、构建工具选择：maven 

### 步骤二：创建 Redis 容器
1. 创建容器（暴露端口：6379，使用 Redis 可视化界面工具（如：Fastoredis）连接 redis 时连接该端口）：
`
docker run -it -p 6379:6379 redis:5.0.3
`
2. 进入容器：
`
docker exec -it 创建的容器名或容器id bash
`
3. 进入redis命令行：
`
redis-cli
`

### 步骤三：配置 Redis session
1.pom.xml 添加以下依赖
```
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```
2.新建 com/example/spring_boot_session_redis_demo/config/RedisSessionConfig.java
```
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30)
public class RedisSessionConfig {
}
```
3.redis-cli 命令中查看保存的 session；spring:session:sessions:xxx 是 hash 结构；
```
127.0.0.1:6379> keys *
1) "spring:session:expirations:1568895960000"
2) "spring:session:sessions:expires:a8b0bdd8-c47a-4cf1-815b-315b88d3e655"
3) "spring:session:sessions:a8b0bdd8-c47a-4cf1-815b-315b88d3e655"
127.0.0.1:6379> type spring:session:sessions:a8b0bdd8-c47a-4cf1-815b-315b88d3e655
hash
127.0.0.1:6379> type spring:session:sessions:expires:a8b0bdd8-c47a-4cf1-815b-315b88d3e655
string
127.0.0.1:6379> type spring:session:expirations:1568895960000
set
127.0.0.1:6379> hkeys spring:session:sessions:a8b0bdd8-c47a-4cf1-815b-315b88d3e655
1) "maxInactiveInterval"
2) "creationTime"
3) "lastAccessedTime"
4) "sessionAttr:uid"
127.0.0.1:6379> HGETALL spring:session:sessions:a8b0bdd8-c47a-4cf1-815b-315b88d3e655
1) "maxInactiveInterval"
2) "\xac\xed\x00\x05sr\x00\x11java.lang.Integer\x12\xe2\xa0\xa4\xf7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00'\x8d\x00"
3) "creationTime"
4) "\xac\xed\x00\x05sr\x00\x0ejava.lang.Long;\x8b\xe4\x90\xcc\x8f#\xdf\x02\x00\x01J\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x01l\xae\xfc>\xfb"
5) "lastAccessedTime"
6) "\xac\xed\x00\x05sr\x00\x0ejava.lang.Long;\x8b\xe4\x90\xcc\x8f#\xdf\x02\x00\x01J\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x01l\xae\xfc\xd3\xf8"
7) "sessionAttr:uid"
8) "\xac\xed\x00\x05sr\x00\x0ejava.util.UUID\xbc\x99\x03\xf7\x98m\x85/\x02\x00\x02J\x00\x0cleastSigBitsJ\x00\x0bmostSigBitsxp\xbbz\x92\xc5\x0b\xf7\xabG\xd9\xf7\xbeQW\xd9G\xef"
127.0.0.1:6379> ttl spring:session:sessions:expires:a8b0bdd8-c47a-4cf1-815b-315b88d3e655
(integer) 2536073
127.0.0.1:6379> smembers spring:session:expirations:1568895960000
1) "\xac\xed\x00\x05t\x00,expires:a8b0bdd8-c47a-4cf1-815b-315b88d3e655"
```
4.访问：http://localhost:8080/uid ，返回 session。

### 参考
参考文章 | 网址
--- | ---
SpringBoot应用之分布式会话 | https://segmentfault.com/a/1190000004358410
Spring Boot(三)：Spring Boot 中 Redis 的使用 | http://www.ityouknow.com/springboot/2016/03/06/spring-boot-redis.html