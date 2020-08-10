# 简介
Spring Cloud是一系列框架的有序集合,他利用spring boot的开发便利性巧妙的简化了分布式系统基础设施的开发,如`服务发现,配置中心,消息总线,负载均衡,断路器,数据监控等`,都可以用spring boot的开发风格做到一键启动和部署.spring cloud并没有重复造轮子,他只是将目前各家公司开发的比较成熟,经得起实际考验的服务框架组合起来,通过spring boot 风格进行再封装屏蔽掉了复杂的配置和实现原理,最终给开发者留出了一套简单易懂易部署和易维护的分布式系统开发工具包.
# 组件
* ## Spring Cloud Netflix 组件
    |  组件名称   | 作用  |
    |  :----  | :----  |
    | Eureka  | 服务注册中心 |
    | Ribbon  | 客户端负载均衡 |
    | Feign | 声明式服务调用 |
    | Hystrix | 客户端容错保护 |
    | Zuul | API网关服务 |
* ## Spring Cloud Alibaba 组件
    |  组件名称   | 作用  |
    |  :----  | :----  |
    | Nacos  | 服务注册中心 |
    | Sentinel | 客户端容错保护 |
* ## Spring Cloud 原生及其他组件
    |  组件名称   | 作用  |
    |  :----  | :----  |
    | Consul  | 服务注册中心 |
    | Gateway | API网关服务 |
    | Config | 分布式配置中心 |
    | Sleuth/Zipkin | 分布式链路追踪 |
