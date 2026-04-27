# SpringBoot开发实用篇_笔记

- 热部署
- 配置高级
- 测试
- 数据层解决方案
- 整合第三方技术
- 监控



## Chapter1_热部署

**热部署**:自动加载更新过的程序

**SpringBoot 热部署的核心实现依赖`spring-boot-devtools`组件**：它会监听项目文件的变化，当代码 / 配置更新时，通过专用的类加载器实现**部分重启**，仅重新加载用户编写的业务代码，不重启整个 Spring 容器和内置 Tomcat，从而实现快速热部署。

### 1-1:手动启动热部署

**步骤①:导入开发者工具对应的坐标**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

**步骤②:构建项目,可以使用快捷键激活此功能(Ctrl+F9)**

![image-20260420182656839](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260420182656839.png)

#### 1-2:热部署底层原理

springboot项目运行分为两个部分,根据加载类型不同分为两大类:

- **base类加载器**:加载jar包中的类,其加载内容不会发生变化
- **restart类加载器**:加载开发者制作的内容

项目启动时,base类加载器执行,加载jar包信息,restart类加载器执行,加载开发者制作的内容,**热部署是销毁旧的restart类加载器,新建一个重新加载你的代码**



<u>难道一定要手动吗?当然不用,接下来,我们重磅推出:**自动!!!!!!!**</u>

### 1-2.自动启动热部署

**步骤①**：设置自动构建项目

​		打开【File】，选择【settings...】,在面板左侧的菜单中找到【Compile】选项，然后勾选【Build project automatically】，意思是自动构建项目

![image-20260420184051094](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260420184051094.png)

**步骤②**：允许在程序运行时进行自动构建

1. 打开 `File → Settings → Advanced Settings`
2. 找到 `Compiler` 分类
3. 勾选 `Allow auto-make to start even if application is running`
4. （和原来 Registry 里的选项是同一个功能，只是挪位置了）
5. 保存设置，重启 IDEA 就生效了。

**有什么用?**:你刷新浏览器 / 接口，就能看到修改后的效果，**全程不用手动按任何键，也不用重启项目**

**<font color="red">注意:</font>**要引入devtools依赖才有用噢~



### 1-3.参与热部署监控的文件范围配置

不是所有文件修改过后都会激活热部署,通过配置可以修改不参与热部署的文件或目录

```yaml
spring:
  devtools:
    restart:
      # 设置不参与热部署的文件或文件夹
      exclude: static/**,public/**,config/application.yml
```

如果想激活热部署了,就从exclude里面移除掉就行,但是不建议,这些静态资源,模板文件,没有必要让他触发重启



### 1-4.关闭热部署

项目在线上运行时,为了防止恶意修改代码,保证稳定,安全,就需要关闭热部署,可以通过关闭热部署功能降低线上程序的资源消耗:

**方式一:配置文件(推荐)**

```yaml
spring:
	devtools:
		restart:
			enabled: false #关闭热部署
```

**方式二:代码强制关闭**:通过系统属性强制关闭,任何配置都无法覆盖

```java
@SpringBootApplication
public class SSMPApplication{
	public static void main(String[] args){
		System.setProperty("spring.devtools.restart.enabled","false");
		SpringApplication.run(SSMPApplication.class,args);
	}
}
```

**总结:**

1. **热部署是开发工具，不是生产工具**：开发时用，方便你改代码；线上必须关。

2. **关闭方式**：

   - 配置文件：`enabled: false`

   - 代码强制：`System.setProperty(..., "false")`

3. **核心目的**：保证线上安全、稳定、节省资源。



## Chapter2_配置高级

#### 2-1.@ConfigurationProperties-把yml变成一个Java对象

**前期回顾:**

配置文件:

```yml
my:
	name: "Nancy"
	age: 20
```

自定义配置类:

```java
@Data
@Component 
@ConfigurationProperties(prefix="my")
public class MyProps{
	private String name;
	private Integer age;
}
```

这里如果不用@Component,在启动类/配置类上面加一个@EnableConfigurationProperties(MyProps.class)效果一样

**疑问:为什么配置类里推荐用Integer,而不是int?**

