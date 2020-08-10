# Spring 
    框架 : 高度抽取可重用代码的一种设置;高度的通用性
## 概述
1. Spring是一个开源框架
2. spring为简化企业级开发而生,使用spring,javabean就可以实现很多以前要靠EJ才能实现的功能.同样的功能,在EJB中要通过繁琐的配置和复杂的代码才能够实现.而spring中却非常的优雅和简洁.
3. spring是一个IOC(DI)和AOP容器框架
4. spring的优良特性
    1. 非侵入式: 基于spring开发的应用中的对象可以不依赖于spring的API
    2. 依赖注入: DI(Dependency Injection),反转控制(IOC)最经典的实现.
    3. 面向切面编程: Aspect Oriented Programming(APO)
    4. 容器: spring是一个容器,因为它包含并且管理应用对象的声明周期
    5. 组件化: spring实现了使用简单的组件配置组合成一个复杂的应用,在spring中可以使用XML和java注解组合这些对象
    6. 一站式: 在IOC和AOP的基础上可以整合各种企业应用的开源框架和优秀的第三方类库(实际上spring自身也提供了表述层的springMVC和持久层的spring JDBC)
## IOC
IOC: (Inversion(反转)Of Control(控制)):控制反转  
### 控制: 获取资源的方式:
* 主动式:(要什么资源自己创建)
```java
    BOOkServlet{
        BOOkServlet bs = new BOOkServlet();//可以
        AirPlane ap = new AirPlane();//复杂对象的创建是比较庞大的工程
    }
```
* 被动式:资源的获取不是我们自己创建的,而是交给一个容器来创建和设置
```java
BOOkServlet{
    BOOkService bs;
    public void test01(){
        bs.checkout();
    }
}
```
### 容器: 管理所有的组件(有功能的类)
    假设,BookServlet受容器管理,BookService也受容器管理,容器可以自动的探查出那些类需要用到另一些类,容器帮我们创建爱你BookService对象,并把BookService对象赋值进去

    主动的new资源变成被动的接受资源

### DI: (Dependency Injection)依赖注入
    容器能知道那个类运行的时候,需要另一个类,容器通过反射的形式,将容器中准备好的BookService对象注入(利用反射给属性赋值)到BookService中.

