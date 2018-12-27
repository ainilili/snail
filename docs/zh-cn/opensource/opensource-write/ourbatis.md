## 一、Mybatis的不足之处
Mybatis是一款优秀的及其灵活的持久层框架，通过XML配置并映射到Mapper接口为Service层提供基础数据操作入口。

这么优秀的框架竟然还有不足之处？

俗话说人无完人，因为Mybatis实在是太灵活了，灵活到每个Mapper接口都需要定制对应的XML，所以就会引发一些问题。

### 问题一：配置文件繁多
假如一个系统中DB中涉及100张表，我们就需要写```100```个Mapper接口，还没完，最可怕的是，我们要为这```100```个Mapper接口定制与之对应的```100```套XML。而每个Mapper都必不可少的需要增删改查功能，我们就要写```100```遍增删改查，作为高贵的Java开发工程师，这是不能容忍的，于是```Mybatis Generator```诞生了，然而又会引发另一个问题！

### 问题二：维护困难
我们使用```Mybatis Generator```解决了问题一，再多的文件生成就是了，简单粗暴，貌似解决了所有的问题，Mybatis完美了！

不要高兴的太早，在系统刚刚建立起来时，我们使用```Mybatis Generator```生成了一堆XML，在开发过程中，产品忽然提了一个新的需求，项目经理根据这个需求在某张表中增加或变动了一个字段，这时，我猜你的操作是这样：
 - 1、找到对应表的XML
 - 2、将该XML中自定义的一段标签复制出来，保存在本地
 - 3、使用```Mybatis Generator```重新生成该表的XML
 - 4、将之覆盖当前的XML
 - 5、将自定义的一段标签再粘贴进新的XML中

在这个过程中，如果我们在第2步时漏复制了一段标签，等整个操作完成之后，又别是一番滋味在心头~

### 问题三：编写XML困难
假如肝不错，问题二也是小CASE，那么问题又来了，我们如何在繁长的XML中去编写和修改我们的XML呢。

当我们打开要编辑的XML，映入眼帘的就是1000多行的XML，其中900行都是通用的增删改查操作，要新增一个标签，我们需要拉至文件底部编写新的数据操作，要更新一个标签，我们需要通过```Ctrl + F```寻找目标标签再进行修改。

如何避免这些问题呢？

如何让Mybatis增强通用性又不失灵活呢？

