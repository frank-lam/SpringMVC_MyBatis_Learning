<!-- TOC -->

- [Spring的bean管理（注解）](#spring的bean管理注解)
    - [注解介绍](#注解介绍)
    - [Spring注解开发准备](#spring注解开发准备)
- [注解注入属性](#注解注入属性)
- [配置文件和注解混合使用](#配置文件和注解混合使用)

<!-- /TOC -->

## Spring的bean管理（注解）

### 注解介绍

1. 代码里面特殊标记，使用注解可以完成功能

2. 注解写法 @注解名称(属性名称=属性值)

3. 注解使用在类上面，方法上面 和 属性上面

### Spring注解开发准备

1. 导入 jar 包  

    （1）导入基本的 jar 包（4个jar包和log4j）

    （2）导入 aop 的 jar 包 （spring-aop-4.3.9.RELEASE.jar  ）

2. 创建类，创建方法  

3. 创建 spring 配置文件，引入约束  

    （1）做 ioc 基本功能，引入约束 beans  

    （2）做 spring 的 ioc 注解开发，引入新的约束  

    ```xml
    xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"  
    ```
    （3）开启注解扫描  
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
        <!-- 开启注解扫描
            到包里面扫描类、方法、属性上面是否有注解
        -->
        <context:component-scan base-package="me.test.domain"></context:component-scan>
        
        <!-- 了解：扫描属性上的注解（范围没有上面的大，使用少） -->
        <context:annotation-config></context:annotation-config>
    </beans>
    ```

4. 注解创建对象 

    在创建对象的类上面使用注解实现  

    ```java
    package me.test.domain;
    import org.springframework.stereotype.Component;
    
    @Component(value="user")
    public class User {
        public void add() {
            System.out.println("add...");
        }
    }
    ```
    测试
    ```java
    //注解创建对象
    @Test
    public void userTest1() {
    	ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
    	User user = (User) context.getBean("user");
    	System.out.println(user);
    	user.add();
    }
    
    // 结果：me.test.domain.User@d6da883  add...
    ```
* 创建对象有四个注解 

    1. @Component 
       下面三个是衍生注解  

    2. @Controller web层  

    3. @Service 业务层  

    4. @Repository 持久层  

       目前这四个注解功能是一样的，都创建对象  
* 创建对象单实例还是多实例
    ```java
    @Component(value="user")  
    @Scope(value="prototype")   //多实例
    ```
## 注解注入属性
创建 service 类，创建 dao 类，在 service 得到 dao 对象  
1. 创建 dao 和 service 对象

```java
package me.test.domain;
import org.springframework.stereotype.Repository;

@Repository(value="userDao")
public class UserDao {
    public void add() {
        System.out.println("dao...");
    }
}
```
在 service 类里面定义 dao 类型属性
**@Autowired** 自动注入：根据 UserDao 类名称找到类对应的对象，注入进来
**@Resource(name="userDao")** 第二种注解方式  

```java
package me.test.domain;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service(value="userService")
public class UserService {

    //得到UserDao对象
    //1.定义UserDao类型属性
    //在UserDao属性上使用注解，完成对象注入

    // 第一种方式 @Autowired
    @Resource(name="userDao") // 第二种方式 name的值就是你想注入对象的名称
    private UserDao userDao;
    public void add() {
        System.out.println("service...");
        System.out.println(userDao);
        userDao.add();
    }
}
```

```xml
<context:component-scan base-package="me.test.domain"></context:component-scan>
```
```java
public void userTest2() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
    UserService userService =(UserService) context.getBean("userService");
    System.out.println(userService);
    userService.add();
}
//结果：
me.test.domain.UserService@6c3f5566
service...
me.test.domain.UserDao@12405818
dao...
```
## 配置文件和注解混合使用
1. 创建对象操作使用配置文件方式实现  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 开启注解扫描
        到包里面扫描类、方法、属性上面是否有注解
        -->
    <context:component-scan base-package="me.test.domain"></context:component-scan>

    <bean id="bookService" class="me.test.domain.UserService"></bean>
    <bean id="bookDao" class="me.test.domain.UserDao"></bean>
</beans>
```

2. 注入属性的操作使用注解方式实现

```java
@Resource(name="userDao") 
private UserDao userDao;
//得到UserDao对象
```