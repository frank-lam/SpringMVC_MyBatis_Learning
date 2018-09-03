## Spring的jdbcTemplate操作
*   spring框架一站式框架  
    （1）针对javaee三层，每一层都有解决技术  
    （2）在dao层，使用 jdbcTemplate  
*   spring对不同的持久化层技术都进行封装  
    |ORM持久化技术|模板类|  
    |:-:|:-|
    |JDBC|org.springframework.jdbc.core.JdbcTemplate|  
    |Hibernate5|org.springframework.orm.hibernate5.HibernateTemplate|  
    |iBatis(MyBatis)|org.springframework.orm.ibatis.SqlMapClientTemplate|  
    |JPA|org.springframework.orm.jpa.JpaTemplate|  
    
    jdbcTemplate对jdbc进行封装  

*   jdbcTemplate使用和dbutils使用很相似，都对数据库进行crud操作  
### 增加
1.  导入jdbcTemplate使用的jar包  
    spring-tx-4.3.9.RELEASE.jar spring事务包  
    spring-jdbc-4.3.9.RELEASE.jar  
2.  创建对象，设置数据库信息  
3.  创建jdbcTemplate对象，设置数据源  
4.  调用jdbcTemplate对象里面的方法实现操作  
```java
	@Test
	public void addTest() {
		//设置数据库信息
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
		dataSource.setDriverClassName("org.mariadb.jdbc.Driver");
		dataSource.setUrl("jdbc:mariadb://localhost:3306/test1");
		dataSource.setUsername("root");
		dataSource.setPassword("root");
		
		//创建jdbcTemplate对象，设置数据源
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		
		//调用jdbctemplate对象的方法实现操作
		//创建sql语句
		String sql = "insert into test values(?,?)";
		int rows = jdbcTemplate.update(sql, "张三","123");
		System.out.println(rows);
	}
```
### 修改
```java
String sql = "update test set password=? where username=?";
jdbcTemplate.update(sql, "456","张三");
```
### 删除
```java
String sql = "delete from test where username=?";
jdbcTemplate.update(sql, "张三");
```
### 查询
使用jdbcTemplate实现查询操作
```
相对于dbUtils
使用dbUtils，有接口ResultSetHandler  
dbUtils提供了针对不同的结果实现类  

jdbcTemplate实现查询，有接口RowMapper  
jdbcTemplate针对这个接口没有提供实现类，得到不同的类型数据需要自己进行数据封装  
```
*   查询返回某一个值  
    public <T> T queryForObject(String sql, Class<T> requiredType)  
    （1）第一个参数是sql语句  
    （2）第二个参数 返回类型的class  
    ```java
    String sql = "select count(*) from test";
	int rows = jdbcTemplate.queryForObject(sql, Integer.class);
    ```
*   查询返回对象  
    public <T> T queryForObject(String sql, Object[] args, RowMapper<T> rowMapper)  
    第一个参数是sql语句  
    第二个参数是 RowMapper，是接口，类似于dbutils里面接口  
    第三个参数是 可变参数  
    ```java
    class MyRowMapper implements RowMapper<User> {

        @Override
        public User mapRow(ResultSet rs, int num) throws SQLException {
            //1从结果集中把数据得到
            String username = rs.getString("username");
            String password = rs.getString("password");
            
            //2把得到数据封装到对象里面
            User user = new User();
            user.setUsername(username);
            user.setPassword(password);
            
            return user;
        }
    }
    ```
    ```java
    String sql = "select * from test where username=?";
	User user = jdbcTemplate.queryForObject(sql, new MyRowMapper(), "mary");
    ```
*   查询返回list集合  
    public <T> List<T> query(String sql, RowMapper<T> rowMapper)   
    ```java
    String sql = "select * from test";
	List<User> list = jdbcTemplate.query(sql, new MyRowMapper());
    ```
