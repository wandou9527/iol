## tk-mybatis 简介
tk-mybatis即通用mapper，使用它可以减少sql开发量，接口提供了丰富的crud方法，及Example相关的单表操作，简单调用即可。操作目前仅限单表。

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
此版本 springboot 集成 1.2.x 的 mapper-spring-boot-starter 运行会报错

```

- 在 properties 配置中：

```
mapper.mappers=tk.mybatis.mapper.common.Mapper
# Selective式的增改操作是否判断空字符串情况。
mapper.notEmpty=true 

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

该接口可以直接使用：

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

#### 2.2 数据库映射

通用 Mapper 中，默认情况下是将实体类字段按照驼峰转下划线形式的表名列名进行转换。

> 例如<br>
> 实体类的 userName 可以映射到表的 user_name 上。<br>
> 如果想要修改默认的转换方式，可以在后续的配置中，修改 style 全局配置。<br>

通用 Mapper 默认使用了几个简单的注解，其他 JPA 的注解默认并不支持，但是如果你开发自己的通用方法，你可以使用 JPA 注解或者引入自己的注解。

##### 2.2.1 @NameStyle 注解（Mapper） 

这个注解可以在类上进行配置，优先级高于对应的 style 全局配置。
注解支持以下几个选项：

```
normal,                     //原值
camelhump,                  //驼峰转下划线
uppercase,                  //转换为大写
lowercase,                  //转换为小写
camelhumpAndUppercase,      //驼峰转下划线大写形式
camelhumpAndLowercase,      //驼峰转下划线小写形式

```

使用时，直接在类上配置即可，例如：

```
@NameStyle(Style.camelhumpAndUppercase)
public class Country

```

配置该注解后，对该类和其中的字段进行转换时，会将形如 userName 的字段转换为表中的 USER_NAME 字段。

##### 2.2.2 @Table 注解（JPA）

@Table 注解可以配置 name,catalog 和 schema 三个属性，配置 name 属性后，直接使用提供的表名，不再根据实体类名进行转换。其他两个属性中，同时配置时，catalog 优先级高于 schema，也就是只有 catalog 会生效。

配置示例如下：

```
@Table(name = "sys_user")
public class User

将 User 实体映射到 sys_user 表

```

##### 2.2.3 @Column 注解（JPA）

```
@Column 注解支持 name, insertable 和 updateable 三个属性。

name 配置映射的列名。

insertable 对提供的 insert 方法有效，如果设置 false 就不会出现在 SQL 中。

updateable 对提供的 update 方法有效，设置为 false 后不会出现在 SQL 中。

配置示例如：

@Column(name = "user_name")
private String name;
除了直接映射 name 到 user_name 这种用法外，在使用关键字的情况，还会有下面的用法：

@Column(name = "`order`")
private String order;
对于关键字这种情况，通用 Mapper 支持自动转换，可以查看后续配置文档中的 wrapKeyword 配置。

```

##### 2.2.4 @ColumnType 注解（Mapper）

```
这个注解提供的 column属性和 @Column 中的 name 作用相同。但是 @Column 的优先级更高。

除了 name 属性外，这个注解主要提供了 jdbcType 属性和 typeHandler 属性。

jdbcType 用于设置特殊数据库类型时指定数据库中的 jdbcType。

typeHandler 用于设置特殊类型处理器，常见的是枚举。

用法示例如下：

@ColumnType(
        column = "countryname",
        jdbcType = JdbcType.VARCHAR,
        typeHandler = StringTypeHandler.class)
private String  countryname;

```

##### 2.2.5 @Transient 注解（JPA）

```
一般情况下，实体中的字段和数据库表中的字段是一一对应的，但是也有很多情况我们会在实体中增加一些额外的属性，这种情况下，就需要使用 @Transient 注解来告诉通用 Mapper 这不是表中的字段。

默认情况下，只有简单类型会被自动认为是表中的字段（可以通过配置中的 useSimpleType 控制）。

这里的简单类型不包含 Java 中的8种基本类型：

byte,short,int,long,float,double,char,boolean

这是因为在类中，基本类型会有默认值，而 MyBatis 中经常会需要判断属性值是否为空，所以不要在类中使用基本类型，否则会遇到莫名其妙的错误。

对于类中的复杂对象，以及 Map,List 等属性不需要配置这个注解。

对于枚举类型作为数据库字段的情况，需要看配置中的 enumAsSimpleType 参数。

配置示例：

@Transient
private String otherThings; //非数据库表中字段

```

