---
layout: post
title: springboot整合sharding-jdbc实现分库分表、读写分离 
date: 2018-12-06
categories: blog
tags: [springboot,sharding-jdbc,分库分表,读写分离]
description: 
---

## pom.xml依赖
```javascript
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>LATEST</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>LATEST</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.40</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>LATEST</version>
        </dependency>
        <dependency>
            <groupId>io.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/io.shardingjdbc/sharding-jdbc-spring-boot-starter -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>
```

## application.xml配置
```javascript
##########分库分表配置##########
sharding.jdbc.datasource.names=ds
## 这里使用阿里的Druid连接池
#sharding.jdbc.datasource.ds0.type=com.alibaba.druid.pool.DruidDataSource
#sharding.jdbc.datasource.ds0.driver-class-name=com.mysql.jdbc.Driver
#sharding.jdbc.datasource.ds0.url=jdbc:mysql://localhost:3306/ds_0
#sharding.jdbc.datasource.ds0.username=root
#sharding.jdbc.datasource.ds0.password=1234

sharding.jdbc.datasource.ds.type=com.alibaba.druid.pool.DruidDataSource
sharding.jdbc.datasource.ds.driver-class-name=com.mysql.jdbc.Driver
sharding.jdbc.datasource.ds.url=jdbc:mysql://localhost:3306/test
sharding.jdbc.datasource.ds.username=root
sharding.jdbc.datasource.ds.password=root


spring.jpa.properties.hibernate.hbm2ddl.auto=none

##默认的分库策略：user_id为奇数-->数据库ds_1,user_id为偶数-->数据库ds_0
#sharding.jdbc.config.sharding.default-database-strategy.inline.sharding-column=id
#sharding.jdbc.config.sharding.default-database-strategy.inline.algorithm-expression=test$->{id % 2}
## 这里的t_order是逻辑表，由数据源名 + 表名组成，以小数点分隔。多个表以逗号分隔，支持inline表达式
sharding.jdbc.config.sharding.tables.t_user_vo.actualDataNodes=ds.t_user_vo_$->{0..1}
## 行表达式分片策略
sharding.jdbc.config.sharding.tables.t_user_vo.tableStrategy.inline.shardingColumn=id
sharding.jdbc.config.sharding.tables.t_user_vo.tableStrategy.inline.algorithmExpression=t_user_vo_$->{Math.abs(id.hashCode()) % 2}



#分片列
#sharding.jdbc.config.sharding.default-table-strategy.standard.sharding-column=USER_ID
#精确分片算法，用于=和IN,实现类
#sharding.jdbc.config.sharding.default-table-strategy.standard.precise-algorithm-class-name=com.solider76.oo.service.chat.config.MyShardingConfig
#范围分片算法，用于BETWEEN,实现类
#sharding.jdbc.config.sharding.default-table-strategy.standard.range-algorithm-class-name=com.solider76.oo.service.chat.config.MyShardingConfig
```

## Model表配置
```java
@Table(name = "t_user_vo",indexes = {@Index(name="index_user",columnList = "name,age")})
@Entity
@JsonIgnoreProperties(value = { "hibernateLazyInitializer", "handler" })
public class UserModel implements Serializable {

	private static final long serialVersionUID = 1139026566264645366L;

	@Id
    @GenericGenerator(name = "jpa-uuid", strategy = "uuid")
    @GeneratedValue(generator = "jpa-uuid")
    private String id;

    private String name;

    private int age;

    public UserModel() {
    }

    public UserModel(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

## Service 配置
```java
/**
 * Created by leospring on 2018/12/3.
 */
public interface UserService {

    List<UserModel> findAll();

    List<UserModel> findByAge(int age);

    UserModel save(UserModel userModel);

	UserModel getById(String id);
}
```
```java
/**
 * Created by leospring on 2018/12/3.
 */
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    public List<UserModel> findAll() {
        return userDao.findAll();
    }

    public List<UserModel> findByAge(int age) {
        return userDao.findAllByAge(age);
    }

    @Transactional
    public UserModel save(UserModel userModel) {
        return userDao.save(userModel);
    }

	public UserModel getById(String id) {
//		return userDao.findById(id).get();
		return userDao.getOne(id);
	}
}
```
## Repository配置
```java
/**
 * Created by leospring on 2018/12/3.
 */
public interface UserDao extends JpaRepository<UserModel, String> {

    List<UserModel> findAllByAge(int age);
}
```
## Controller代码
```java
/**
 * Created by leospring on 2018/12/3.
 */
@RestController
@RequestMapping("user")
public class UserController {

    @Autowired
    private UserService userService;

    @ResponseBody
    @RequestMapping("getAll")
    public List<UserModel> getUser(){
        return userService.findAll();
    }

    @ResponseBody
    @RequestMapping("getByAge")
    public List<UserModel> findByAge(int age){
        return userService.findByAge(age);
    }
    
    @ResponseBody
    @RequestMapping("save")
    public UserModel save(String name,int age){
        return userService.save(new UserModel(name,age));
    }
    
    @ResponseBody
    @RequestMapping("getById")
    public UserModel getById(String id){
        UserModel userModel = userService.getById(id);
        return userModel;
    }
}
```

## 启动类
```java
/**
 * Created by leospring on 2018/12/3.
 */
@SpringBootApplication
public class ApplicationStart {

    public static void main(String[] args) {
        SpringApplication.run(ApplicationStart.class,args);
    }
}
```