答:因为int有默认值0,Integer默认是null,假如Java里面是int类型,结果是0,那我到底是配置了还是没配置?如果是Integer,默认是null,一看就知道根本没配置age

**疑问:@Component干嘛来着?**

把当前类交给Spring管理,变成一个Bean,变成Bean以后,才能@Autowired注入,让Spring帮我创建对象,赋值

业务类里面注入使用:

```java
@Service
public class UserService{
	@Autowired
	private MyProps myProps;
	public void test(){
		sout(myProps.getName());
		sout(myProps.getAge());
	}
}
```



之前在基础篇学习了**@ConfigurationProperties**----把配置文件(application.yml/application.properties)里的配置,自动绑定到Java类的属性上,前面学习我们用的是自定义配置类,接下来我们来加载第三方的Bean

第三方的bean:比如RedisTemplate,DruidDataSource,RestTemplate,这些源码我改不了,**不能进去加@Component**

**步骤①:**使用@Bean注解定义第三方bean,把DruidDataSource变成Bean

```java
@Bean
public DruidDataSource datasource(){
	DruidDataSource ds=new DruidDataSource();
	return ds;
}
```

**步骤②:**在yml中定义要绑定的属性,注意datasource此时全小写

```yml
datasource:
	driverClassName: com.mysql.jdbc.Driver
```

**步骤③:**使用@ConfigurationProperties注解为第三方bean进行属性绑定,把.yml文件里的datasource配置塞进去,注意前缀是全小写的datasource

```java
@Bean
@ConfigurationProperties(prefix="datasource")
public DruidDataSource datasource(){
	DruidDataSource ds=new DruidDataSource();
	return ds;
}
```

与自定义操作方式一样,只不过此时注解不仅可以添加到类上,还可以添加到方法上,给其返回的对象绑定属性,本质一样,比如上面这个就是给ds对象绑定了driverClassName的属性

问题是:@bean既可以写在类上也可以写在方法上,想去找配置类的时候不好找,因为其位置太过分散,如何解决呢?



### **@EnableConfiguration**

**步骤①:**在配置类上开启@EnbleConfigurationProperties注解,标注要使用@ConfigurationProperties注解绑定属性的类

```java
@Configuration
@EnableConfigurationProperties({
    ServerConfig.class,
    DatabaseConfig.class,
    RedisConfig.class
})
public class AppConfig {
    // 这里什么都不用做，只需要把上面的类列出来
}
```

- 一眼看清所有配置类,不用一个个打开文件去看有没有Component
- 想关闭某个配置,直接注释掉就行

**步骤②:**在对应类上直接开启@ConfigurationProperties进行属性绑定

```java
@Data
@ConfigurationProperties(prefix="servers")//把配置文件servers开头的属性,绑定到这个类的字段上
public class ServerConfig{ //自定义配置绑定类
	private String ipAddress;
	private int port;
	private long timeout;
}
```

当使用@EnableConfigurationProperties声明进行属性绑定的bean后，无需使用@Component注解再次进行bean声明



### 2-2.宽松绑定/松散绑定

什么是宽松绑定,就是配置类属性和Java类属性字段名不必一模一样,可以某种程度上改变,大大增加了编程的灵活性,不必死板的一模一样

举个例子:

```java
@Component
@Data
@ConfigurationProperties(prefix="servers")
public class ServerConfig{
	private String ipAddress;
}
```

里面的 `ipAddress`可与以下形式的配置属性名全兼容

```yml
servers:
	ipAddress: xxx  #驼峰
	ip_address: xxx #下划线
	ip-address: xxx #烤肉串(推荐)
	IP_ADDRESS: xxx #常量
```

可以看出,匹配的时候,忽略下划线,中划线,大小写字母,这就是对格式上进行宽松处理,所以叫做"**宽松绑定**",不过,这个绑定模式只适用于@Configuration,对@Value可是无效的



### 2-3.常用计量单位绑定

```yml
servers:
	timeout: -1
```

如上,-1代表永不过时,假如我改成 240 那么是240秒,还是240分钟?小时?怎么规定呢?