# 实战
## 基础搭建
### 1. 创建父工程maven项目,修改pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.glh</groupId>
    <artifactId>spring_cloud_demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j>1.2.17</log4j>
        <lombok.version>1.16.18</lombok.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>1.5.9.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.0.4</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.0.31</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.3.0</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${lombok.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
### 2. 在父工程中创建子工程 Model =>Maven(microservicecloud-api)
```xml
<!-- 创建完成后会在父工程的pom中生成 -->
    <modules>
        <module>microservicecloud-api</module>
    </modules>
```
3. 在子工程中创建实体类和修改pom
```java
package cn.glh.entityes;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;
import java.io.Serializable;
@Data
@NoArgsConstructor
@Accessors(chain = true)//链式编程
@SuppressWarnings("serial")
public class Dept implements Serializable {
    private Long deptno;//主键
    private String dname;//部门名称
    private String db_source;//来自哪一个数据库,因为微服务架构可以一个服务对应一个数据库,同一个信息被储存到不同的数据库
    public Dept(String dname) {
        this.dname = dname;
    }
}
```
```xml
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```
### 3. 在父工程中创建子工程 Model =>Maven(microservicecloud-provider-dept-8001)
1. 创建配置文件
```yml
server:
  port: 8001
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在位置
  type-aliases-package: cn.glh.entityes                     # 所有Entity别名类所在包
  mapper-locations: classpath:mybatis/mapper/**/*.xml       # mapper映射文件
spring:
  application:
    name: microservicecloud-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01              # 数据库名称
    username: root
    password: 1218
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
```
2. 修改pom.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring_cloud_demo</artifactId>
        <groupId>cn.glh</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-provider-dept-8001</artifactId>

<dependencies>
    <dependency>
        <groupId>cn.glh</groupId>
        <artifactId>microservicecloud-api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
<!--    热部署-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>springloaded</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
</dependencies>
</project>
```
3. 根据application.yml的信息配置(mybatis.cfg.xml)
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
<!--        二级缓存开启-->
        <setting name="cacheEnabled" value="true"></setting>
    </settings>
</configuration>
```
4. 创建数据库
```sql
DROP DATABASE IF EXISTS cloudDB01;
CREATE DATABASE cloudDB01 CHARACTER SET UTF8;
USE cloudDB01;
CREATE TABLE dept(
deptno BIGINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
dname VARCHAR(60),
db_source VARCHAR(60)
);
INSERT INTO dept(dname,db_source) VALUES('开发部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('人事部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('财务部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('市场部',DATABASE());
INSERT INTO dept(dname,db_source) VALUES('运维部',DATABASE());
```
5. 创建实体类对应的Dao
```java
package cn.glh.dao;
import cn.glh.entityes.Dept;
import org.apache.ibatis.annotations.Mapper;
import java.util.List;
@Mapper
public interface DeptDao {
    boolean addDept(Dept dept);
    Dept findBId(Long id);
    List<Dept> findAll();
}
```

6. 根据application.yml中的位置创建mapper映射文件
```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="cn.glh.dao.DeptDao">
    <select id="findById" resultType="Dept" parameterType="Long">
        SELECT
          `deptno`,
          `dname`,
          `db_source`
        FROM
          `clouddb01`.`dept` WHERE deptno=#{deptno}
    </select>
    <select id="findAll" resultType="Dept">
        SELECT
          `deptno`,
          `dname`,
          `db_source`
        FROM
          `clouddb01`.`dept`
    </select>
    <insert id="addDept" parameterType="Dept">
        insert into dept(dname,db_source) values(#{dname},DATABASE())
    </insert>
</mapper>
```
7. 创建service层
```java
package cn.glh.service;
import cn.glh.entityes.Dept;
import java.util.List;
public interface DeptService {
    boolean add(Dept dept);
    Dept get(Long id);
    List<Dept> list();
}
```
8. 创建service层的实现类
```java
package cn.glh.service.impl;
import cn.glh.dao.DeptDao;
import cn.glh.entityes.Dept;
import cn.glh.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import java.util.List;
public class DeptServiceImpl implements DeptService {
    @Autowired
    private DeptDao dao;
    @Override
    public boolean add(Dept dept) {
        return dao.addDept(dept);
    }
    @Override
    public Dept get(Long id) {
        return dao.findById(id);
    }
    @Override
    public List<Dept> list() {
        return dao.findAll();
    }
}
```
9. 创建controller类
```java
package cn.glh.controller;

import cn.glh.entityes.Dept;
import cn.glh.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;

    @RequestMapping(value = "/dept/add",method = RequestMethod.POST)
    public boolean add(@RequestBody Dept dept){
        return deptService.add(dept);
    }

    @RequestMapping(value = "/dept/get/{id}",method = RequestMethod.GET)
    public Dept get(@PathVariable("id") Long id){
        return deptService.get(id);
    }
    
    @RequestMapping(value = "/dept/list",method = RequestMethod.GET)
    public List<Dept> list(){
        return deptService.list();
    }
}

```
10. 创建启动类
```java
package cn.glh;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class DeptProvider8001_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_App.class,args);
    }
}
```
### 4. 在父工程中创建子工程 Model =>Maven(microservicecloud-consumer-dept-80)
1. 修改pom.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring_cloud_demo</artifactId>
        <groupId>cn.glh</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>microservicecloud-consumer-dept-80</artifactId>
    <description>部门微服务消费者</description>
<dependencies>
    <dependency>
        <groupId>cn.glh</groupId>
        <artifactId>microservicecloud-api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--    热部署-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>springloaded</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
</dependencies>
</project>
```
2. 添加spring核心配置文件(application.yml)
```yml
server:
  port: 80
```
3. 创建(cfgbeans)包并创建(ConfigBean)类
```java
package cn.glh.cfgbeans;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
@Configuration
public class ConfigBean {
    @Bean
    public RestTemplate getRestTemlate(){
        return new RestTemplate();
    }
}
```
4. 创建controller(DeptController_Consumer)
```java
package cn.glh.controller;
import cn.glh.entityes.Dept;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import java.util.List;
@RestController
public class DeptController_Consumer {
    private static final String REST_URL_PREFIX="http://localhost:8001";
    @Autowired
    private RestTemplate restTemplate;
    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept){
        return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
    }
    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id,Dept.class);
    }
    @SuppressWarnings("unchecked")//压制警告
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list(Dept dept){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list",List.class);
    }
}
```
5. 创建启动类
```java
package cn.glh;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class,args);
    }
}
```
## Eureka:服务注册
### 1. 在父工程中创建子工程 Model =>Maven(microservicecloud-eureka-7001):服务注册
1. 修改pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring_cloud_demo</artifactId>
        <groupId>cn.glh</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>microservicecloud-eureka-7001</artifactId>
<dependencies>
<!--    eureka-server服务端-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
    <!--    热部署-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>springloaded</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
</dependencies>
</project>
```
2. 创建spring核心配置文件(application.yml)
```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
```
3. 创建启动类
```java
package cn.glh;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
@SpringBootApplication
@EnableEurekaServer//EurekaServer服务器端启动类,接受其他微服务注册进来
public class EurekaServer7001_App {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001_App.class,args);
    }
}
```
4. 测试

    启动启动类运行:`localhost:7001`,出现界面说明启动成功

