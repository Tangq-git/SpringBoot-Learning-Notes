# SpringBoot学习笔记

### 一.快速上手SpringBoot

#### 1.SpringBoot概述:

- **parent**
  - 开发SpringBoot程序要继承Spring-boot-starter-parent
  - spring-boot-starter-parent中定义了若干个依赖管理
  - 继承parent模块可以**避免**多个依赖使用相同技术时出现以来**版本冲突**
  - 继承parent的形式也可以采用引入依赖的形式实现效果
- **starter**
  - 开发SpringBoot程序需要导入坐标时通常导入对应的starter
  - 每个starter根据功能不同,通常包含多个依赖坐标
  - 使用starter可以实现快速配置的效果,达到简化配置的目的
- **引导类**
  - SpringBoot工程提供引导类用来启动程序,运行main方法就可以启动项目,例如:<u>SpringBootApplication</u>
  - SpringBoot工程启动后创建并初始化Spring容器,扫描引导类所在包加载bean
- **辅助功能(内嵌tomcat)**
  - 1.内嵌Tomcat服务器是SpringBoot辅助功能之一
  - 2.内嵌Tomcat工作原理是将**Tomcat服务器作为对象运行,并将该对象交给Spring容器管理**
  - 3.变更内嵌服务器思想是去除现有服务器,添加全新服务器
- **Idea中隐藏指定文件或指定类型文件**
  - Setting->File Types->Ignored Files and Folders
  - 输入要隐藏的文件名,支持*号通配符
  - 回车确认添加

#### [补]知识加油站:Rest开发

#### 1.Rest简介

- Representational State Transfer,表现形式状态转换

- 优点:

  - 隐藏资源的访问行为,无法通过地址得知对资源是何种操作

    ```
    http://localhost/user/getById?id=1
    
    http://localhost/user/1
    ```

- 操作行为:
  - GET(查询)  http://localhost/users
  - POST(新增/保存) http://localhost/users/1
  - PUT(修改/更新) http://localhost/users
  - DELETE(删除) http://localhost/users/1

#### 2.RESTful入门案例

- Rest风格
  - **@RequestParam:**用于接收url地址传参或表单传参
  - **@RequestBody**:用于接收json数据
  - **@PathVariable**:用于接收路径参数,使用{参数名称}描述路径参数

- 应用:
  - 后期开发中,发送请求参数超过1个时候,以json格式为主,@RequestBody应用较广
  - 如果发送非json格式数据,选用@RequestParam接收请求参数
  - 采用RESTful进行开发,当参数数量较少时,例如1个,采用@PathVariable接收请求路径变量,通常用于传递id值

#### 3.REST快速开发

- **名称:@RestController**
  
  - 类型:**类注解**
  - 位置:基于SpringMVC的RESTful开发控制器类定义上方
  - 作用:设置当前控制器类为RESTful风格,等同于@Controller与@ResponseBody两个注解组合功能
  
- **标准请求动作映射**

  - 名称:@Get/Post/Put/DeleteMapping

  - 类型:**方法注解**

  - 位置:基于SpringMVC的RESTful开发控制器方法定义上方

  - 作用:设置当前控制器方法请求访问路径与请求动作,每种对应一个请求动作,例如@GetMapping对应GET请求

  - 范例:

    - ```java
      @GetMapping("/{id}")
      public String getById(@PathVariable Integer id){
      	System.out.println("....");
      	return "{....}";
      }
      ```

  - 属性:value(默认):请求访问路径

### 二.基础配置

- **复制工程:**
  1. 在工作空间中复制对应工程,并修改工程名称
  2. 删除与idea相关配置文件,仅保留src目录与pom.xml文件
  3. 修改pom.xml文件中的artifactId与新工程/模块名相同
  4. 删除name标签(可选)
  5. 保留备份工程供后期使用

- **属性配置**

  - SpringBoot默认配置文件application.properties,通过**键值对**配置对应属性
  - SpringBoot内置属性查询:https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties:官方参考文档第一项:Application Properties
  - SpringBoot导入对应starter后,提供对应配置属性
  - 书写SpringBoot配置采用关键字+提示形式书写

- **配置文件分类**

  - properties(优先级最高)
  - <u>**yml**:主流</u>
  - yaml(优先级最低)
  - 不同配置文件中相同配置按照加载优先级相互覆盖,不同配置文件中不同配置**全部保留**

- **yaml文件**

  - 举例

    ```YAML
    subject:
    	- Java
    	- 前端
    	- 大数据
    enterprise:
    	name: itcast
    	Name: iteima #大小写敏感
        age: 16
        subject:
        	- Java
            - 前端
            - 大数据
    likes: [王者荣耀,刺激战场]			#数组书写缩略格式
    users:							 #对象数组格式一
      - name: Tom
       	age: 4
      - name: Jerry
        age: 5
    users:							 #对象数组格式二
      -  
        name: Tom
        age: 4
      -   
        name: Jerry
        age: 5			    
    users2: [ { name:Tom , age:4 } , { name:Jerry , age:5 } ]	#对象数组缩略格式
    ```


  - 注意属性名冒号后面与数据之间有一个**空格**