```java
@Component
@Data
@ConfigurationProperties(prefix="servers")
public class ServerConfig{
	@DurationUnit(ChronoUnit.HOURS)
	private Duration serverTimeOut;
	@DataSizeUnit(DataUnit.MEGABYTES)
	private DataSize dataSize;
}
```

- **Duration:**表示时间间隔,通过 `DurationUnit`描述时间单位,上例代表小时
- **DataSize**:表示存储空间,通过 `DataSizeUnit`描述存储空间单位,上例代表MB

Duration常用单位:![image-20260422201922275](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260422201922275.png)

DataSize常用单位:![image-20260422201937901](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260422201937901.png)



### 2-4.校验

写配置的时候,可能配置的属性值与代码中的数据类型冲突,比如说配置里面是String,代码里写的int,对此,我们需要一个校验框架对数据进行校验,这里我们用Hivernate提供的框架

**步骤①:**开启校验框架,注入依赖

```xml
<!--1.导入JSR303规范-->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
</dependency>
<!--使用hibernate框架提供的校验器做实现-->
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```

**步骤②:**在需要开启校验功能的类上使用注解@Validated开始校验功能

```Java
@Component
@Data
@ConfigurationProperties(prefix="server")
@Validated //对当前bean的属性注入校验
public class ServerConfig{

}
```

**步骤③**:对具体字段设置校验规则

```Java
@Component
@Data
@ConfigurationProperties(prefix="server")
@Validated //对当前bean的属性注入校验
public class ServerConfig{
	@Max(value=8888,message="最大值不超过8888")
	@Min(value=202,message="最小值不能低于202")
	private int port;
}
```

**总结**

1. 开启Bean属性校验功能一共3步：导入JSR303与Hibernate校验框架坐标、使用@Validated注解启用校验功能、使用具体校验规则规范数据校验格式



### 2-5.数据类型转换

在配置文件书写时,对于数字的定义支持进制书写格式,如需使用字符串,请使用**引号**明确标注

![image-20260422205233813](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260422205233813.png)



## Chapter3_测试

### 3-1.加载测试专用属性

1. **临时属性:**

   - 使用注解@SpringBootTest的properties属性就可以为当前测试用例添加临时的属性,覆盖源码配置文件中对应的属性值进行测试.<u>这样就不用总是在application.xml文件里总是改来改去了</u>

     ```java
     //properties属性可以为当前测试用力添加临时的属性配置
     
     @SpringBootTest(properties={"test.prop=testValue1"})
     public class PropertiesAndArgsTest{
     	@Value("${test.prop}")
     	private String msg;
     	
     	@Test
     	void testProperties(){
     		System.out.println(msg);
     	}
     }
     ```

2. **临时参数:**

   - 运维不改代码，不改配置文件，他们只通过命令行参数启动程序，用来覆盖配置，这就是为什么 Spring Boot 要提供 args 的原因。

     ```java
     //properties属性可以为当前测试用力添加临时的属性配置
     
     @SpringBootTest(properties={"--test.prop=testValue1"}) //多了横线'--'
     public class PropertiesAndArgsTest{
     	@Value("${test.prop}")
     	private String msg;
     	
     	@Test
     	void testProperties(){
     		System.out.println(msg);
     	}
     }
     ```

如果两者都设置了不同的值,临时参数(也就是命令行)的**优先级更高**,会覆盖掉临时属性



### 3-2.加载测试专用配置

**步骤①:**在测试包test中创建专用的测试环境配置类(下列配置仅用于演示,实际开发不能这么注入String类型的数据)

```java
@Configuration
public class MsgConfig{
	@Bean
	public String msg(){
		return "bean msg";
	}
}
```

**步骤②**:启动测试环境时,导入测试环境专用的配置类,使用<u>@**import**注解</u>即可实现

```java
@SpringBootTest//告诉Spring这个是单元测试,要启动Spring容器
@Import({MsgConfig.class})//告诉Spring,在运行这个测试的时候,额外把MygConfig这个配置也加载进来
public class ConfigurationTest{

	@Autowired
	private String msg;
	
	@Test
	void testConfiguration(){
		System.out.println(msg);
	}
}
```

- 不用`@Import`：要给测试加 Bean，只能改主配置，或者写条件注解，很麻烦，还容易影响别人。
- 用`@Import`：只在测试里写配置，只在测试里生效，干净又灵活。