##### 2.2.6 @Id 注解（JPA）

```
上面几个注解都涉及到映射。 @Id 注解和映射无关，它是一个特殊的标记，用于标识数据库中的主键字段。

正常情况下，一个实体类中至少需要一个标记 @Id 注解的字段，存在联合主键时可以标记多个。

如果表中没有主键，类中就可以不标记。

当类中没有存在标记 @Id 注解的字段时，你可以理解为类中的所有字段是联合主键。使用所有的 ByPrimaryKey 相关的方法时，有 where 条件的地方，会将所有列作为条件。

配置示例：

@Id
private Integer id;
或者联合主键：

@Id
private Integer userId;
@Id
private Integer roleId;

```


##### 2.2.7 @GeneratedValue 注解（JPA）

```

主键策略注解，用于配置如何生成主键。
如下配置是实现自增主键，并支持主键返回。

@Id
@GeneratedValue(generator = "JDBC")
private Long id;


```

##### 2.2.8 @Version 注解（Mapper）

```
@Version 是实现乐观锁的一个注解，大多数人都不需要。

乐观锁实现中，要求一个实体类中只能有一个乐观锁字段。

配置 @Version
想要使用乐观锁，只需要在实体中，给乐观锁字段增加 @tk.mybatis.mapper.annotation.Version 注解。

如想自己扩展实现，参考：https://github.com/abel533/Mapper/wiki/2.4-version

```

##### 2.2.9 @RegisterMapper 注解

```
为了解决通用 Mapper 中最常见的一个错误而增加的标记注解，该注解仅用于开发的通用接口，不是实体类上使用的，这里和其他注解一起介绍了。

4.0 版本提供的所有通用接口上都标记了该注解，因此自带的通用接口时，不需要配置 mappers 参数，该注解的具体用法会在 第五章 扩展通用接口 中介绍。

```


### 3 Example 用法

#### 3.1 mybatis-generator 生成的 Example

用法如下：

```
UserExample userExample = new UserExample();
userExample.createCriteria().andRealNameEqualTo("乔峰");
List<User> users = userMapper.selectByExample(userExample);

sql如下：

==>  Preparing: SELECT id,username,password,real_name,phone,wechat,sex,update_time FROM user WHERE ( real_name = ? ) 
==> Parameters: 乔峰(String)
<==      Total: 1


UserExample userExample1 = new UserExample();
userExample1.createCriteria().andIdBetween(5L, 10L);
List<User> users1 = userMapper.selectByExample(userExample1);

==>  Preparing: SELECT id,username,password,real_name,phone,wechat,sex,update_time FROM user WHERE ( id between ? and ? 
==> Parameters: 5(Long), 10(Long)
<==      Total: 5

```

生成的 UserExample 中包含了和字段相关的多种方法，根据自己的需要设置相应的条件即可。

#### 3.2 通用 Example

这是由通用 Mapper 提供的一个类，这个类和 MBG 生成的相比，需要自己设置属性名。这个类还额外提供了更多的方法。

##### 3.2.1 查询

示例：

```
Example example = new Example(User.class);
example.createCriteria().andGreaterThan("id", 5L).andLessThan("id", 11L);
example.or().andLessThan("id", 2L);
example.or().andEqualTo("realName", "摘星子").andEqualTo("username", "1120");
//todo 可以理解为每个 example.createCriteria() 就是在sql中生成个括号（）

sql：

==>  Preparing: SELECT id,username,password,real_name,phone,wechat,sex,update_time FROM user WHERE ( id > ? and id < ? ) or ( id < ? ) or ( real_name = ? and username = ? ) 
==> Parameters: 5(Long), 11(Long), 2(Long), 摘星子(String), 1120(String)
<==      Total: 7

```

##### 3.2.2 排序

示例：

```
Example example = new Example(User.class);
example.orderBy("realName").orderBy("phone").desc().orderBy("id").asc();
List<User> users = userMapper.selectByExample(example);

sql：

==>  Preparing: SELECT id,username,password,real_name,phone,wechat,sex,update_time FROM user order by real_name,phone DESC,id AS
==> Parameters: 
<==      Total: 20

```


## 注意事项
- 【删除慎用】通用mapper的删除方法为物理删除