## HelloWord
### 配置文件
1. 导入依赖
```xml
     <!-- 添加Spring核心依赖 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>3.1.1.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>3.1.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>3.1.1.RELEASE</version>
    </dependency>
<!--    日志-->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
<!--测试工具-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
    </dependency>
```
2. 创建配置文件(ioc.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
     http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
<!--注册一个Person对象,spring会自动创建这个Person对象-->
<!--
    一个bean标签可以注册一个组件
    class:要注册的组件的全类名
    id:这个对象的唯一标识
-->
    <bean class="cn.glh.bean.Person" id="person01" >
    <!--
        使用property标签为Person对象的属性赋值
        name:属性名
        value:要赋的值
    -->
        <property name="lastName" value="张三"></property>
        <property name="age" value="18"></property>
        <property name="email" value="2028304680@qq.com"></property>
        <property name="gender" value="女"></property>
    </bean>
</beans>
```
3. 创建bean
```java
package cn.glh.bean;
public class Person {
    private String lastName;
    private Integer age;
    private String gender;
    private String email;
    public Person(String lastName, Integer age, String gender, String email) {
        this.lastName = lastName;
        this.age = age;
        this.gender = gender;
        this.email = email;
    }
    public Person() {
    }
    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", gender='" + gender + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
    public String getLastName() {
        return lastName;
    }
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
    public String getGender() {
        return gender;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
        this.email = email;
    }
}
```
4. 创建测试类:

    * (一)通过id获取bean的实例
    ```java
    package cn.glh.test;
    import cn.glh.bean.Person;
    import org.junit.Test;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import org.springframework.context.support.FileSystemXmlApplicationContext;

    public class IOCTest {
        /**
        * 从容器中拿到这个组件
        */
        @Test
        public void test(){
            //ApplicationContext: 代表ioc容器

            //ClassPathXmlApplicationContext: 当前应用的xml配置文件在ClassPath下
            //ApplicationContext ioc = new ClassPathXmlApplicationContext("classpath:ioc.xml");

            //FileSystemXmlApplicationContext:文件的本地路径
            ApplicationContext ioc = new FileSystemXmlApplicationContext("G:\\mySpring\\src\\main\\resources\\ioc.xml");

            //容器帮我们创建好对象了
            Person person01 =(Person)ioc.getBean("person01");//通过容器内对象id获取对象
            System.out.println(person01.toString());
        }
    }
    ```
    容器中的对象在创建容器完成的时候已经创建完成了
    同一个组件在ioc容器中是单实例的
    ioc容器在创建这个组件对象的时候,(property)会利用setter方法为javaBean的属性进行赋值
    javaBean的属性名是由getter/setter方法名决定的(set去掉,后面的一串字符小写就是属性名)
    * (二)根据bean的类型从IOC容器中获取bean的实例
    ```java
    //这种方法容器中只能定义一个这个类型的实例,如果有多个会报错
    @Test
    public void test02(){
        Person bean = ioc.getBean(Person.class);
        System.out.println(bean);
    }
    //这种方法就不会出现上面的问题通过di和类型查找
    @Test
    public void test03(){
        Person bean = ioc.getBean("person02",Person.class);
        System.out.println(bean);
    }
    ```
    (三)通过构造器为bean的属性赋值(index,type属性介绍),通过`p`命名空间为bean赋值
    ```xml
    <!--        调用有参构造器进行创建对象并赋值-->
    <!--
        构造器中有多少个参数就写几个constructor-arg标签
        属性:
            name: 构造器属性名
            value: 要赋的值
    -->
            <constructor-arg name="lastName" value="小明"></constructor-arg>
            <constructor-arg name="age" value="18"></constructor-arg>
            <constructor-arg name="email" value="2028304680@qq.com"></constructor-arg>
            <constructor-arg name="gender" value="男"></constructor-arg>
        </bean>
        <!--
            构造器中有多少个参数就写几个constructor-arg标签
            属性:
                value: 要赋的值
            如果没有name属性value就要按照构造器的参数顺序赋值
        -->
        <bean id="person04" class="cn.glh.bean.Person">
            <constructor-arg value="小王"></constructor-arg>
            <constructor-arg value="19"></constructor-arg>
            <constructor-arg value="男"></constructor-arg>
            <constructor-arg value="202@qq.com"></constructor-arg>
        </bean>

        <!--
        构造器中有多少个参数就写几个constructor-arg标签
        属性:
            value: 要赋的值
        如果没有name属性value就要按照构造器的参数顺序赋值
        不按照顺序的话可以添加index属性表示这个值赋值到第几个参数(从0开始)
    -->
        <bean id="person05" class="cn.glh.bean.Person">
            <constructor-arg value="小王"></constructor-arg>
            <constructor-arg value="19"></constructor-arg>
            <constructor-arg value="202@qq.com" index="3"></constructor-arg>
            <constructor-arg value="男" index="2"></constructor-arg>
        </bean>

        <!--
            如果有多个相同数量参数的构造器的时候就需要用到type参数来指定参数的类型
        -->
        <bean id="person06" class="cn.glh.bean.Person">
            <constructor-arg value="小王"></constructor-arg>
            <constructor-arg value="19" type="java.lang.String"></constructor-arg>
            <constructor-arg value="男" type="java.lang.String"></constructor-arg>
        </bean>
    ```
    p命名空间赋值
    * 导入命名空间
    ```xml
    <bean id="person07" class="cn.glh.bean.Person" p:age="18" p:email="17600250974@163.com" p:lastName="哈哈"></bean>
    ```
(四)复杂类型属性赋值
1. 创建Car类
```java
package cn.glh.bean;
public class Car {
    private String carName;
    private Integer price;
    private String color;
    public Car() {
    }
    public Car(String carName, Integer price, String color) {
        this.carName = carName;
        this.price = price;
        this.color = color;
    }
    public String getCarName() {
        return carName;
    }
    public void setCarName(String carName) {
        this.carName = carName;
    }
    public Integer getPrice() {
        return price;
    }
    public void setPrice(Integer price) {
        this.price = price;
    }
    public String getColor() {
        return color;
    }
    public void setColor(String color) {
        this.color = color;
    }
    @Override
    public String toString() {
        return "Car{" +
                "carName='" + carName + '\'' +
                ", price=" + price +
                ", color='" + color + '\'' +
                '}';
    }
}
```
2. 创建Book类
```java
package cn.glh.bean;
public class Book {
    private String bookName;
    private String author;
    public Book(String bookName, String author) {
        this.bookName = bookName;
        this.author = author;
    }
    public Book() {
    }
    @Override
    public String toString() {
        return "Book{" +
                "bookName='" + bookName + '\'' +
                ", author='" + author + '\'' +
                '}';
    }
    public String getBookName() {
        return bookName;
    }
    public void setBookName(String bookName) {
        this.bookName = bookName;
    }
    public String getAuthor() {
        return author;
    }
    public void setAuthor(String author) {
        this.author = author;
    }
}
```
3. 进行赋值
    * null赋值
    ```xml
        <bean id="person01" class="cn.glh.bean.Person">
    <!--    value="null" 这种是不行的,要进行复杂的赋值需要在property标签内部赋值-->
        <property name="lastName">
            <null></null>
        </property>
    </bean>
    ```
    * 对象赋值1
    ```xml
    <bean name="car01" class="cn.glh.bean.Car">
        <property name="carName" value="兰博"></property>
        <property name="color" value="金色"></property>
        <property name="price" value="1"></property>
    </bean>
    <bean id="person01" class="cn.glh.bean.Person">
    <!--    为car赋值
    方法1:引用外部部bean-->
        <property name="car" ref="car01"></property>
    </bean>
    ```
    * 对象赋值2
    ```xml
    <bean id="person01" class="cn.glh.bean.Person">
    <!--    为car赋值
    方法2:使用内部bean-->

    <!-- 内部bean是不能被获取到的,只能内部使用 -->
    <property name="car">
        <bean class="cn.glh.bean.Car">
            <property name="price" value="20"></property>
            <property name="color" value="黄色"></property>
            <property name="carName" value="自行车"></property>
        </bean>
    </property>
    </bean>
    ```
    * List赋值
    ```xml
    <!--    List类型赋值-->
        <bean class="cn.glh.bean.Person" id="person02">
            <property name="books">
            <!-- 相当于List list = new List<Book> -->
                <list>
                <!-- 内部bean -->
                    <bean id="book01" class="cn.glh.bean.Book" p:bookName="西游记"></bean>
                    <!-- 引用外部 -->
                    <ref bean="book02"></ref>
                </list>
            </property>
        </bean>
        <bean id="book02" class="cn.glh.bean.Book">
            <property name="bookName" value="东游记"></property>
        </bean>
    ```
    * map赋值
    ```xml
    <bean class="cn.glh.bean.Person" id="person02">
        <property name="maps">
            <map>
                <entry key="key01" value="张三"></entry>
                <entry key="key02" value="18"></entry>
                <!-- 引用外部bean-->
                <entry key="key03" value-ref="book02"></entry>
                <!-- 使用内部bean-->
                <entry key="key04">
                    <bean id="book" class="cn.glh.bean.Book">
                        <property name="bookName" value="童话历险记"></property>
                    </bean>
                </entry>
                <!-- 值是一个map-->
                <entry key="key05">
                    <map>
                        <entry key="key01" value-ref="book"></entry>
                    </map>
                </entry>
            </map>
        </property>
    </bean>
    <bean id="book02" class="cn.glh.bean.Book">
        <property name="bookName" value="东游记"></property>
    </bean>
    ```
    * properties赋值
    ```xml
    <bean class="cn.glh.bean.Person" id="person02">
        <property name="properties">
            <!--new Properties();-->
            <!--key和value都是string类型-->
            <props>
                <!--值可以直接写在标签体内-->
                <prop key="username">root</prop>
                <prop key="password">123456</prop>
            </props>
        </property>
    </bean>
    ```
    * 级联赋值
    ```xml
    <bean name="car01" class="cn.glh.bean.Car">
        <property name="carName" value="兰博"></property>
        <property name="color" value="金色"></property>
        <property name="price" value="1"></property>
    </bean>
    <!--    级联属性赋值,属性的属性-->
        <bean id="person" class="cn.glh.bean.Person">
    <!--        为car赋值的时候改变car的价格,因为是引用所以这里改了的话别的地方也会改-->
            <property name="car" ref="car01"></property>
            <property name="car.price" value="30"></property>
        </bean>
    ```
* util名称空间创建集合类型的bean,方便别人引用
    ```xml
    <bean class="cn.glh.bean.Person" id="person03">
        <property name="maps" ref="mymap"></property>
    </bean>
    <!--相当于new LinkedHashMap<>()-->
    <util:map id="mymap">
        <entry key="key01" value="回个"></entry>
    </util:map>
    ```
4. 通过继承实现bean配置的重用
```xml
 <bean class="cn.glh.bean.Person" id="person05">
        <property name="lastName" value="张三"></property>
        <property name="age" value="18"></property>
        <property name="gender" value="男"></property>
        <property name="email" value="1863@qq.com"></property>
    </bean>
    <!--    parent:指定当前bean的配置信息继承于哪个bean-->
    <bean class="cn.glh.bean.Person" id="person06" parent="person05">
        <!--        改变person05的信息-->
        <property name="lastName" value="李四"></property>
    </bean>
```
5. 通过abstract属性创建模板bean
```xml
    <!-- 当前bean是抽象的只能被继承,不能获取他的实例 -->
    <bean class="cn.glh.bean.Person" id="person05" abstract="true">
        <property name="lastName" value="张三"></property>
        <property name="age" value="18"></property>
        <property name="gender" value="男"></property>
        <property name="email" value="1863@qq.com"></property>
    </bean>
    <!--parent:指定当前bean的配置信息继承于哪个bean-->
    <bean class="cn.glh.bean.Person" id="person06" parent="person05">
        <!--改变person05的信息-->
        <property name="lastName" value="李四"></property>
    </bean>
```
6. bean之间的依赖
```xml
<!--    原来是按照配置的顺序创建bean-->
<!--    改版bean的创建顺序-->
    <bean class="cn.glh.bean.Person" id="person"></bean>
    <bean class="cn.glh.bean.Car" id="car" depends-on="person,book"></bean>
    <bean class="cn.glh.bean.Book" id="book"></bean>
```
7. bean的作用域
```xml

<!--    bean的作用域
        prototype:多实例
            容器启动默认不会创建多实例bean
            获取的时候创建bean
            每次获取都会创建新的实例
        singleton:单实例 默认
            在容器启动完成之前就已经创建好对象,保存在容器中了
            任何时候获取都是之前创建好的
        request:在web环境下,同一次请求创建一个bean实例
        session:在web环境下,同一次会话创建一个bean实例
        -->
    <bean class="cn.glh.bean.Book" id="book" scope="prototype"></bean>
```
8. 配置通过静态工厂方法创建bean,实例工厂方法创建bean,factorybean

    工厂模式:工厂帮我们创建对象;有一个专门帮我们创建对象的类,这个类就是工厂
    ```java
    package cn.glh.factory;
    import cn.glh.bean.AirPlane;
    /**
    * 实例工厂
    */
    public class AirPlaneInstanceFactory {
        public AirPlane getAirPlane(String jzName){
            AirPlane airPlane = new AirPlane();
            airPlane.setFdj("太行");
            airPlane.setFjsName("战三");
            airPlane.setJzName(jzName);
            airPlane.setYc("198.5m");
            airPlane.setPersonNum(176);
            return airPlane;
        }
    }
    ```
    ```java
    package cn.glh.factory;
    import cn.glh.bean.AirPlane;
    /**
    * 静态工厂
    */
    public class AirPlaneStaticFactory {
        public static AirPlane getAirPlane(String jzName){
            AirPlane airPlane = new AirPlane();
            airPlane.setFdj("太行");
            airPlane.setFjsName("战三");
            airPlane.setJzName("gaolingj");
            airPlane.setYc("198.5m");
            airPlane.setPersonNum(176);
            return airPlane;
        }
    }
    ```
    静态工厂:工厂本身不用创建对象;通过静态方法调用,对象=工厂类.工厂方法名();
    ```xml
    <!--静态工厂(不需要创建工厂本身)
    factory-method:那个方法是工厂方法-->
        <bean id="airPlane01" class="cn.glh.factory.AirPlaneStaticFactory" factory-method="getAirPlane">
            <!--为方法指定参数-->
            <constructor-arg name="jzName" value="机长"></constructor-arg>
        </bean>
    ```
    实例工厂:工厂本身需要创建对象;工厂类 工厂对象 = new 工厂类();
    
    工厂对象.getAirPlane("张三");
    ```xml
    <!--    实例工厂
        先配置实例工厂对象
        在配置我们要创建的AirPlane使用那个工厂实例
            factory-bean:指定使用那个工厂实例
            factory-method:使用那个工厂方法
    -->
    <bean id="airPlaneInstanceFactory" class="cn.glh.factory.AirPlaneInstanceFactory"></bean>
    <bean class="cn.glh.bean.AirPlane" id="airPlane02" factory-bean="airPlaneInstanceFactory" factory-method="getAirPlane">
        <constructor-arg value="王五"></constructor-arg>
    </bean>
    ```
   Factorybean是Spring规定的一个接口;只要是这个接口的实现类,Spring都认为是一个工厂
   ```java
    package cn.glh.factory;
    import cn.glh.bean.Book;
    import org.springframework.beans.factory.FactoryBean;
    import java.util.UUID;
    /**
    * 实现了FactoryBean接口的类是Spring可以认识的工厂类
    * Spring会自动的调用工厂方法创建实例
    * 1.编写一个FactoryBean的实现类
    * 2.在spring配置文件中进行注册bean
    *
    */
    public class MyFactoryBeanImple implements FactoryBean<Book> {
        /**
        * 工厂方法
        * 返回创建的对象
        * @return
        * @throws Exception
        */
        @Override
        public Book getObject() throws Exception {
            Book book = new Book();
            book.setBookName(UUID.randomUUID().toString());
            return book;
        }
        /**
        * 放回创建对象的类型
        * @return
        */
        @Override
        public Class<?> getObjectType() {

            return Book.class;
        }
        /**
        * 是否是单例
        * @return
        */
        @Override
        public boolean isSingleton() {
            return false;
        }
    }
   ```

   ```xml
    <!--    FactoryBean
        自动创建MyFactoryBeanImple类中的方法返回的类对象
        ioc容器启动的时候不会创建实例
        获取的时候才创建对象
    -->
        <bean id="myFactoryBeanImple" class="cn.glh.factory.MyFactoryBeanImple"></bean>
   ```
9. 创建带有生命周期方法的bean
```java
package cn.glh.bean;

public class Book {
    private String bookName;
    private String author;

    public Book(String bookName, String author) {
        this.bookName = bookName;
        this.author = author;
    }

    public Book() {
    }

    public void myInit(){
        System.out.println("初始化方法");
    }
    public void myDestory(){
        System.out.println("图书的销毁方法");
    }
    @Override
    public String toString() {
        return "Book{" +
                "bookName='" + bookName + '\'' +
                ", author='" + author + '\'' +
                '}';
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }
}

```

```xml
<!--
    生命周期:bean的创建到销毁
        ioc容器中注册的bean
            1. 单例bean,容器启动的时候就会创建好,容器关闭也会销毁创建的bean
            2. 多实例的bean,获取的时候才创建
        我们可以为bean自定义一些生命周期方法;spring在创建或者销毁的时候就会调用指定的方法
        Bean的声明周期:
        构造器->初始化方法->销毁方法
-->
    <bean class="cn.glh.bean.Book" id="book2" destroy-method="myDestory" init-method="myInit" ></bean>
```
10. bean的后置处理器
```java
package cn.glh.bean;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

/**
 * 1. 编写后置处理器的实现类
 * 2. 将后置处理器注册在配置文件中
 */
public class MyBeanPostProcessor implements BeanPostProcessor {
    /**
     * 初始化之前调用
     * @param bean
     * @param beanname
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanname) throws BeansException {
        System.out.println(beanname+"将要初始化,当前对象"+bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanname) throws BeansException {
        System.out.println(beanname+"初始化完成,当前对象"+bean);
        return bean;//初始化完成后的bean
    }
}
```
```xml
<!--    bean后置处理器
        可以在bean的初始化前后调用方法
-->
    <bean class="cn.glh.bean.MyBeanPostProcessor" id="beanPostProcessor"></bean>
```
11. 加载外部配置文件
```xml
 <!--加载外部配置文件 固定写法classpath:,表示引用类路径下的一个资源-->
    <context:property-placeholder location="classpath:dbconfig.properties"></context:property-placeholder>
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <!--username是spring的key中的一个关键字,为了防止配置文件中的key和spring的冲突,可以在配置文件前面加上前缀-->
        <property name="user" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
        <property name="driverClass" value="${jdbc.driverClass}"></property>
    </bean>
```
创建dbconfig.properties文件
```properties
jdbc.username=root
jdbc.password=1218
jdbc.jdbcUrl=jdbc:mysql://localhost:3306/test
jdbc.driverClass=com.mysql.jdbc.Driver
```
12. 基于xml的自动装配(自定义类型自动赋值)

```xml
<!--    基于xml的自动装配(自定义类型自动赋值)-->
    <bean id="car04" class="cn.glh.bean.Car">
        <property name="carName" value="号码"></property>
        <property name="color" value="黑色"></property>
    </bean>
<!--为person里面的自定义类型的属性赋值
    property属性是手动赋值的
    自动赋值:autowire
        参数: default: 不自动装配
              byName: 按照属性名字作为id去容器中找到这个组件;如果找不到就赋值null
              byType: 以属性的类型去容器中找到这个组件;如果容器中有多个这个类型的组件就会报错;没找到赋值null
              constructor: 以构造器赋值
                按照构造器进行赋值
                    先按照有参构造器参数的类型进行装配;没有就直接装配null
                    如果按照类型找到多个,就让参数的名作为id继续匹配;找不到就装配null
               -->
    <bean id="person03" class="cn.glh.bean.Person" autowire="constructor"></bean>
```
13. SpEl(Spring Expression Language):Spring表达式语言
```xml
    <!--
        1. 在SpEL中使用字面量
        2. 引用其他bean
        3. 引用其他bean的某个属性
        4. 调用非静态方法
        5. 调用静态方法
        6. 使用运算符
    -->
        <bean class="cn.glh.bean.Person" id="person2">
            <property name="age" value="#{100*75}"></property>
            <property name="car" value="#{car}"></property>
            <property name="lastName" value="#{book.bookName}"></property>
            <!--调用非静态方法:bean的id.方法-->
            <property name="gender" value="#{book2.getBookName()}"></property>
            <!--调用静态方法: #{T(全类名).静态方法(参数)}-->
            <property name="email" value="#{T(java.util.UUID).randomUUID().toString()}"></property>
        </bean>
```

### 注解
通过注解分别创建Dao,Service,Controller.

通过给bean上添加某些注解,可以快速的将bean加入到ioc容器中

spring的四个注解:
* @Controller : 控制器
```java
package cn.glh.servlet;
import org.springframework.stereotype.Controller;
@Controller
public class BookServlet {
}
```
* @Service : 业务逻辑
```java
package cn.glh.service;
import org.springframework.stereotype.Service;
@Service
public class BookService {
}
```
* @Repository : 持久化
```java
package cn.glh.dao;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Repository;
/**
 * 组件的id,默认就是组件的类名首字母小写
 * 可以在组件后面的小括号里写入需要的组件id
 * 组件的作用域默认是单例的
 *  可以添加@Scope(value = "prototype")变为多实例的
 */
@Scope(value = "prototype")
@Repository("bookDao01")
public class BookDao {
}

```
* @Component : 不属于上面的组件添加

加入后组要在配置文件中加入扫描注解的功能
base-package : 指定扫描的基础包,把基础包及他下面所有的包的所有加了注解的类,自动的扫描进ioc容器
```xml
    <context:component-scan base-package="cn.glh"></context:component-scan>
```
#### 测试
```java
private ApplicationContext ioc = new FileSystemXmlApplicationContext("G:\\mySpring\\src\\main\\resources\\ioc3.xml");
    /**
     * 组件的id,默认就是组件的类名首字母小写
     * 组件的作用域默认是单例的
     */
    @Test
    public void test07() {
        Object bookDao = ioc.getBean("bookDao");
        Object bookDao1 = ioc.getBean("bookDao");
        System.out.println(bookDao==bookDao1);
    }
```
* context:exclude-filter指定扫描包时不包含的类
```xml
<context:component-scan base-package="cn.glh">
    <!--context:exclude-filter指定扫描包时不包含的类
       -type:annotation:指定排除规则;按照注解进行排除.标-了指定注解的组件不要
       -    expression:注解的全类名
       -type:aspectj:aspectj表达式
       -type:assignable:指定排除某个具体的类,按照类排除
       -    expression:要排除类的全类名
       -type:custom:自动义一个TypeFilter的实现类;
       -type:regex:正则表达式
       -->
    <context:exclude-filter type="assignable" expression="cn.glh.service.BookService"/>
</context:component-scan>
```
* context:include-filter指定扫描包时要包含的类
```xml
<!--  使用context:include-filter前一定要先禁用他的默认规则  -->
    <context:component-scan base-package="cn.glh" use-default-filters="false">
        <!--context:include-filter指定扫描包时要包含的类
            type:annotation:指定规则;按照注解进行排除.标注了指定注解的组件不要
                expression:注解的全类名
            type:aspectj:aspectj表达式
            type:assignable:指定某个具体的类
                expression:要排除类的全类名
            type:custom:自动义一个TypeFilter的实现类;
            type:regex:正则表达式
        -->
        <context:include-filter type="assignable" expression="org.springframework.stereotype.Service"/>
    </context:component-scan>
```
* Autowired自动装配
```txt
    原理:先按照类型去容器中找到对应组件
    找到就赋值
        找到多个
            按照变量名继续匹配
                匹配不到可以在注入时加上注解@Qualifier("")
                指定一个名作为id,让spring别使用变量名作为id
                    如果实在是找不到就可以在@Autowired注解中写required = false他就会在找不到的时候赋值null,防止报错
    找不到就抛异常
```
```java
    /**
     * 在方法上使用@Autowired
     * 这个方法会在bean创建的时候自动执行
     * 这个方法的每一个参数都会自动注入值
     */
    @Autowired(required = false)
    public void hahah(BookDao bookDao,@Qualifier("bookService") BookService bookService){
        System.out.println(bookDao);
    }
```
* @Resource自动装配
@Autowired,@Resource,@Inject,都是自动装配

@Resource:扩展性很强,如果切换成另外一个容器框架,@Resource还是可以使用的,@Autowired就不行
### 单元测试
1. 导入spring单元测试依赖
2. 测试
```java
package cn.glh.test;
import cn.glh.servlet.BookServlet;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
/**
 * 使用Spring的单元测试
 * 1. 导包
 * 2.@ContextConfiguration(locations = ""):指定spring的配置文件的位置
 * 3.@RunWith()指定用那种驱动进行单元测试,默认就是junit
 *  @RunWith(SpringJUnit4ClassRunner.class)
 *  使用Spring的单元测试模块类执行标注了@Test注解的测试方法
 *  以前@Test注解只是由Junit执行
 * 好处:我们不用ioc.getBean()获取组件了;直接Autowired组件,Spring为我们自动装配
 */
@ContextConfiguration(locations = "classpath:ioc3.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class IOCTest01 {
    @Autowired
    BookServlet bookServlet;
    @Test
    public void test(){
        System.out.println(bookServlet);
    }
}
```
### 泛型依赖注入
注入一个组件的时候,他的泛型也是参考标准
```java
//BaseDao
public abstract class BaseDao<T> {

    public abstract void save();
}
//bookDao
@Repository
public class BookDao extends BaseDao<Book> {
    @Override
    public void save() {
        System.out.println("BookDao保存方法");
    }
}
//UserDao
@Repository
public class UserDao extends BaseDao<User>{
    @Override
    public void save() {
        System.out.println("保存用户");
    }
}
//BaseService
public class BaseService<T> {
    @Autowired
    BaseDao<T> baseDao;
    public void save(){
        System.out.println("自动注入的dao:"+baseDao);
        baseDao.save();
    }
}
//UserService
@Service
public class UserService extends BaseService<User>{
}
//BookService
@Service
public class BookService extends BaseService<Book>{
}
//测试
@Test
public void test08(){
    BookService bookService = ioc.getBean(BookService.class);
    UserService userService = ioc.getBean(UserService.class);
    bookService.save();
    userService.save();
}
```
## AOP
Aspect Oriented Programming : 面向切面编程

基于OOP基础之上新的编程思想;

指在程序运行期间,将某段代码动态的切入到指定方法的指定位置进行运行的编程方式
### 场景:计算器运行计算方法的时候添加日志记录
加日志记录:
1. 直接写在方法内部:不推荐,修改维护麻烦
2. 使用动态代理
    * 计算接口
    ```java
    package cn.glh.inter;
    public interface Calculator {
        int add(int i,int j);
        int sub(int i,int j);
        int mul(int i,int j);
        int div(int i,int j);
    }
    ```
    * 计算接口实现类
    ```java
    package cn.glh.inter.impl;
    import cn.glh.inter.Calculator;
    public class MyMathCalculator implements Calculator {
        @Override
        public int add(int i, int j) {
            int result = i+j;
            return result;
        }
        @Override
        public int sub(int i, int j) {
            int result = i-j;
            return result;
        }
        @Override
        public int mul(int i, int j) {
            int result = i*j;
            return result;
        }
        @Override
        public int div(int i, int j) {
            int result = i/j;
            return result;
        }
    }
    ```
    * 动态代理类
    ```java
    package cn.glh.proxy;
    import cn.glh.inter.Calculator;
    import cn.glh.utils.LogUtils;
    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.InvocationTargetException;
    import java.lang.reflect.Method;
    import java.lang.reflect.Proxy;
    import java.util.Arrays;
    /**
    * 帮Calculator.jsva生成代理对象
    */
    public class CalculatorProxy {
        /**
        *
        * @param calculator 被代理对象
        * @return 代理对象
        */
        public static Calculator getProcy(final Calculator calculator){
            ClassLoader loader = calculator.getClass().getClassLoader();//被代理对象的类加载器
            Class<?>[] interfaces = calculator.getClass().getInterfaces();//被代理对象所实现的所有接口
            InvocationHandler h= new InvocationHandler() {
                /**
                * @param proxy 代理对象;给jdk使用,任何时候都不要动这个对象
                * @param method 当前将要执行的目标对象的方法
                * @param args 方法传入的参数
                * @return
                * @throws Throwable
                */
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    //System.out.println("这是动态代理将要执行的方法");
                    //利用反射执行目标方法
                    //calculator:执行那个对象的方法
                    //invoke:目标方法执行后的返回值
                    Object invoke = null;
                    try {
                        LogUtils.logStart(method,args);
                        invoke = method.invoke(calculator, args);
                        LogUtils.logReturn(method,invoke);
                    } catch (Exception e) {
                        LogUtils.logException(method,e);
                        e.printStackTrace();
                    } finally {
                        LogUtils.logEnd(method);
                    }
                    return invoke;
                }
            };//方法执行器,帮我们目标对象执行目标方法

            Object o = Proxy.newProxyInstance(loader, interfaces, h);
            return (Calculator) o;
        }
    }
    ```
    * 日志工具类
    ```java
    package cn.glh.utils;
    import java.lang.reflect.Method;
    import java.util.Arrays;
    public class LogUtils {
        public static void logStart(Method method , Object args){
            System.out.println("["+method.getName()+"]方法开始执行了,用的参数列表是"+ Arrays.asList(args) +"");
        }
        public static void logReturn(Method method,Object result){
            System.out.println("["+method.getName()+"]方法执行完成了,计算结果是"+result+"");
        }
        public static void logException(Method method, Exception e) {
            System.out.println("["+method.getName()+"]方法执行出现了异常,异常信息是"+e.getCause());
        }
        public static void logEnd(Method method) {
            System.out.println("["+method.getName()+"]方法执行结束");
        }
    }
    ```
    * 测试类
    ```java
    package cn.glh.test;
    import cn.glh.inter.Calculator;
    import cn.glh.inter.impl.MyMathCalculator;
    import cn.glh.proxy.CalculatorProxy;
    import org.junit.Test;
    public class AOPTest {
        @Test
        public void test(){
            Calculator myMathCalculator = new MyMathCalculator();
            int add = myMathCalculator.add(1, 2);
            System.out.println(add);
            System.out.println("[==============================================]");
            Calculator procy = CalculatorProxy.getProcy(myMathCalculator);
            procy.add(2,6);
            procy.div(2,0);
        }
    }
    ```
    * 缺点:
    ```text
        1. 写起来难
        2. jdk默认的动态代理,如果目标对象没有实现任何接口,是无法为它创建代理对象的
    ```
3. spring的AOP
    1. 导包
    ```xml
    <dependencies>
        <!-- 添加Spring核心依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>3.1.1.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>commons-logging</groupId>
                    <artifactId>commons-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>3.1.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>3.1.1.RELEASE</version>
        </dependency>
        <!--    日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.21</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.21</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <!--测试工具-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
        </dependency>
        <!--    数据库连接池-->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>
        <!--    java 连接mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.25</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.0.0.RELEASE</version>
        </dependency>
    </dependencies>
    ```
    2. 写配置
        * 将目标类和切面类(封装了通知方法(在目标方法执行前后执行的方法))加入到ioc容器中
        * 告诉spring那个类是切面类`@Aspect:作用是把当前类标识为一个切面供容器读取`
        * 