### 3-3.Web环境模拟测试

**为什么要模拟Web环境?**

只想测试一个controller的接口,不想重启项目耗费太多时间,内存(Controller时Web层代码,依赖Servlet,请求,响应,接口路径,必须有Web环境才能跑)

对表现层功能进行测试:

1. 一个基础:运行测试程序的时候,启动web环境
2. 一个功能:发送web请求

**1.测试类中启动web环境**

@SpringBootTest注解中有一个属性:**webEnvironment**,可以指定启动web环境的对应端口

```java
@SpringBootTest(webEnvironment= SpringBootTest.webEnvironment.RANDOM_PORT)
public class WebTest{
}
```

![image-20260424191602941](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260424191602941.png)

- MOCK:适配性配置:当测试用到Servlet API时,就会启动轻量模拟Web环境,否则自动降级,不启动Web环境
- DEFINED_PORT: 使用自定义端口作为web服务器端口,模拟真实运行环境
- RANDOM_PORT: 使用随机端口作为web服务器端口(<u>推荐</u>),避免端口冲突
- NONE: 不启动web环境,只测试Service,Mapper,不碰Web相关代码

**2.测试类中发送请求**

**步骤一:**在测试类中开启web虚拟调用功能,通过注解@AutoConfigureMockMvc实现此功能的开启

```java
@SpringBootTest(webEnvironment=SpringBootTest.webEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc //开启虚拟MVC调用
public class webTest{
}
```

**疑问:什么是MVC?**

MVC是一种软件设计模式,核心就是把代码按职责分成3部分

- Model:数据和业务逻辑
- View:视图-展示给用户看的部分
- Controller:接受请求,协调Model和View

同一个Model可以给多个View用,比如同一套业务逻辑手机App和网页端可共用,**分工清晰**,**解耦**:修改界面不用动业务逻辑,修改业务逻辑也不用动界面

**步骤二:**定义发起虚拟调用的对象MockMVC,通过自动装配的形式初始化对象

```Java
@SpringBootTest(webEnvironment=SpringBootTest.webEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc //开启虚拟MVC调用
public class webTest{
    @Test
    void testWeb(@Autowired MockMVC mvc){
    }
}
```

**步骤三:**创建一个虚拟请求对象,封装请求的路径,并使用MockMVC对象发送对应请求

```java
@SpringBootTest(webEnvironment=SpringBootTest.webEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc //开启虚拟MVC调用
public class webTest{
    @Test
    void testWeb(@Autowired MockMVC mvc)throw Exception{
    	//http://localhost:8080/books
        //创建虚拟请求,当前访问/books
    	MockHttpServletRequestBuilder builder=MockMvcRequestBuilders.get("/books");
        //执行对应请求
    	mvc.perform(builder);
    }
}
```

路径写/books就行,不用写前面那些端口什么的,应为当前是虚拟的web环境,无需指定

**总结:**

- 进行测试web层接口的时候要保障测试类启动时启动web容器,使用@SpringBootTest注解的webEnvironment属性可以虚拟web环境用于测试
- 为测试方法注入MockMvc对象,通过这个对象可以发送虚拟请求,模拟web请求调用过程



#### **web环境请求结果对比**

既然模拟出了web环境也发送了web请求,那么如何检验测试结果对不对呢?

- 响应状态匹配

```java
@Test
void testStatus(@Autowired MockMvc mvc)throws Exception{
    MockHttpServletRequestBuilder builder=MockMvcRequestBuilders.get("/books");
    
}
```

- 响应体匹配(json数据格式,开发中的主流)

```java
@Test
void testJson(@Autowired MockMvc mvc)throws Exception{
    MockHttpServletRequestBuilder builder=MockMvcReuqestBuilder.get("/books");
    ResultActions action=mvc.perform(builder);
    //设定预期值 与真实值进行比较,成功测试通过,失败测试失败
    ContentResultMatchers content=MockMvcResultMatchers.content();
    // 预期返回的JSON是 {"id":1,"name":"springboot2","type":"springboot"}
    ResultMatcher result = content.json("{\"id\":1,\"name\":\"springboot2\",\"type\":\"springboot\"}");
    //添加预计值到本次调用过程中进行匹配
    action.andExpect(result);
}
```