### 2. 将已有服务注册进服务注册中心(修改microservicecloud-provider-dept-8001)
1. 修改pom.xml
```xml
<!--    将微服务provider侧注册进Eureka-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
```
2. 修改application.yml
```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://localhost:7001/eureka
```
3. 修改启动类
```java
//类上面添加
@EnableEurekaClient
```
4. 测试

    先启动Eureka服务端,再启动微服务端,重新进入`localhost:7001`
5. 完善
    1. 主机名称修改(修改微服务端application.yml)
        ```yml
        eureka:
            client: #客户端注册进eureka服务列表内
                service-url:
                defaultZone: http://localhost:7001/eureka
            instance:
                #自定义服务名称信息
                instance-id: microservicecloud-dept8001
        ```
    2. 修改IP提示
        ```yml
        eureka:
            client: #客户端注册进eureka服务列表内
                service-url:
                defaultZone: http://localhost:7001/eureka #自定义服务名称信息
            instance:
                instance-id: microservicecloud-dept8001
                prefer-ip-address: true #访问路径可以显示ip地址
        ```
    3. 微服务info内容详细信息
        ```xml
        <!--    actuator监控信息完善-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
        ```
        yml
        ```yml
        info:
            app.name: microservicecloud-dept8001
            company.name: www.baidu.com
            build.artifactId: ${project.artifactId}
            build.version: ${project.version}
        ```
        修改父工程pom.xml
        ```xml
            <build>
                <finalName>父工程项目名</finalName>
                <resources>
                    <resource>
                    <!-- 可以访问所有服务的src/main/resources文件夹 -->
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-resources-plugin</artifactId>
                        <configuration>
                            <delimiters>
                                <delimit>$</delimit>
                            </delimiters>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        ```
### 自我保护机制
`在自我保护模式中,Eureka Server会保护服务注册表中的信息,不再注销任何服务实例.当它收到的心跳数重新恢复到阈值以上时,该Eureka Server节点就会退出自我保护模式.它的设计理念就是宁可保留错误的服务注册信息,也不盲目的注销任何可能健康的服务实例.`

* 关闭自我保护模式(修改注册中心的yml)建议不关闭:
    ```yml
    eureka:
    server:
        enable-self-preservation: false
    ```
## Eureka服务发现
### 1. 修改微服务端(80端口和8001端口)conrtoller
```java
    @Autowired
    private DiscoveryClient client;
    @RequestMapping(value = "/dept/discovery",method = RequestMethod.GET)
    public Object discovery(){
        List<String> list = client.getServices();
        System.out.println("+++++"+list);
        List<ServiceInstance> service = client.getInstances(" microservicecloud-dept");
        for (ServiceInstance element : service) {
            System.out.println(element.getServiceId()+"\t"+element.getHost()+"\t"+element.getPort()+"\t"+element.getUri());
        }
        return this.client;
    }
```
### 2. 修改启动类
```java
//添加注解
@EnableDiscoveryClient
```
## Eureka集群搭建
### 1. 在父工程中创建子工程 Model =>Maven(microservicecloud-eureka-7002和microservicecloud-eureka-7003)
1. 复制microservicecloud-eureka-7002的pom文件内容到microservicecloud-eureka-7002和microservicecloud-eureka-7003
2. 复制主程序(如上)
3. 修改电脑host文件
```txt
路径:C:\Windows\System32\drivers\etc
文件:host
127.0.0.1 eureka7001.com
127.0.0.2 eureka7002.com
127.0.0.3 eureka7003.com
```
4. 修改yml
    1. 7001
        ```yml
        server:
            port: 7001

        eureka:
            server:
                enable-self-preservation: false
            instance:
                hostname: eureka7001.coom #eureka服务端的实例名称
            client:
                register-with-eureka: false #false表示不向注册中心注册自己
                fetch-registry: false #false表示自己就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
                service-url:
                ##单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
                defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
        ```
    2. 7002
        ```yml
        server:
            port: 7002
        eureka:
            server:
                enable-self-preservation: false
            instance:
                hostname: eureka7002.coom #eureka服务端的实例名称
            client:
                register-with-eureka: false #false表示不向注册中心注册自己
                fetch-registry: false #false表示自己就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
                service-url:
                ##单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
                defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/
        ```
    3. 7003
        ```yml
        server:
            port: 7003
        eureka:
            server:
                enable-self-preservation: false
            instance:
                hostname: eureka7003.coom #eureka服务端的实例名称
            client:
                register-with-eureka: false #false表示不向注册中心注册自己
                fetch-registry: false #false表示自己就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
                service-url:
                ##单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
                defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
        ```