- **yaml数据读取**

  - **读取单一数据**

    - yaml中保存的单个数据，可以使用Spring中的注解直接读取，使用@Value可以读取单个数据，属性名引用方式：<font color="#ff0000"><b>${一级属性名.二级属性名……}</b></font>

      ![image-20211126180433356](file:///C:/Users/Tangq/Desktop/SpringBoot2/SpringBoot%E5%9F%BA%E7%A1%80%E7%AF%87%E2%80%94%E8%B5%84%E6%96%99/%E9%85%8D%E5%A5%97%E8%B5%84%E6%BA%90/img/image-20211126180433356.png?lastModify=1775831492)

  - **读取全部数据**
    - ![image-20260410223857210](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260410223857210.png)
  - **读取对象数据**
    
    - ![image-20260410224229347](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260410224229347.png)

- yaml文件数据引用

  - 在配置文件中可以使用${属性名}方式引用属性值

    - ```yaml
      baseDir: /usr/local/fire
      	center:
          dataDir: ${baseDir}/data
          tmpDir: ${baseDir}/tmp
          logDir: ${baseDir}/log
          msgDir: ${baseDir}/msgDir
      ```

  - 如果属性中出现特殊字符，可以使用双引号包裹起来作为字符解析

    - ```yaml
      lesson: "Spring\tboot\nlesson"
      ```

    #### **自动提示消失解决方案**

- Setting->project Structure->Facets
- 选择对应项目/工程
- Customize Spring Boot
- 选择配置文件

**---------------------------------------------------------------------------------------4.10----------------------------------------------------------------------------------------------**

### **三.基于SpringBoot实现SSMP整合**

#### **3-1.整合JUnit**:专门用来快速、单独测试 Java 方法的框架

**问题:**

- 为什么 Spring 原生要 `@RunWith(SpringJUnit4ClassRunner.class)`？
  - 让JUnit委托Spring来执行测试
- `@ContextConfiguration` 是干嘛的？
  - 加载Spring环境时要设置具体的环境配置
- SpringBoot 的 `@SpringBootTest` 为什么能替代这两个注解？
  - 其内部实现了这两个功能
- 什么情况下需要给 `@SpringBootTest` 加 `classes` 属性？
  - SpringBoot无法找到启动类或需要自定义加载配置的时候

**引导类必须放在项目的「根包」下**

- 比如根包是`com.company.project`，引导类就放在`com.company.project`下，不要放在子包（比如`com.company.project.controller`），否则会导致组件扫描不全，Bean 注入失败。

**一个项目只能有一个主引导类**

- 不能有多个`@SpringBootApplication`标记的类，否则启动会报错。

**测试类的包路径尽量和引导类一致**

- 这样可以省略`@ContextConfiguration`，代码更简洁，符合企业开发规范。

**总结:**

1. **导入测试对应的starter:**这个 starter 就是`spring-boot-starter-test`，在`pom.xml`里的完整依赖如下：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-test</artifactId>
       <scope>test</scope> <!-- 仅测试环境生效，不会打包到生产环境 -->
   </dependency>
   ```

2. **测试类使用 @SpringBootTest 修饰**:启动 SpringBoot 应用上下文，初始化 IOC 容器，让测试类能拿到 Spring 管理的 Bean（比如`@Autowired`注入的 Dao/Service）。
3. **使用自动装配的形式添加要测试的对象**:用`@Autowired`注解，把要测试的 Bean（比如`BookDao`、`UserService`）注入到测试类中，不用手动`new`对象。
4. **测试类如果存在于引导类所在包或子包中无需指定引导类**:测试类必须放在`src/test/java`下，包路径和主代码**完全一致**，比如主代码在`com.example.demo`，测试类也在`com.example.demo`，这样永远不用手动指定，代码最简洁。
5. **测试类如果不存在于引导类所在的包或子包中需要通过 classes 属性指定引导类**:
   
   - 直接给`@SpringBootTest`加`classes`属性
   - 搭配`@ContextConfiguration`注解

**实例:**

```java
package com.example.demo; // 必须和引导类同包/子包

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

// 步骤2：加@SpringBootTest
@SpringBootTest
class BookDaoTests {

    // 步骤3：@Autowired注入要测试的对象
    @Autowired
    private BookDao bookDao;

    @Test
    void testSave() {
        // 执行测试方法
        bookDao.save();
    }
}
```



### 3-2.整合MyBatis:专门帮Java程序跟数据库说话的框架



1. 整合操作需要勾选的MyBatis技术,也就是导入MyBatis对应的starter

   ![image-20260412143329309](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260412143329309.png)

2. 数据库相关信息转换成配置

   ![image-20260412143346077](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260412143346077.png)

   **mybatis后面每一个冒号后面必须加空格**

3. 数据库SQL映射需要添加@Mapper被容器识别到

4. MySQL 8.X驱动强制要求设置时区

   - 修改url,添加serverTimezone设定

     ```yaml
     #2.配置相关信息
     spring:
       datasource:
         driver-class-name: com.mysql.cj.jdbc.Driver
         url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
         username: root
         password: root
     ```

   - 修改MySQL数据库配置

5. 驱动类过时时,提醒更换为com.mysql.cj.jdbc.Driver

#### 3-3.整合MyBatis-Plus

**步骤①:导入对应的starter**:你在创建 SpringBoot 项目时，**不会自动勾选** MyBatis-Plus，需要手动在 `pom.xml` 里添加依赖（坐标）。

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
```

命名规范:

| starter所属 | 命名规则                                                    | 示例                                                  |
| ----------- | ----------------------------------------------------------- | ----------------------------------------------------- |
| 官方提供    | spring-boot-starter-技术名称                                | spring-boot-starter-web  spring-boot-starter-test     |
| 第三方提供  | 第三方技术名称-spring-boot-starter                          | mybatis-spring-boot-starter druid-spring-boot-starter |
| 第三方提供  | 第三方技术名称-boot-starter（第三方技术名称过长，简化命名） | mybatis-plus-boot-starter                             |

**步骤②:配置数据源相关信息**

```yaml
#2.配置相关信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/ssm_db
    username: root
    password: root
```

**映射接口(Dao)**:

```Java
@Mapper
public interface BookDao extends BaseMapper<Book> {
}
//不用写 XML 映射文件，也不用写 SQL 语句，直接继承了一套完整的增删改查 API。
```

- **企业替代方案**：在 SpringBoot 启动类上加 `@MapperScan("com.example.dao")`，可以统一指定扫描包，这样每个 Dao 接口就**不用单独写 `@Mapper`**，这是企业主流写法。
- 图片下方列出的方法（如 `insert`、`selectById`、`deleteBatchIds` 等）是 `BaseMapper` 为你预设的通用方法。你可以直接调用，无需手动实现：

![image-20260412153053116](C:\Users\Tangq\AppData\Roaming\Typora\typora-user-images\image-20260412153053116.png)

#### **企业开发核心规范与避坑**

##### 1. 实体类 Book 的规范

既然 `BaseMapper<Book>`，那么 `Book` 实体类必须按 MP 规范写，否则报错：

- 必须加 `@Data`（Lombok）或手动写 Get/Set。

- 如果表主键是自增的,主键字段必须加 `@TableId(type = IdType.AUTO)` 声明自增。

- ```Java
  @Data
  @TableName("t_book") // 对应数据库表名
  public class Book {
      @TableId(type = IdType.AUTO)
      private Long id;
      private String bookName;
      private BigDecimal price;
  }
  ```

- **依赖版本**：导入 MyBatis-Plus 依赖时，**不要指定 `<scope>test</scope>`**，因为它是主代码运行的核心包，必须参与编译。

在企业数据库设计中，为了区分不同模块的表，通常会给表加统一前缀。

- **数据库表名**：比如 `tbl_book`（书籍表）、`tbl_user`（用户表）。

- **实体类名**：你的 Java 实体类是 `Book`、`User`（**注意：类名不能带下划线 tbl_**）。

- **矛盾点**：如果不加配置，MP 会默认认为实体类 `Book` 对应表名 `book`，而实际表名是 `tbl_book`，执行 SQL 时会报错**表不存在**。

- ```yaml
  mybatis-plus:
    global-config:
      db-config:
        table-prefix: tbl_ # 设置所有表的通用前缀为 tbl_
  ```

  - **作用**：**全局统一规则**。MP 会自动帮你拼接前缀，你写实体类 `Book`，它会自动转换成 `tbl_book` 去查询数据库。

------

### 3-4.整合Druid

更换数据源:

1. 导入对应的技术坐标
2. 配置使用指定的数据源类型

**步骤①:导入对应的starter**

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.2.6</version>
    </dependency>
</dependencies>
```

**步骤②:修改application.yml配置**

```yaml
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
      username: root
      password: root
```

注意观察，配置项中，在datasource下面并不是直接配置url这些属性的，而是先配置了一个druid节点，然后再配置的url这些东西。

这是我们做的第4个技术的整合方案，还是那两句话：<font color="#ff0000"><b>导入对应starter，使用对应配置</b></font>。没了，SpringBoot整合其他技术就这么简单粗暴。

**总结**:

1. 整合Druid需要导入的starter
2. 根据Druid提供的配置方式进行配置
3. 整合第三方技术通用方式
   - 导入对应的starter
   - 根据提供的配置格式,配置非默认值对应的配置项

------

### 3-5.SSMP整合综合案例