- 响应体信息匹配

```java
@Test
void testHeader(@Autowired MockMvc mvc)throws Exception{
	MockHttpServletRequestBuilder builder=MockMvcRequestBuilders.get("/books");
	ResultAction action=mvc.perform(builder);
	
	HeaderResult header=MockMvcResultMatchers.header();
    // 预期响应头的 Content-Type 是 application/json
	ResultMatcher contentType=header.string("Content-Type","application/json");
	action.andExpect(contentType);
}
```

以上头信息,正文信息,状态信息都有了,恶意组合出一个完美的响应结果对比了

```java
@Test
void testGetById(@Autowired MockMvc mvc)throws Exception{
	MockHttpServletRequestBuilder builder=MockMvcRequestBuilders.get("/books");
	ResultAction action=mvc.perform(builder);
	
	StatusResultMathcers status=MockMvcResultMatchers.status();
	ResultMatcher ok=status.isOk();
	action.andExpect(ok);
	
	HeaderResultMathcers header=MockMvcResultMatchers.header();
	ResultMatcher contentType=header.string("Content-Type","application/json");
	action.andExpect(contentType);
	
	ContentResultMatchers content=MockMvcResultMathcers.content();
	ResultMatcher result=content.json("{\id}")
}
```

**教你一招:复制工程**

1.在文件夹事先把模板模块复制一份,记得把description删掉,还有artifactId改掉

2..点击project structure,添加import moldule,

3.导入响应的模块,apply->OK,就行啦!



### 3-4.数据层测试回滚

测试过程中会产生一些垃圾数据遗留在数据库中,所以我们需要解决这个问题,避免数据库的影响

**@Transactional**:在原始测试用例中添加注解@Transactional就可以实现当前测试用例的事务不提交,springboot会认为这是一个测试程序,无需提交事务,就可以避免事务的提交,并且这个注解常常与@Rollback搭配使用,只不过系统会默认@Rollback(true),即默认回滚

​		如果开发者想提交事务，也可以，再添加一个@RollBack的注解，设置回滚状态为false即可正常提交事务，是不是很方便？springboot在辅助开发者日常工作这一块展现出了惊人的能力，实在太贴心了。

```java
@SpringBootTest
@Transactional
@Rollback(true) //系统默认是true
public class DaoTest {
    @Autowired
    private BookService bookService;

    @Test
    void testSave(){
        Book book = new Book();
        book.setName("springboot3");
        book.setType("springboot3");
        book.setDescription("springboot3");

        bookService.save(book);
    }
}
```

**总结:**

1. 在springboot的测试类中通过添加注解@Transactional来阻止测试用例提交事务
2. 通过注解@Rollback控制springboot测试类执行结果是否提交事务,需要配合注解@Transactional使用



但是这样的数据都是写死的,当然不可以这样,测试用例数据只需要随机产生,就🆗

### 3-5.测试用例数据设定

在测试文件里面写下如下代码;

```yml
testcase:
  book:
    id: ${random.int}
    id2: ${random.int(10)}
    type: ${random.int!5,10!}
    name: ${random.value}
    uuid: ${random.uuid}
    publishTime: ${random.long}
```

当前配置每次运行数据时要创建一组随机数据,避免每次运行时数据都是固定值的尴尬现象发生,有助于测试功能的进行.数据的加载按照之前加载数据的形式,使用@ConfigurationProperties注解即可

```java
@Component
@Data
@ConfigurationProperties(prefix="testcase.book")
public class BookCase{
	private int id;
	private int id2;
	private int type;
	private String name;
	private String uuid;
	private long publishTime;
}
```

还可以对于随机值指定小的限定规则

![image-20260426180815484](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260426180815484.png)

- ${random.int}表示随机整数
- ${random.int(10)}表示10以内的随机数
- ${random.int(10,20)}表示10到20的随机数
- 其中()可以是任意字符，例如[]，!!均可





## 数据层解决方案

- 数据源技术:Druid
- 持久化技术:MyBatisPlus
- 数据库技术:MySQL