5. 修改8001端口yml文件
    ```yml
    eureka:
        client: #客户端注册进eureka服务列表内
            service-url:
            defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
        instance:
            instance-id: microservicecloud-dept8001
            prefer-ip-address: true
    ```
6. 分别启动7001,7002,7003,8001  
浏览器访问:`http://eureka7001.com:7001/`

## Ribbon负载均衡
### 1. 80端口
修改pom文件
```xml
<!--    Ribbon负载均衡-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
```
application.yml
```yml
server:
  port: 80
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/http://eureka7002.com:7002/eureka/
```
configBean.java
```java
package cn.glh.cfgbeans;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
@Configuration
public class ConfigBean {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemlate(){
        return new RestTemplate();
    }
}
```
启动类
```java
package cn.glh;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
@SpringBootApplication
@EnableEurekaClient
public class DeptConsumer80_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class,args);
    }
}
```
controller
```java
    //客户端名称:MICROSERVICECLOUD-DEPT
    private static final String REST_URL_PREFIX="http://MICROSERVICECLOUD-DEPT";
```
### 2. 增加微服务提供者
1. 分别创建8002，8003服务提供端
2. 复制8001的pom文件到8002,8003
3. 复制8001的controller，dao，service，启动类到8002,8003
4. 复制配置文件
5. 复制数据库并改名
6. 启动7001,7002,7003,8001,8002,8003,80
7. 访问80端口的数据
### 负载均衡算法
* `RoundRobinRule` : 轮询 （排好队一个一个来）
* `RandomRule` : 随机
* `AvailabilityFilteringRule` : 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,还有并发的连接数量超过阈值的服务,然后对剩下的服务列表按照轮询策略进行访问
* `WeightedResponseTimeRule` : 根据平均响应时间计算所有的服务的权重,响应时间越快服务权重越大被选中的概率越高,刚启动时如果统计信息不足,则使用RoundRobinRule策略,等统计信息足够时会切换到WeightedResponseTimeRule
* `RetryRule` : 先按照RoundRobinRule的策略获取服务,如果获取服务失败则在指定时间内会进行重试,获取可用的服务
* `BestAvailableRule` : 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务
* `ZoneAvoidanceRule` : 默认规则,复合判断server所在区域的性能和server的可用性选择服务器
### 修改负载均衡算法
1. 修改80的ConfigBean类
```java
//增加,随机分配算法
    @Bean
    public IRule myRule(){
        return new RoundRobinRule();
    }
```
### 自定义负载均衡算法
1. 新建配置类  
`这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下,否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享,也就是说我们达不到特殊化定制的目的了`
```java
package cn.myrule;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RoundRobinRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule(){
        return new RandomRule_GLH();
    }
}
```
自定义算法
```java
package cn.myrule;
import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;
import java.util.List;
public class RandomRule_GLH extends AbstractLoadBalancerRule {
    private int total = 0;
    private int currentIndex = 0;
    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {
    }
    public Server choose(Object key) {
        ILoadBalancer loadBalancer = getLoadBalancer();
        if (loadBalancer == null)return null;
        Server server =null;
        while(server ==null){
            if (Thread.interrupted()){//线程中断
                return null;
            }
            List<Server> uplist = loadBalancer.getReachableServers();
            List<Server> allList = loadBalancer.getAllServers();
            int serverCount = allList.size();
            if (serverCount == 0){
                return null;
            }
            if (total<5){
                server = uplist.get(currentIndex);
                total++;
            }else {
                total = 0;
                currentIndex++;
                if (currentIndex>=uplist.size()){
                    currentIndex = 0;
                }
            }
        }
        return server;
    }
}
```

