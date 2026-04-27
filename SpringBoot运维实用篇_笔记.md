# SpringBoot运维实用篇

### YW-1.SpringBoot程序的打包和运行

我们以后制作的程序都是运行在专用的服务器上的,那么如何将我们的程序运行在这台专用的电脑上呢?

--将程序先组织成一个文件,然后将这个文件传输到这台服务器上,也就是两个过程:1.打包 2.运行

#### 1.程序打包

Springboot基于Maven创建,在Maven种提供有打包的指令,打包后产生一个与工程名类似的jar文件,其名称是由**模块名+版本号+.jar**组成

```java
mvn package
```

运行完之后,后台日志显示BUILD SUCCESS即打包成功,打包好的jar包会放在target目录里面,可以通过在同文件路径后台cmd执行命令:

#### 2.程序运行

```
java -jar 工程报名.jar//(可以按首字母再直接按tab键,电脑会自动补全)
```

想要清楚上次打包的jar包重新建一个,并且可以让后台日志不要显示那么多冗余信息,可以输入以下指令:

```java
mvn clean package -q
```

**总结**:

1. SpringBoot工程可以基于Java环境下独立运行jar文件启动服务
2. SpringBoot工程执行mvn命令package进行打包
3. 执行jar命令:java -jar 工程名.jar

### SpringBoot程序打包失败处理

​		有些小伙伴打包以后执行会出现一些问题，导致程序无法正常执行，例如下面的现象

![image-20260418145912896](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260418145912896.png)

在SpringBoot工程的pom.xml中有下面这组配置，这组配置决定了打包出来的程序包是否可以执行

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

1. SpringBoot程序添加配置后会打出一个特殊的包,包含SPring框架部分功能,原始工程内容,原始工程依赖的jar包
2. 首先读取MANIFEST.MF文件中的Main-Class属性,用来标记java -jar命令后运行的类
3. JarLauncher类执行时回找到StartClass属性,也就是启动类类名
4. 运行启动类时会启动当前工程的内容
5. 运行当前工程时会使用依赖的jar包,从lib目录中查找

**总结**:spring-boot-maven-plugin插件用于将当前程序打包成一个可以独立运行的程序包

------

### 命令行启动常见问题及解决方案

```java
//查询端口
netstat -ano
//查询指定端口
netstat -ano |findstr "端口号"
//根据进程PID查询进程名称
tasklist |findstr "进程PID号"
//根据PID杀死任务
taskkill /F /PID "进程PID号"
根据进程名称杀死任务
taskkill -f -t -im "进程名称"
```

------

### YW-3.多环境开发

电脑上写的程序最终要放到别人的服务器上去运行。每个计算机环境不一样，这就是多环境。

- 开发环境-自己用
- 测试环境-自己公司用
- 生产环境-甲方用

![image-20260418154908966](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260418154908966.png)

#### YW-3-2.多环境开发（yaml多文件版）

在 SpringBoot 里，如果有配置项（比如日志级别、Tomcat 端口）在开发、测试、线上环境都用同一个值，就把它们写进**主配置文件**；只有不同环境需要不一样的配置（比如数据库地址），才写进对应的**环境配置文件**。

- **主配置文件**（比如 `application.yml`）：<u>放所有环境通用的公共配置，是全局生效的。</u>

  - ```yaml
    # 主配置：所有环境都通用的公共配置
    server:
      port: 8080  # 所有环境都用 8080 端口，写在这里即可
    
    logging:
      level:
        root: info  # 日志级别全局统一
    
    spring:
      # 指定默认激活的环境
      profiles:
        active: dev #可更改
    ```

- **环境配置文件**（比如 `application-dev.yml` / `application-prod.yml`）：<u>放不同环境需要不一样的配置，是局部生效的，并且会覆盖主配置里的同名属性。</u>

  - **开发环境配置(application-dev.yml):**

    - ```yaml
      # 开发环境：仅开发时生效，会覆盖主配置的同名属性
      spring:
        datasource:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://localhost:3306/scenic_dev?useSSL=false&serverTimezone=UTC
          username: root
          password: 123456  # 开发环境密码简单
      ```

  - **生产环境配置(application-prod.yml):**

    - ```yaml
      # 生产环境：仅线上生效，会覆盖主配置的同名属性
      spring:
        datasource:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://prod-db-server:3306/scenic_prod?useSSL=true&serverTimezone=UTC
          username: scenic_admin
          password: ********  # 线上密码复杂且不写死在代码里（实际会用环境变量注入）
      ```

- 可以使用独立配置文件定义环境属性
- 独立配置文件便于线上系统维护更新并保障系统安全性

启动时指定环境，SpringBoot 会自动合并主配置 + 对应环境的配置，环境配置的同名属性会覆盖主配置的值。

#### YW-3-3.多环境开发（properties多文件版）_了解

**主配置文件**

```properties
spring.profiles.active=pro
```

**环境配置文件**

**application-pro.properties**

```properties
server.port=80
```

**application-dev.properties**

```properties
server.port=81
```

文件的命名规则为：application-环境名.properties。

与yaml文件不同的是,**properties文件多环境配置仅支持多文件格式**;

#### YW-3-4.多环境开发独立配置文件书写技巧

将所有的配置根据功能对配置文件中的信息进行拆分，并制作成独立的配置文件，命名规则如下

- application-**devDB**.yml
- application-**devRedis**.yml
- application-**devMVC**.yml

多环境开发使用**group**属性设置配置文件分组,便于线上维护管理

```yaml
Spring:
	profiles:
		active:dev
		group:
			"dev":devDB,devRedis,devMVC
			"pro":proDB,proRedis,proMVC
			"test":testDB,testRedis,testMVC
```

