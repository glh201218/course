# 搭建环境
## 导入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.glh</groupId>
    <artifactId>jpa</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
<!--        测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    <!--   hibernate对jpa的支持包 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.0.7.Final</version>
        </dependency>
<!--        c3p0,数据库连接池-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-c3p0</artifactId>
            <version>5.0.7.Final</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>
        
    </dependencies>
</project>
```
## 配置文件
`在类路径下创建名为META-INF的文件夹,在文件夹中创建名为persistence.xml的文件`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
<!--需要配置persistence-unit节点
    持久化单元
        name:持久化单元名称
        transaction-type:事务管理的方式
            JTA:分布式事务管理
            RESOURCE_LOCAL:本地事务管理
-->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
<!--        jpa的实现方式-->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
<!--        数据库信息-->
        <properties>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="1218"/>
            <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/jpa?characterEncoding=utf8"/>
            <!--        可选配置:配置jpa实现方的配置信息
                显示sql:hibernate.show_sql    true|false
                自动创建数据库表:hibernate.hbm2ddl.auto
                    create:程序运行时创建数据库表(如果有表,先删除表再创建)
                    update:程序运行时创建表(如果有表,不会创建表)
                    none:不会创建表
            -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="create"/>
        </properties>
    </persistence-unit>
</persistence>
```
## 编写实体类
```java
package cn.glh.entity;

import javax.persistence.*;

/**
 * 客户的实体
 *  配置映射关系
 *      1.实体类和表的映射关系
 *  @Entity:声明实体类
 *  @Table:配置实体类和表的映射关系
 *      name:表名
 * 2.实体类中属性和表中字段的映射关系
 *
 */
@Entity
@Table(name="cst_customer")
public class Customer {
    /**
     * @Id:声明主键的配置
     * @GeneratedValue:主键的生成策略
     *  strategy
     *      GenerationType.IDENTITY:自增
     *          *底层数据库必须支持自动增长
     *      GenerationType.SEQUENCE:序列
     *          *底层数据库必须支持序列
     *      GenerationType.TABLE:jpa提供的一种机制,通过一张数据库表的形式帮助我们完成主键自增
     *      GenerationType.AUTO:由程序自动的帮助我们选择主键生成策略
     * @Column:属性和字段的映射关系
     *  name:数据库表中字段的名称
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId;//客户的主键
    @Column(name = "cust_name")
    private String custName;//客户名称
    @Column(name = "cust_source")
    private String custSource;//客户来源
    @Column(name = "cust_level")
    private String custLevel;//客户级别
    @Column(name = "cust_industry")
    private String custIndustry;//客户所属行业
    @Column(name = "cust_phone")
    private String custPhone;//客户联系方式
    @Column(name = "cust_address")
    private String custAddress;//客户地址

    public Long getCustId() {
        return custId;
    }

    public void setCustId(Long custId) {
        this.custId = custId;
    }

    public String getCustName() {
        return custName;
    }

    public void setCustName(String custName) {
        this.custName = custName;
    }

    public String getCustSource() {
        return custSource;
    }

    public void setCustSource(String custSource) {
        this.custSource = custSource;
    }

    public String getCustLevel() {
        return custLevel;
    }

    public void setCustLevel(String custLevel) {
        this.custLevel = custLevel;
    }

    public String getCustIndustry() {
        return custIndustry;
    }

    public void setCustIndustry(String custIndustry) {
        this.custIndustry = custIndustry;
    }

    public String getCustPhone() {
        return custPhone;
    }

    public void setCustPhone(String custPhone) {
        this.custPhone = custPhone;
    }

    public String getCustAddress() {
        return custAddress;
    }

    public void setCustAddress(String custAddress) {
        this.custAddress = custAddress;
    }

    @Override
    public String toString() {
        return "Customer{" +
                "custId=" + custId +
                ", custName='" + custName + '\'' +
                ", custSource='" + custSource + '\'' +
                ", custLevel='" + custLevel + '\'' +
                ", custIndustry='" + custIndustry + '\'' +
                ", custPhone='" + custPhone + '\'' +
                ", custAddress='" + custAddress + '\'' +
                '}';
    }
}

```
# 测试
```java
import cn.glh.entity.Customer;
import org.junit.Test;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
public class JpaTest {
    /**
     *测试jpa的保存
     *  案例:保存一个客户到数据库中
     *  jpa的操作步骤
     *      1.加载配置文件创建工厂(实体管理类工厂)对象
     *      2.通过实体管理类工厂获取实体管理器
     *      3.获取事务对象,开启事务
     *      4.完成增删改查操作
     *      5.提交事务(回滚事务)
     *      6.释放资源
     */
    @Test
    public void testSave(){
        //1.加载配置文件创建工厂(实体管理类工厂)对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        //2.通过实体管理类工厂获取实体管理器
        EntityManager entityManager = factory.createEntityManager();
        //3.获取事务对象,开启事务
        EntityTransaction transaction = entityManager.getTransaction();
        //开启事务
        transaction.begin();
        //4.完成增删改查操作
        Customer customer=new Customer();
        customer.setCustName("glh");
        customer.setCustIndustry("哈哈哈");
        entityManager.persist(customer);
        //5.提交事务(回滚事务)
        transaction.commit();
        //6.释放资源
        entityManager.close();
        factory.close();
    }
}
```
# 优化
## 创建工具类
```java
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaUtils {
    private static EntityManagerFactory factory;
    static {
        //1.加载配置文件,创建entityManagerFactory
        factory= Persistence.createEntityManagerFactory("myJpa");
    }
    public static EntityManager getEntityManager(){
        return factory.createEntityManager();
    }
}
```
## 测试
### 添加
```java

import cn.glh.entity.Customer;
import cn.glh.utils.JpaUtils;
import org.junit.Test;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaTest {
    /**
     *测试jpa的保存
     *  案例:保存一个客户到数据库中
     *  jpa的操作步骤
     *      1.加载配置文件创建工厂(实体管理类工厂)对象
     *      2.通过实体管理类工厂获取实体管理器
     *      3.获取事务对象,开启事务
     *      4.完成增删改查操作
     *      5.提交事务(回滚事务)
     *      6.释放资源
     */
    @Test
    public void testSave(){
        //工具类获取实体管理器
        EntityManager entityManager= JpaUtils.getEntityManager();
        //3.获取事务对象,开启事务
        EntityTransaction transaction = entityManager.getTransaction();
        //开启事务
        transaction.begin();
        //4.完成增删改查操作
        Customer customer=new Customer();
        customer.setCustName("glh");
        customer.setCustIndustry("哈哈哈");
        entityManager.persist(customer);
        //5.提交事务(回滚事务)
        transaction.commit();
        //6.释放资源
        entityManager.close();
    }
}
```
### 查找
#### find
```java
    @Test
    public void testFind(){
        //1.通过工具类获取entityManager
        EntityManager entityManager = JpaUtils.getEntityManager();
        //2.开启事务
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();
        //3.增删改查
        /**
         * find:根据id查询数据
         *  class:根据数据的结果需要包装的实体类型的字节码
         *  id:查询的主键的取值
         */
        Customer customer = entityManager.find(Customer.class, 1);
        System.out.println(customer);
        //4.提交事务
        transaction.commit();
        //5.释放资源
        entityManager.close();
    }
```
#### getReference
