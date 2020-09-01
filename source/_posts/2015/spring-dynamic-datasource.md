layout: post
title: "spring动态数据源"
date: "2015-12-04 14:29"
categories: 技术
---
# 核心思想
通过使用Spring提供的AbstractRoutingDataSource替代原有 dataSource，在每次操作数据库的时候通过类中的determineTargetDataSource()方法获取当前数据源。这样我们就可以通过切面技术，在不同的切面，切入不同的数据源名称，使Spring获取的时候拿到的是不同的数据源。就相当于在多个 dataSource前面增加了一个路由层。
# 基础用法
## 原理
通过查看AbstractRoutingDataSource的determineTargetDataSource实现，可以发现基本原理就是通过determineCurrentLookupKey()获取当前数据源的 key，然后从resolvedDataSources获取对应数据源并返回。无法获取则返回默认配置数据源。而resolvedDataSources是通过targetDataSources初始化的。也就是说，我们在使用中配置两个地方。
1. defaultTargetDataSource
2. targetDataSources
```
	public void afterPropertiesSet() {
		if (this.targetDataSources == null) {
			throw new IllegalArgumentException("Property 'targetDataSources' is required");
		}
		this.resolvedDataSources = new HashMap<Object, DataSource>(this.targetDataSources.size());
		for (Map.Entry<Object, Object> entry : this.targetDataSources.entrySet()) {
			Object lookupKey = resolveSpecifiedLookupKey(entry.getKey());
			DataSource dataSource = resolveSpecifiedDataSource(entry.getValue());
			this.resolvedDataSources.put(lookupKey, dataSource);
		}
		if (this.defaultTargetDataSource != null) {
			this.resolvedDefaultDataSource = resolveSpecifiedDataSource(this.defaultTargetDataSource);
		}
	}
```
```
    protected DataSource determineTargetDataSource() {
		Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
		Object lookupKey = determineCurrentLookupKey();
		DataSource dataSource = this.resolvedDataSources.get(lookupKey);
		if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
			dataSource = this.resolvedDefaultDataSource;
		}
		if (dataSource == null) {
			throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
		}
		return dataSource;
	}
```
## 步骤1
通过 ThreadLocal 记录获取当前使用的数据源 Key
```
public class DataSourceHolder {
    //线程本地环境
    private static final ThreadLocal<String> dataSources = new ThreadLocal<String>();
    //设置数据源
    public static void setDataSource(String customerType) {
        dataSources.set(customerType);
    }
    //获取数据源
    public static String getDataSource() {
        return (String) dataSources.get();
    }
    //清除数据源
    public static void clearDataSource() {
        dataSources.remove();
    }
}
```

## 步骤2
实现AbstractRoutingDataSource
```
public class DynamicDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceHolder.getDataSource();
    }
}
```
## 步骤3
配置 spring 文件
```
<bean id="adamDataSource1" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
    <property name="url" value="jdbc:mysql://localhost:3306/am?useUnicode=true&amp;characterEncoding=utf8"/>
    <property name="username" value="root" />
    <property name="password" value="root" />
    <property name="filters" value="stat" />
    <property name="maxActive" value="500" />
    <property name="initialSize" value="10" />
    <property name="maxWait" value="60000" />
    <property name="minIdle" value="10" />
    <property name="timeBetweenEvictionRunsMillis" value="60000" />
    <property name="minEvictableIdleTimeMillis" value="300000" />
    <property name="validationQuery" value="SELECT 'x'" />
    <property name="testWhileIdle" value="true" />
    <property name="testOnBorrow" value="false" />
    <property name="testOnReturn" value="false" />
</bean>
<bean id="adamDataSource2" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
    <property name="url" value="jdbc:mysql://localhost:3306/am2?useUnicode=true&amp;characterEncoding=utf8"/>
    <property name="username" value="root" />
    <property name="password" value="root" />
    <property name="filters" value="stat" />
    <property name="maxActive" value="500" />
    <property name="initialSize" value="10" />
    <property name="maxWait" value="60000" />
    <property name="minIdle" value="10" />
    <property name="timeBetweenEvictionRunsMillis" value="60000" />
    <property name="minEvictableIdleTimeMillis" value="300000" />
    <property name="validationQuery" value="SELECT 'x'" />
    <property name="testWhileIdle" value="true" />
    <property name="testOnBorrow" value="false" />
    <property name="testOnReturn" value="false" />
</bean>

<bean id="dataSource" class="com.guall.dao.DynamicDataSource">
    <property name="targetDataSources">
    <map key-type="java.lang.String">
       <entry key="adamDataSource1" value-ref="adamDataSource1"></entry>
       <entry key="adamDataSource2" value-ref="adamDataSource2"></entry>
    </map>
    </property>
    <!-- 默认目标数据源为你主库数据源 -->
    <property name="defaultTargetDataSource" ref="adamDataSource"/>
</bean>
<bean id="adamJdbcTemplate" class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
    <constructor-arg name="dataSource" ref="dataSource"/>
</bean>
<bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
            <prop key="hibernate.hbm2ddl.auto">update</prop>
            <prop key="hibernate.show_sql">true</prop>
            <prop key="hibernate.format_sql">true</prop>
        </props>
    </property>
</bean>
```
## 步骤4
前三步已经完成了所有的准备工作，剩下的就是在采取不同的方式，根据需要来切换数据源了。可以用到的方式，如 Filter、Interceptor、AOP 等。

# 高级用法
不预先配置多种数据源，通过重写determineTargetDataSource方法，按照自己的方式进行数据源的创建和获取操作。