## Feign负载均衡
1. 创建新的客户端(microservicecloud-consumer-dept-feign)
2. 复制80端口文件
3. 修改启动类(DeptConsumer80_Feign_App)
4. 修改pom文件
```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
```
5. 修改microservicecloud-api工程  
pom
```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
```
创建接口
```java
package cn.glh.service;
import cn.glh.entityes.Dept;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import java.util.List;
@FeignClient(value = "MICROSERVICECLOUD-DEPT")
public interface DeptClientService {
    @RequestMapping(value = "/dept/get/{id}",method = RequestMethod.GET)
    public Dept get(long id);
    @RequestMapping(value = "/dept/list",method = RequestMethod.GET)
    List<Dept> list();
    @RequestMapping(value = "/dept/add",method = RequestMethod.POST)
    boolean add(Dept dept);
}
```
修改controller
```java
package cn.glh.controller;

import cn.glh.entityes.Dept;
import cn.glh.service.DeptClientService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.util.List;

@RestController
public class DeptController_Consumer {

    @Autowired
    private DeptClientService service;

    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept){
        return this.service.add(dept);
    }

    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id){
        return this.service.get(id);
    }

    @SuppressWarnings("unchecked")//压制警告
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list(){
        return this.service.list();
    }
}

```
修改启动类
```java
//添加
@EnableFeignClients(basePackages = {"cn.glh"})
```
## Hystrix 断路器
`Hystrix是一个用于处理分布式系统的延迟和容错的开源库,在分布式系统里.许多依赖不可避免的会调用失败,比如超时,异常等,Hystrix能够保证在一个依赖出问题的情况下,不会导致整个服务失败,避免级联故障,以提高分布式系统的弹性.`  
`"断路器"本身是一种开关装置,当某个服务单元发生故障之后,通过断路器的故障监控(类似熔断保险丝),向调用方返回一个符合预期的,可处理的备选响应(FallBack),而不是长时间的等待或者抛出调用方无法处理的异常,这样就保证了服务调用方的线程不会被长时间,不必要的占用,从而避免了故障在分布式系统中的蔓延,乃至雪崩`
### 熔断
### 1. 创建新的model(microservicecloud-provider-dept-hystrix-8001)
### 2. 修改pom文件
```xml
        <!--        hystrix熔断-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
```
### 3. 修改yml
```yml
    instance-id: microservicecloud-dept8001-Hystrix
```
### 4. 修改DeptController
```java
package cn.glh.controller;
import cn.glh.entityes.Dept;
import cn.glh.service.DeptService;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;
    @RequestMapping(value = "/dept/get/{id}",method = RequestMethod.GET)
    //一旦调用服务方法失败并抛出了错误信息后,会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法
    @HystrixCommand(fallbackMethod = "processHystrix_Get")
    public Dept get(@PathVariable("id") Long id){
        Dept dept = deptService.get(id);
        if (null == dept){
            throw new RuntimeException("该id:"+id+"没有对应的信息");
        }
        return dept;
    }
    public Dept processHystrix_Get(@PathVariable("id") Long id){
        return new Dept()
                .setDeptno(id)
                .setDname("该id:"+id+"没有对应的信息,null---@HystrixCommand")
                .setDb_source("no this database in mysql");
    }
}
```
修改启动类
```java
//增加注解
@EnableCircuitBreaker//对Hystrix熔断机制的支持
```
### 服务降级
1. 在microservicecloud-api中新建类(DeptClientServiceFallbackFactory)
```java
package cn.glh.service;
import cn.glh.entityes.Dept;
import feign.hystrix.FallbackFactory;
import java.util.List;
@Component
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept get(long id) {
                return new Dept().setDeptno(id).setDb_source("no this database in Mysql").setDname("该id:"+id+"没有对应的信息");
            }
            @Override
            public List<Dept> list() {
                return null;
            }
            @Override
            public boolean add(Dept dept) {
                return false;
            }
        };
    }
}
```
2. 修改(DeptClientService)
```java
//修改注解
@FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory = DeptClientServiceFallbackFactory.class)
```
3. 修改(microservicecloud-consumer-dept-feign)
```yml
//修改yml文件
feign:
  hystrix:
    enabled: true
```
4. 启动测试
启动7001,7002,7003,8001,80(feign)

