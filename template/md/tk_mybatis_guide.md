## tk-mybatis 简介
tk-mybatis即通用mapper，使用它可以减少sql开发量，接口提供了丰富的crud方法，及Example相关的单表操作，简单调用即可。

## 正确的打开姿势

### 1 通用 mapper 集成

通用 Mapper 有很多种集成方式，这里会介绍大部分情况下的配置方式。<br>
Java 编码方式集成是最少见的一种情况，但是通过这种集成方式可以很容易让大家看清通用 Mapper 集成的入口。<br>
和 Spring 集成是最常见的，Spring Boot 也在慢慢成为主流，为了便于在集成通用 Mapper 的情况下仍然可以和第三方的工具集成，这里也会有很多种集成的方式。<br>
**本例主讲 SpringBoot 集成的方式。**<br>

-  **依赖：**

```
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.1.7</version>
</dependency>

更新日志：
2.0.4 Aug, 2018

此例版本关系：
mapper-spring-boot-starter 1.1.7 
springboot 1.5.2
此版本 springboot 集成 1.2.x 的 mapper-starter 运行会报错

```

- 在 properties 配置中：

```
mapper.mappers=tk.mybatis.mapper.common.Mapper,tk.mybatis.mapper.common.Mapper2
mapper.notEmpty=false

```

- Mapper 接口

```
每个接口需 extend tk.mybatis.mapper.common.Mapper<> 并指定泛型即实体类 
接口上加 @Mapper 注解

```

### 2 对象关系映射

#### 2.1 简单示例

示例针对 MySql 数据库（数据库对主键影响较大，和 insert 关系密切）。

数据库有如下表：

```
CREATE TABLE user
(
	id          BIGINT AUTO_INCREMENT
		PRIMARY KEY,
	username    VARCHAR(64)                        NULL
	COMMENT '用户名，服务端生成，用户的公开唯一标识，。一般为纯数字组成的字符串',
	phone       VARCHAR(20) DEFAULT ''             NULL,
	birth       VARCHAR(50) DEFAULT '0'            NULL
	COMMENT '生日',
	wechat      VARCHAR(64)                        NULL,
	real_name   VARCHAR(20)                        NULL
	COMMENT '真实名',
	password    VARCHAR(40)                        NULL,
	sex         TINYINT(3) UNSIGNED DEFAULT '0'    NULL
	COMMENT '0weizhi；1男；2女；',
	nick        VARCHAR(50)                        NULL
	COMMENT '昵称',
	idcard      VARCHAR(50) DEFAULT '2300'         NULL
	COMMENT '身份证号',
	create_time DATETIME DEFAULT CURRENT_TIMESTAMP NULL,
	update_time DATETIME DEFAULT CURRENT_TIMESTAMP NULL
);


```

对应的 Java 实体类型如下：

```
@Data
public class User {

    @Id
    @GeneratedValue(generator = "JDBC") //支持自增id的返回
    private Long id;
    private String username;
    private String password;
    private String realName;
    private String phone;
    private String wechat;
    private Integer sex;
    private Date updateTime;

}

```

最简单的情况下，只需要一个 @Id 标记字段为主键即可。数据库中的字段名和实体类的字段名是完全相同的，这中情况下实体和表可以直接映射。

**提醒：如果实体类中没有一个标记 @Id 的字段，当你使用带有 ByPrimaryKey 的方法时，所有的字段会作为联合主键来使用，也就会出现类似 where id = ? and countryname = ? and countrycode = ? 的情况。**

通用 Mapper 提供了大量的通用接口，这里以最常用的 Mapper 接口为例

该实体类对应的数据库操作接口如下：

```
import tk.mybatis.mapper.common.Mapper;

@org.apache.ibatis.annotations.Mapper
public interface UserMapper extends Mapper<User> {
}

```

只要配置 MyBatis 时能注册或者扫描到该接口，该接口提供的方法就都可以使用。该接口默认继承的方法如下：

- selectOne
- select
- selectAll
- selectCount
- selectByPrimaryKey
- 方法太多，省略其他...

从 MyBatis 中获取该接口后就可以直接使用：

```
//从 MyBatis 或者 Spring 中获取 countryMapper，然后调用 selectAll 方法
List<User> userList = userMapper.selectAll();
//根据主键查询
User user = userMapper.selectByPrimaryKey(1L);
//或者使用对象传参，适用于1个字段或者多个字段联合主键使用
User query = new User();
query.setId(1);
user = userMapper.selectByPrimaryKey(query);

```