## 二、使用Ourbatis辅助Mybatis
Ourbatis是一款Mybatis开发增强工具，小巧简洁，项目地址：
 - Github：[https://github.com/ainilili/ourbatis](https://github.com/ainilili/ourbatis)
 - Gitee：[https://gitee.com/ainilili/ourbatis](https://gitee.com/ainilili/ourbatis)
 - Wiki：[https://github.com/ainilili/ourbatis/wiki](https://github.com/ainilili/ourbatis/wiki)
 - Demo：[https://github.com/ainilili/ourbatis-simple](https://github.com/ainilili/ourbatis-simple)

特性：
 - **1**、简洁方便，可以让Mybatis无XML化开发。
 - **2**、优雅解耦，通用和自定义的SQL标签完全隔离，让维护更加轻松。
 - **3**、无侵入性，Mybatis和Ourbatis可同时使用，配置简洁。
 - **4**、灵活可控，通用模板可自定义及扩展。
 - **5**、部署快捷，只需要一个依赖，两个配置，即可直接运行。
 - **6**、多数据源，在多数据源环境下也可以照常使用。

### 关于Ourbatis使用的一个小Demo
环境：
 - Spring Boot 2.0.5.RELEASE
 - Ourbatis 1.0.5
 - JAVA 8
 - Mysql

 以```Spring Boot 2.0.5.RELEASE```版本为例，在可以正常使用Mybatis的项目中，```pom.xml```添加如下依赖：
 ```xml
    <dependency>
        	<groupId>com.smallnico</groupId>
        	<artifactId>ourbatis-spring-boot-starter</artifactId>
        	<version>1.0.5</version>
	</dependency>
```
在配置文件中增加一下配置：
```java
ourbatis.domain-locations=实体类所在包名
```
接下来，Mapper接口只需要继承```SimpleMapper```即可：
```java
import org.nico.ourbatis.domain.User;
  public interface UserMapper extends SimpleMapper<User, Integer>{
}
```
至此，一个使用Ourbatis的简单应用已经部署起来了，之后，你就可以使用一些Ourbatis默认的通用操作方法：
```java
	public T selectById(K key);

	public T selectEntity(T condition);

	public List<T> selectList(T condition);

	public long selectCount(Object condition);

	public List<T> selectPage(Page<Object> page);

	default PageResult<T> selectPageResult(Page<Object> page){
		long total = selectCount(page.getEntity());
		List<T> results = null;
		if(total > 0) {
			results = selectPage(page);
		}
		return new PageResult<>(total, results);
	}

	public K selectId(T condition);

	public List<K> selectIds(T condition);

	public int insert(T entity);

	public int insertSelective(T entity);

	public int insertBatch(List<T> list);

	public int update(T entity);

	public int updateSelective(T entity);

	public int updateBatch(List<T> list);

	public int delete(T condition);

	public int deleteById(K key);

	public int deleteBatch(List<K> list);
```

### Mapper自定义方法
在很多场景中，我们使用以上的自带的通用方法远远不能满足我们的需求，我们往往需要额外扩展新的Mapper方法、XML标签，我们使用了Ourbatis之后该如何实现呢？

首先看一下我们的需求，在上述Demo中，我们在UserMapper中增加一个方法```selectNameById```：
```java
import org.nico.ourbatis.domain.User;
public interface UserMapper extends SimpleMapper<User, Integer>{
    public String selectNameById(Integer userId);
}
```
和Mybatis一样，需要在```resources```资源目录下新建一个文件夹```ourbatis-mappers```，然后在其中新建一个XML文件，命名规则为：
```
DomainClassSimpleName + Mapper.xml
```
其中```DomainClassSimpleName```就是我们实体类的类名，这里是为```User```，那么新建的XML名为```UserMapper.xml```。
```java
src/main/resources
 - ourbatis-mappers
   - UserMapper.xml
```
之后，打开```UserMapper.xml```，开始编写Mapper中```selectNameById```方法对应的标签：
```xml
<select id="selectNameById" resultType="java.lang.String">
    select name from user where id = #{userId}
</select>
```
注意，整个文件中只需要写标签就行了，其他的什么都不需要，这是为什么呢？深入之后你就会明白，这里先不多说！

接下来，就没有接下来了，可以直接使用```selectNameById```方法了。
### 深入了解Ourbatis
![ourbatis 流程图](https://github.com/ainilili/snail/blob/master/docs/images/ourbatis-1-1.jpg?raw=true)

当服务启动的时候，Ourbatis首先会扫描```ourbatis.domain-locations```配置包下的所有实体类，将之映射为与之对应的表结构数据：
![ourbatis Mapping](https://github.com/ainilili/snail/blob/master/docs/images/ourbatis-1-2.jpg?raw=true)

然后通过```ourbatis.xml```的渲染，生成一个又一个的XML文件，最后将之重新Build到Mybatis容器中！

整个过程分为两个核心点：
 - 1、映射实体类为元数据
 - 2、使用```ourbatis.xml```渲染元数据为XML文件

我会一一介绍之~
#### 映射实体类为元数据
在映射时，我们要根据自己数据库字段命名的风格去调整映射规则，就需要在第1个核心点中去做处理，Ourbatis使用包装器来完成：
```
public interface Wrapper<T> {
	public String wrapping(T value);
}
```
对于需要映射的字段，如**表名**和**表字段名**，它们都将会经过一个包装器链条的处理之后再投入到```ourbatis.xml```中做渲染，这样就使得我们可以自定义包装器出更换映射的字段格式，具体方式可以参考官方Wiki：[Wrapper包装器](https://github.com/ainilili/ourbatis/wiki/Wrapper%E5%8C%85%E8%A3%85%E5%99%A8)

#### 使用```ourbatis.xml```渲染元数据为XML文件
而在于第2个核心点中，Ourbatis通过自定义标签做模板渲染，我们可以先看一下官方默认的```ourbatis.xml```内部构造：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="@{mapperClassName}">
	<resultMap id="BaseResultMap" type="@{domainClassName}">
		<ourbatis:foreach list="primaryColumns" var="elem">
			<id column="@{elem.jdbcName}" property="@{elem.javaName}" />
		</ourbatis:foreach>
		<ourbatis:foreach list="normalColumns" var="elem">
			<result column="@{elem.jdbcName}" property="@{elem.javaName}" />
		</ourbatis:foreach>
	</resultMap>

	<sql id="Base_Column_List">
		<ourbatis:foreach list="allColumns" var="elem"
			split=",">
			`@{elem.jdbcName}`
		</ourbatis:foreach>
	</sql>

	<select id="selectById" parameterType="java.lang.Object"
		resultMap="BaseResultMap">
		select
		<include refid="Base_Column_List" />
		from @{tableName}
		where 1 = 1
		<ourbatis:foreach list="primaryColumns" var="elem">
			and `@{elem.jdbcName}` = #{@{elem.javaName}}
		</ourbatis:foreach>
	</select>

	<select id="selectEntity" parameterType="@{domainClassName}"
		resultMap="BaseResultMap">
		select
		<include refid="Base_Column_List" />
		from @{tableName}
		where 1 = 1
		<ourbatis:foreach list="allColumns" var="elem">
			<if test="@{elem.javaName} != null">
				and `@{elem.jdbcName}` = #{@{elem.javaName}}
			</if>
		</ourbatis:foreach>
		limit 1
	</select>

	<select id="selectCount" parameterType="@{domainClassName}"
		resultType="long">
		select count(0)
		from @{tableName}
		where 1 = 1
		<ourbatis:foreach list="allColumns" var="elem">
			<if test="@{elem.javaName} != null">
				and `@{elem.jdbcName}` = #{@{elem.javaName}}
			</if>
		</ourbatis:foreach>
		limit 1
	</select>

	<select id="selectPage"
		parameterType="org.nico.ourbatis.entity.Page"
		resultMap="BaseResultMap">
		select
		<include refid="Base_Column_List" />
		from @{tableName}
		where 1 = 1
		<if test="entity != null">
			<ourbatis:foreach list="allColumns" var="elem">
				<if test="entity.@{elem.javaName} != null">
					and `@{elem.jdbcName}` = #{entity.@{elem.javaName}}
				</if>
			</ourbatis:foreach>
		</if>
		<if test="orderBy != null">
			order by ${orderBy}
		</if>
		<if test="start != null and end != null">
			limit ${start},${end}
		</if>
	</select>

	<select id="selectList" parameterType="@{domainClassName}"
		resultMap="BaseResultMap">
		select
		<include refid="Base_Column_List" />
		from @{tableName}
		where 1 = 1
		<ourbatis:foreach list="allColumns" var="elem">
			<if test="@{elem.javaName} != null">
				and `@{elem.jdbcName}` = #{@{elem.javaName}}
			</if>
		</ourbatis:foreach>
	</select>

	<select id="selectId" parameterType="@{domainClassName}"
		resultType="java.lang.Object">
		select
		<ourbatis:foreach list="primaryColumns" var="elem"
			split=",">
			`@{elem.jdbcName}`
		</ourbatis:foreach>
		from @{tableName}
		where 1 = 1
		<ourbatis:foreach list="allColumns" var="elem">
			<if test="@{elem.javaName} != null">
				and `@{elem.jdbcName}` = #{@{elem.javaName}}
			</if>
		</ourbatis:foreach>
		limit 1
	</select>

	<select id="selectIds" parameterType="@{domainClassName}"
		resultType="java.lang.Object">
		select
		<ourbatis:foreach list="primaryColumns" var="elem"
			split=",">
			`@{elem.jdbcName}`
		</ourbatis:foreach>
		from @{tableName}
		where 1 = 1
		<ourbatis:foreach list="normalColumns" var="elem">
			<if test="@{elem.javaName} != null">
				and `@{elem.jdbcName}` = #{@{elem.javaName}}
			</if>
		</ourbatis:foreach>
	</select>

	<delete id="deleteById" parameterType="java.lang.Object">
		delete
		from @{tableName}
		where 1=1
		<ourbatis:foreach list="primaryColumns" var="elem">
			and `@{elem.jdbcName}` = #{@{elem.javaName}}
		</ourbatis:foreach>
	</delete>

	<insert id="insert" keyProperty="@{primaryColumns.0.jdbcName}"
		useGeneratedKeys="true" parameterType="@{domainClassName}">
		insert into @{tableName}
		(
		<include refid="Base_Column_List" />
		)
		values (
		<ourbatis:foreach list="allColumns" var="elem"
			split=",">
			#{@{elem.javaName}}
		</ourbatis:foreach>
		)
	</insert>

	<insert id="insertSelective"
		keyProperty="@{primaryColumns.0.jdbcName}" useGeneratedKeys="true"
		parameterType="@{domainClassName}">
		insert into @{tableName}
		(
		<ourbatis:foreach list="primaryColumns" var="elem"
			split=",">
			`@{elem.jdbcName}`
		</ourbatis:foreach>
		<ourbatis:foreach list="normalColumns" var="elem">
			<if test="@{elem.javaName} != null">
				,`@{elem.jdbcName}`
			</if>
		</ourbatis:foreach>
		)
		values (
		<ourbatis:foreach list="primaryColumns" var="elem">
			#{@{elem.javaName}}
		</ourbatis:foreach>
		<ourbatis:foreach list="normalColumns" var="elem">
			<if test="@{elem.javaName} != null">
				, #{@{elem.javaName}}
			</if>
		</ourbatis:foreach>
		)
	</insert>

	<insert id="insertBatch"
		keyProperty="@{primaryColumns.0.jdbcName}" useGeneratedKeys="true"
		parameterType="java.util.List">
		insert into @{tableName}
		(
		<include refid="Base_Column_List" />
		)
		values
		<foreach collection="list" index="index" item="item"
			separator=",">
			(
			<ourbatis:foreach list="allColumns" var="elem"
				split=",">
				#{item.@{elem.javaName}}
			</ourbatis:foreach>
			)
		</foreach>
	</insert>

	<update id="update" parameterType="@{domainClassName}">
		update @{tableName}
		<set>
			<ourbatis:foreach list="normalColumns" var="elem"
				split=",">
				`@{elem.jdbcName}` = #{@{elem.javaName}}
			</ourbatis:foreach>
		</set>
		where 1=1
		<ourbatis:foreach list="primaryColumns" var="elem">
			and `@{elem.jdbcName}` = #{@{elem.javaName}}
		</ourbatis:foreach>
	</update>

	<update id="updateSelective" parameterType="@{domainClassName}">
		update @{tableName}
		<set>
			<ourbatis:foreach list="primaryColumns" var="elem"
				split=",">
				`@{elem.jdbcName}` = #{@{elem.javaName}}
			</ourbatis:foreach>
			<ourbatis:foreach list="normalColumns" var="elem">
				<if test="@{elem.javaName} != null">
					,`@{elem.jdbcName}` = #{@{elem.javaName}}
				</if>
			</ourbatis:foreach>
		</set>
		where 1=1
		<ourbatis:foreach list="primaryColumns" var="elem">
			and `@{elem.jdbcName}` = #{@{elem.javaName}}
		</ourbatis:foreach>
	</update>

	<update id="updateBatch" parameterType="java.util.List">
		<foreach collection="list" index="index" item="item"
			separator=";">
			update @{tableName}
			<set>
				<ourbatis:foreach list="normalColumns" var="elem"
					split=",">
					`@{elem.jdbcName}` = #{item.@{elem.javaName}}
				</ourbatis:foreach>
			</set>
			where 1=1
			<ourbatis:foreach list="primaryColumns" var="elem">
				and `@{elem.jdbcName}` = #{item.@{elem.javaName}}
			</ourbatis:foreach>
		</foreach>
	</update>

	<delete id="deleteBatch" parameterType="java.util.List">
		delete from @{tableName} where @{primaryColumns.0.jdbcName} in
		<foreach close=")" collection="list" index="index" item="item"
			open="(" separator=",">
			#{item}
		</foreach>
	</delete>

	<delete id="delete" parameterType="@{domainClassName}">
		delete from @{tableName} where 1 = 1
		<ourbatis:foreach list="allColumns" var="elem">
			<if test="@{elem.javaName} != null">
				and `@{elem.jdbcName}` = #{@{elem.javaName}}
			</if>
		</ourbatis:foreach>
	</delete>

	<ourbatis:ref path="classpath:ourbatis-mappers/@{domainSimpleClassName}Mapper.xml" />
</mapper>
```
可以看出来，```ourbatis.xml```内容类似于原生的Mybatis的XML，不同的是，有两个陌生的标签：
 - ourbatis:foreach 对元数据中的列表进行循环渲染
 - ourbatis:ref 引入外界文件内容

这是Ourbatis中独有的标签，Ourbatis也提供着对应的入口让我们去自定义标签：
```
Class: org.nico.ourbatis.Ourbatis
Field:
public static final Map<String, AssistAdapter> ASSIST_ADAPTERS = new HashMap<String, AssistAdapter>(){
		private static final long serialVersionUID = 1L;
		{
			put("ourbatis:foreach", new ForeachAdapter());
			put("ourbatis:ref", new RefAdapter());
		}
	};
```
我们可以修改```org.nico.ourbatis.Ourbatis```类中的静态参数```ASSIST_ADAPTERS```去删除、更新和添加自定义标签，需要实现一个标签适配器，我们可以看一下最简单的```RefAdapter```适配器的实现：
```java
public class RefAdapter extends AssistAdapter{
	@Override
	public String adapter(Map<String, Object> datas, NoelRender render, Document document) {
		String path = render.rending(datas, document.getParameter("path"), "domainSimpleClassName");
		String result =  StreamUtils.convertToString(path.replaceAll("classpath:", ""));
		return result == null ? "" : result.trim();
	}
}
```
Ourbatis中只定义了上述两个自定义标签已足够满足需求，通过```foreach```标签，将元数据中的集合遍历渲染，通过```ref```标签引入外界资源，也就是我们之前所说的对Mapper接口中方法的扩展！
```
<ourbatis:ref path="classpath:ourbatis-mappers/@{domainSimpleClassName}Mapper.xml" />
```
其中的path就是当前项目classpath路径的相对路径，而```@{domainSimpleClassName}```就代表着实体类的类名，更多的系统参数可以参考Wiki：[元数据映射](https://github.com/ainilili/ourbatis/wiki/%E5%85%83%E6%95%B0%E6%8D%AE%E6%98%A0%E5%B0%84)

通过这种模板渲染的机制，Ourbatis是相当灵活的，我们不仅可以通过引入外部文件进行扩展，当我们需要添加或修改通用方法时，我们可以可以自定义```ourbatis.xml```的内容，如何做到呢？复制一份将之放在资源目录下就可以了！

看到这里，相信大家已经知道Ourbatis的基本原理已经使用方式，我就再次不多说了，更多细节可以去官方Wiki中阅读：[Ourbtis Wiki](https://github.com/ainilili/ourbatis/wiki/%E5%85%83%E6%95%B0%E6%8D%AE%E6%98%A0%E5%B0%84)