### 数据源技术

如果开发者没有设置数据源技术(比如我们之前用的Druid),Springboot会提供给3款内嵌数据源技术:

- HikariCP  (springboot官方推荐)
- Tomcat(提供DataSource)
- Commons DBCP

如何使用?把我们之前数据库配置代码的 `duird`注释掉就行, 见下面的代码

```yml
spring:
	datasource:
		url:xxx
		driver-class-name:xxx
		username:xxx
		password:xxx
```

这样子用的就是默认数据源HiKariCP,如果要在配置文件上面写上 `HiKariCP`,就要把url隔开,还可以添加其他独立的属性

```java
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
    hikari:
      driver-class-name: com.mysql.cj.jdbc.Driver
      username: root
      password: root
      maximum-pool-size: 50
```

其他数据源都是如此,以此类推,后面我们做数据层,数据源对象的选中就不再是单一的使用druid数据源技术,需要自行选择



### 持久化技术

**疑问:什么是持久化?**

把内存中瞬时的数据,保存到磁盘/数据库/文件等永久存储设备中,保存程序重启,断电后数据不丢

**常见场景:**

1. Java:对象序列化,IO文件读写,Mybatis/MyBatis-Plus操作数据库
2. 缓存:Redis持久化(RDB/AOF)防止宕机丢缓存数据
3. 日常:保存文档,聊天记录,用户账号信息

**JdbcTemplate**:springboot自己提供的数据层技术,就是回归jdbc最原始的变成形式进行数据层开发

**步骤一:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency
```

**步骤二:**自动装配jdbcTemplate对象

```Java
@SpringBootTest
class Springboot15SqlApplicationTests {
    @Test
    void testJdbcTemplate(@Autowired JdbcTemplate jdbcTemplate){
    }
}
```

**步骤三:**使用JdbcTemplate实现查询操作(非实体类封装数据的查询操作)

```Java
@Test
void testJdbcTemplate(@Autowired JdbcTemplate jdbcTemplate){
    String sql = "select * from tbl_book";
    List<Map<String, Object>> maps = jdbcTemplate.queryForList(sql);
    System.out.println(maps);
}
```

**步骤四:**使用JdbcTemplate实现查询操作(实体类封装数据的查询操作)

```java
@Test
void testJdbcTemplate(@Autowired JdbcTemplate jdbcTemplate){

    String sql = "select * from tbl_book";
    RowMapper<Book> rm = new RowMapper<Book>() {
        @Override
        public Book mapRow(ResultSet rs, int rowNum) throws SQLException {
            Book temp = new Book();
            temp.setId(rs.getInt("id"));
            temp.setName(rs.getString("name"));
            temp.setType(rs.getString("type"));
            temp.setDescription(rs.getString("description"));
            return temp;
        }
    };
    List<Book> list = jdbcTemplate.query(sql, rm);
    System.out.println(list);
}
```

**步骤五:**增删改操作

```Java
@Test
void testJdbcTemplateSave(@Autowired JdbcTemplate jdbcTemplate){
    String sql = "insert into tbl_book values(3,'springboot1','springboot2','springboot3')";
    jdbcTemplate.update(sql);
}
```

相对JdbcTemplate对象进行相关配置,可以在yml文件中进行设定:

```yaml
spring:
  jdbc:
    template:
      query-timeout: -1   # 查询超时时间
      max-rows: 500       # 最大行数
      fetch-size: -1      # 缓存行数
```



### 数据库技术

springboot同样也提供了内嵌的数据库技术

- H2
- HSQL
- Derby

这三种技术除了可以独立安装之外,还可以像是tomcat服务器一样,采用内嵌的形式运行在springboot容器中,内嵌在容器中运行

**应用场景:**   方便进行功能测试测试结束,数据和数据库一并清空,**注意:项目上显示必须把H2关掉,换回MySQL,保证数据不会丢**

到这里,SQL相关技术就讲完了,总结一下

- 数据源技术:Druid,Hikari,tomcat DataSource,DBCP
- 持久化技术:MyBatisPlus,MyBatis,JdbcTemplate
- 数据库技术:MySQL,H2,HSQL,Derby