注意:**当主环境dev与其他环境有相同属性时，主环境属性生效；其他环境中有相同属性时，最后加载的环境属性生效**



#### YW-3-5.多环境开发控制

**作用:**<font color="red">**让项目在 开发电脑,测试服务器,生产服务器自动切换不同配置,不用手动改代码,不用手动改配置文件,打包带时候指定环境就行**</font>

- **Maven**:项目构建管理工具,最终负责打包,构建整个工程,是大BOSS
- **SpringBoot**:是简化开发的框架,运行在Maven构建的工程之上,是员工

环境的控制权由Maven主导,SPringBoot直接读取Maven配置

**第一步:配置Maven多环境**

```xml
<profiles>
	<profile>
		<id>env_dev</id>
		<properties>
			<!-- 定义一个属性,用来表示当前环境 -->
			<profile.active>dev</profile.active>
		</properties>
		<activation>
			<!-- 默认激活开发环境 -->
			<activeByDefault>true</activeByDefault>
		<activation>
	</profile>
	<profile>
		<id>env_pro</id>
		<properties>
			<profile.active>pro</profile.active>
		</properties>
	</profile>
</profiles>
```

**说明:**

- 定义了env_dev和env_pro两个环境
- 每个环境都定义了一个共同的属性 profile.active 值分别为dev和pro
- env_dev默认被激活

**第二步:让SpringBoot读取Maven配置**

在application.yml文件里,让SpringBoot直接读取Maven的profile.active属性

```yaml
spring:
	profiles:
		active:@profile.active@
```

**第三步:效果验证**

1. **默认**:Maven默认激活env_dev,所以构建以后,spring.profiels.active就会被替换成dev,SpringBoot自动使用开发环境配置
2. **切换到生产环境**:执行**`mvn clean package -P env_pro`**命令,激活**env_pro profile**,构建后,spring.profiles.active就会被替换成pro,SpringBoot自动使用生产环境配置

**注意:**在 IDEA 中开发测试时，修改了 `pom.xml` 的 profile 配置后，**需要手动执行一次 `compile` 或 `package` 构建**，`application.yml` 里的占位符才会被替换成最新的值，否则配置不会生效。

**总结**

1. 当Maven与SpringBoot同时对多环境进行控制,以Maven为主,SpringBoot使用@...@占位符读取Maven对应的配置属性值
2. 基于SpringBoot读取Maven配置的属性前提下,如果在Idea下测试工程时候pom.xml每次更新需要手动compile方可生效



### YW-4.日志

#### YW-4-1.代码中使用日志工具记录日志

- **日志级别详解:**

| 级别    | 用途                              | 使用场景                           |
| ------- | --------------------------------- | ---------------------------------- |
| `TRACE` | 运行堆栈信息，最细粒度            | 生产环境几乎不用                   |
| `DEBUG` | 程序员调试代码用                  | 开发环境常用，记录变量、流程细节   |
| `INFO`  | 记录正常运维数据                  | 线上环境默认级别，记录关键业务节点 |
| `WARN`  | 记录潜在风险 / 异常但不影响主流程 | 比如参数过期、资源不足预警         |
| `ERROR` | 记录错误信息和堆栈                | 程序报错时必须记录，方便定位问题   |
| `FATAL` | 灾难级错误（合并入 ERROR）        | 系统崩溃、服务不可用               |

- **代码中记录日志**

  ```java
  @Slf4j
  @RestController
  @RequestMapping("/books")
  public class BookController extends BaseClass{
  	@GetMapping
  	public String getById(){
  		log.debug(...);
  		log.info(...);
  		log.warn(...);
  		log.error(...);
  		return "springboot is running ...2";
  	}
  }
  ```

- **日志级别配置**(细粒度控制)

- ```yaml
  logging:
  	#自定义日志组
  	group:
  		myapp:com.itheima.controller,com.itheima.service
  	level:
  		#全局默认warn级别
  		root:warn 
  		#给组设置级别
  		myapp:info
  		#直接给包设置级别
  		com.itheima.controller:debug 
  ```

- **全局**(root):整个项目所有代码的默认日志级别
- **包**(package):给某一个具体包单独设置日志级别-----正在调试某个模块,只想看这个包的日志
- **组**(group):把多个包打包成一组,统一设置级别
- **优先级:包>组>全局**

**总结:**

- 常用日志级别:`debug`<`info`<`warn`<`error`,级别越高输出越少
- 推荐用Lombok的@slf4j注解简化日志对象创建
- 日志级别支持全局,按组,按包三种方式配置,企业开发优先用细粒度控制



### YW-4-2.日志输出格式

![image-20260419144440353](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260419144440353.png)



### YW-4-3.日志文件

线上环境日志量很大，单个文件会越来越大，导致：

- 日志文件占满服务器磁盘
- 查找日志效率极低
- 无法按日期归档历史日志

日常开发:程序运行后，会在项目根目录生成 `server.log` 文件

```yaml
logging:
  file:
    name: server.log
```

所以企业级项目必须配置**日志滚动**，按时间 / 大小拆分日志文件。日志文件的常用配置方式：

```yaml
logging:
  file:
    name: logs/app.log  # 指定日志文件目录，统一管理
  logback:
    rollingpolicy:
      max-file-size: 100MB
      # 日志文件保留天数，避免磁盘被占满
      max-history: 30
      # 总日志文件大小上限
      total-size-cap: 10GB
      file-name-pattern: logs/app.%d{yyyy-MM-dd}.%i.log.gz # 自动压缩归档
```

`%i`：序号变量，同一天内文件超过 `max-file-size` 时，序号递增生成新文件

- 比如：`server.2026-04-19.0.log`、`server.2026-04-19.1.log`