以上写的代码都是在单元测试中的写到的，真正开发中jdbc模板要用到dao层，dao里边要用到dao模板，spring中有ioc，ioc就是把对象创建交给spring容器管理，在dao里边可以写这个设置数据库信息的代码，但不建议这么写，把这些过程交给spring配置实现  
## Spring配置c3p0连接池和dao使用jdbcTemplate
### Spring配置c3p0连接池
1. 导入jar包  
2. 创建spring配置文件，配置连接池  
    以前代码中实现方式
    ```java
        //c3p0设置数据库信息
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setDriverClass("org.mariadb.jdbc.Driver");
		dataSource.setJdbcUrl("jdbc:mariadb://localhost:3306/test1");
		dataSource.setUser("root");
		dataSource.setPassword("root");
    ```
    将代码在spring配置文件中进行配置
    ```xml
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!-- 注入属性值 -->
        <property name="driverClass" value="org.mariadb.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mariadb://localhost:3306/test1"></property>
        <property name="user" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
    ```
### dao使用jdbcTemplate
1. 创建service和dao，配置service和dao对象，在service注入dao对象  
2. 创建jdbcTemplate对象，把模板对象注入到dao里面  
3. 在jdbcTemplate对象里面注入dataSource  
```xml
    <!-- 配置对象 -->
    <bean id="userService" class="me.test.service.UserService">
        <property name="userDao" ref="userDao" ></property>
    </bean>
    <bean id="userDao" class="me.test.dao.UserDao">
        <!-- 注入jdbcTemplate对象 -->
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>

    <!-- 配置jdbcTemplate对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 把dataSourece传递到模板对象里面 -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```
dao层
```java
package me.test.dao;

import org.springframework.jdbc.core.JdbcTemplate;

public class UserDao {
	
	//得到jdbcTemplate对象
	private JdbcTemplate jdbcTemplate;

	public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}
	
	//添加操作
	public void add() {
		String sql = "insert into test values(?,?)";
		jdbcTemplate.update(sql, "张三", "123");
	}
}
```
## Spring的事务管理
### 事务概念
1.  什么事务  
    事务是操作中最基本的单元，表示一组操作要么都成功，有一个失败那么所有都失败  
2.  事务特性  
    原子性 一致性 隔离性 持久性  
3.  不考虑隔离性产生读问题  
    隔离性：多个事务之间没有影响  
    （1）脏读  
    （2）不可重复读  
    （3）虚读  
4.  解决读问题  
    设置隔离级别  
### Spring事务管理api
在hibernate时要写很多行代码实现  
创建sessionfactory session 开启提交回滚事务。要自己写代码操作事务  
用到spring后事务的代码就不用我们写了，交给spring通过配置来完成  
1. spring事务管理两种方式  
    第一种 编程式事务管理（不用）  
    第二种 声明式事务管理  
    （1） 基于xml配置文件实现  
    （2） 基于注解实现  
2. spring事务管理的api介绍  
    接口：PlatformTransactionManager 事务管理器  
    
### 搭建转账环境
1.  创建数据库表account，添加数据  
2.  创建service和dao类，完成注入关系  
    在bean.xml中引入约束，并在项目中导入aop tx等包  
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:aop="http://www.springframework.org/schema/aop"
            xmlns:tx="http://www.springframework.org/schema/tx" xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    </beans>
    ```
### 声明式事务管理（xml配置）
配置文件方式使用aop思想配置  
1.  配置事务管理器，指定对那个数据库进行操作  
2.  配置事务增强  
3.  配置切面  
```xml
    <!-- 1.配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入dataSource -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <!-- 2.配置事务增强 -->
    <tx:advice id="txadvice" transaction-manager="transactionManager">
        <!-- 做事务操作 -->
        <tx:attributes>
            <!-- 设置进行事务操作的方法匹配规则 -->
            <tx:method name="accout*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    
    <!-- 3配置切面 -->
    <aop:config>
        <!-- 切入点 -->
        <aop:pointcut expression="execution(* me.test.service.OrderService.*(..))" id="pointcut1"/>
        <!-- 切面 
       advice-ref 事务应用在哪个切入点上
        -->
        <aop:advisor advice-ref="txadvice" pointcut-ref="pointcut1"/>
    </aop:config>
</beans>
```
### 声明式事务管理（注解）
1.  配置事务管理器  
2.  配置事务注解  
    ```xml
    <!-- 1.配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!-- 2开启事务注解 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    ```
3.  在要使用事务的方法所在类上面添加注解  
    ```java
    @Transaction
    public class OrederService{

    }
    ```