### 服务监控
1. 新建model(microservicecloud-consumer-hystrix-dashboard)
2. 修改pom
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring_cloud_demo</artifactId>
        <groupId>cn.glh</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-consumer-hystrix-dashboard</artifactId>

<dependencies>
    <dependency>
        <groupId>cn.glh</groupId>
        <artifactId>microservicecloud-api</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
    </dependency>
</dependencies>
</project>
```
3. 创建启动类
```java
package cn.glh;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@EnableHystrixDashboard
@SpringBootApplication
public class DeptConsumer_DashBoard_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_DashBoard_App.class,args);
    }
}

```
4. 创建yml
```yml
server:
  port: 9001
```

## zuul路由网关
Zuul包含了队请求的路由和过滤两个主要的功能:

其中路由功能负责将外部请求转发到具体的微服务实例上,是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预,是实现请求校验,服务聚合等功能的基础.zuul和eureka进行整合,将zuul自身注册为eureka服务治理下的应用,同时从eureka中获得其他微服务的消息,也即以后的访问微服务都是通过zuul跳转后获得.

注意 : zuul服务最终还是会注册进eureka
### 新建model模块(microservicecloud-zuul-gateway-9527)
1. 修改pom文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring_cloud_demo</artifactId>
        <groupId>cn.glh</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-zuul-gateway-9527</artifactId>

    <dependencies>
        <dependency>
            <groupId>cn.glh</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```
2. 修改yml
```yml
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true
info:
  app.name: zeluo-microcloud
  company.name: www.baidu.com
    build.artifactId: ${project.artifactId}
    build.version: ${project.version}
server:
  port: 9527
spring:
  application:
    name: microservicecloud-zuul-gateway
```
3. 新建启动类
```java
package cn.glh;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
public class Zuul_9527_StartSpringCloudApp {
    public static void main(String[] args) {
        SpringApplication.run(Zuul_9527_StartSpringCloudApp.class,args);
    }
}
```
4. 测试
启动7001,7002,7003,8001,9527,访问http://localhost:9527/microservicecloud-dept/dept/get/2
### 路由访问映射规则
1. 修改yml
```yml
增加
zuul:
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```
2. 单入口,之前的路径忽略(修改yml)
```yml
zuul:
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
  ignored-services: microservicecloud-dept #忽略microservicecloud-dept访问路径
  ignored-services: "*"
```
3. 增加访问前缀(yml)
```yml
zuul:
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
  ignored-services: microservicecloud-dept
  prefix: /glh #前缀
```
## SpringCloud Config(分布式配置中心)
1. 登陆github创建仓库(microservicecloud-config)
2. 本地创建文件夹并初始化本地仓库(git clone 仓库连接)
3. 本地文件夹创建文件(application.yml)并保存为utf-8
4. yml
```yml
spring:
  profiles:
    active: -dev
---
spring:
  profiles: dev
  application:
    name: microservicecloud-config-glh-dev
---
spring:
  profiles: test
  application:
    name: microservicecloud-config-glh-test
```
5. 本地yml推送到远程仓库
6. 新建model(microservicecloud-config-3344)
7. 添加yml
```yml
server:
  port: 3344
spring:
  application:
    name: microservicecloud-config
  cloud:
    config:
      server:
        git:
          uri: git@github.com:GLH1218/microservicecloud-config.git
```
8. 添加pom
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring_cloud_demo</artifactId>
        <groupId>cn.glh</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>microservicecloud-config-3344</artifactId>


    <dependencies>
        <dependency>
            <groupId>cn.glh</groupId>
            <artifactId>microservicecloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config</artifactId>
        </dependency>
    </dependencies>
</project>
```