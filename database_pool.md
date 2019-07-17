# 1.c3p0简介
c3p0被设计成易于使用。只需要在项目中导入相关jar包即可,导入相关包时，需要注意，包的版本的及依赖包的版本
```
<dependencies>
  <!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
	<dependency>
	    <groupId>com.mchange</groupId>
	    <artifactId>c3p0</artifactId>
	    <version>0.9.5.4</version>
	</dependency>
	<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
	<dependency>
	    <groupId>mysql</groupId>
	    <artifactId>mysql-connector-java</artifactId>
	    <version>5.1.6</version>
	</dependency>
  </dependencies>
```
即可进行简单的数据库操作
```
ComboPooledDataSource cpds = new ComboPooledDataSource();
cpds.setDriverClass( "org.postgresql.Driver" ); //loads the jdbc driver            
cpds.setJdbcUrl( "jdbc:postgresql://localhost/testdb" );
cpds.setUser("dbuser");                                  
cpds.setPassword("dbpassword"); 
```
如果需要打开PreparedStatement池，需要设置maxStatements、 maxStatementsPerConnection ，它们的默认值都是0.
`cpds.setMaxStatements( 180 );   `
使用完成后，清理
`cpds.close();`就是这样，剩下的就是细节了。
# 2.c3p0使用
获取c3p0池支持的DataSource有三种方法：
- 直接实例化和配置一个ComboPooledDataSource实例。(推荐使用此方法)
- 使用DataSources工厂类
- 通过直接实例化PoolBackedDataSource并设置其ConectionPoolDataSource来“构建您自己的”池支持的DataSource

大多数用户可能会发现实例化ComboPooledDataSource是最方便的方法。 实例化后，c3p0 DataSources几乎可以绑定到任何符合JNDI的名称服务。

实例化和配置一个ComboPooledDataSource
```
ds = new ComboPooledDataSource();
//			ds.setDriverClass(env.getProperty("jdbc.driver"));
//			ds.setJdbcUrl(url);
//			ds.setUser(username);
//			ds.setPassword(password);
//			ds.setMaxPoolSize(200);
//			ds.setMinPoolSize(10);
//			ds.setInitialPoolSize(50);
//			ds.setMaxStatements(180);
```

# 3. c3p0 PooledDataSources的清理工作
```
 DataSources.destroy( ds_pooled );
```
或者，c3p0的PooledDataSource接口包含一个close（）方法，当您知道已完成DataSource时可以调用该方法
# 4.c3p0配置
虽然c3p0不需要很多配置，但它是非常可调整的。有几种方法可以修改c3p0属性：您可以直接更改代码中与特定DataSource关联的属性值，也可以从外部配置c3p0
- 通过一个简单的Java属性文件
- 通过XML配置文件
数据源通常在使用之前进行配置，或者在构建过程中进行配置，或者在构建之后立即进行配置。然而，c3p0确实支持中途修改属性.如果通过实例化ComboPooledDataSource获取DataSource，则在尝试调用getConnection（）之前，通过调用该类提供的适当setter方法来配置它。 请参阅上面的示例。如果你通过使用`com.mchange.v2.c3g0.DataSources`类中的工厂方法获取DataSources, 并且不想使用默认的配置，你可以提供一个带有属性键值对的Map.
# 5.基本的池配置
- acquireIncrement
- initialPoolSize
- maxPoolSize 
- maxIdleTime 
- minPoolSize
nitialPoolSize，minPoolSize，maxPoolSize定义将池的Connections数。 请确保minPoolSize <= maxPoolSize。 将忽略initialPoolSize的不合理值，而使用minPoolSize。在minPoolSize和maxPoolSize之间的范围内，池中的连接数根据使用模式而有所不同.每当用户请求连接，没有可用的连接，并且池中的管理连接数尚未达到maxPoolSize时，连接数就会增加.由于连接获取速度非常慢，因此更早的增加连接数量几乎总是有用的，而不是强迫每个客户端等待新连接在负载增加时引发单个获取。acquireIncrement确定当池用完连接时c3p0池将尝试获取的连接数。无论 acquireIncrement设置多少，池永远都不会允许超过  maxPoolSize。
# 6.管理池大小和连接时间
不同的应用程序在性能，占用空间和可靠性之间的权衡方面有不同的需求。C3P0提供了多种选项，用于控制在负载下变大的池恢复到minPoolSize的速度，以及是否应主动替换池中的“旧”连接以保持其可靠性。
- maxConnectionAge 
- maxIdleTime 
- maxIdleTimeExcessConnections 
默认情况下，池永远不会到期连接。 如果您希望Connections随着时间的推移而过期，以保持“新鲜度”，设置maxIdleTime and/or maxConnectionAge. maxIdleTime定义在从池中删除连接之前，应该允许连接闲置多长时间。maxConnectionAge 强制池剔除过去从数据库中获取的超过设置秒数的任何连接。maxIdleTimeExcessConnections 是关于在池未加载时最小化c3p0池所持有的连接数。默认情况下，c3p0池在负载下会增长，但只有在连接未通过连接测试或通过上述参数过期时才会收缩。一些用户希望他们的池在使用高峰后快速释放不必要的连接。关于所有这些超时参数的一些一般建议：慢下来！ 连接池的要点是承担仅获取一次Connection的成本，然后多次重复使用Connection。 大多数数据库支持一次保持打开数小时的连接。 每隔几秒钟或几分钟就无需流失所有连接。 将maxConnectionAge或maxIdleTime设置为1800（30分钟）非常激进。 对于大多数数据库，几个小时可能更合适。 您可以通过测试来确保Connections的可靠性，而不是通过抛弃它们来确保它们的可靠性。 （请参阅配置连接测试。）通常应设置为几分钟或更短时间的这些参数中唯一一个是maxIdleTimeExcessConnections。
# 7.hibernate c3p0
导入相关包，注意包的版本问题，否则无法支持相关功能：
```
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-c3p0 -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-c3p0</artifactId>
    <version>5.3.3.Final</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-core -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.3.3.Final</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.jboss.logging/jboss-logging -->
<dependency>
    <groupId>org.jboss.logging</groupId>
    <artifactId>jboss-logging</artifactId>
    <version>3.3.2.Final</version>
</dependency>

```
其相关依赖包都要相应导入，导入时需注意包的版本依赖
![image.png](https://upload-images.jianshu.io/upload_images/3004516-43fadab7a86c3f9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
相关包导入后，即可在项目中，进行相关配置，来使用数据库连接池的功能了
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
   "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
   "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
	<session-factory>
		<property name="dialect">org.hibernate.dialect.Oracle9iDialect</property>
		<property name="connection.driver_class">oracle.jdbc.driver.OracleDriver</property>
		<property name="connection.url">jdbc:oracle:thin:@127.0.0.1:1521:orcl</property>
		<property name="connection.username">username</property>
		<property name="connection.password">password</property>
		<property name="format_sql">false</property>
		<property name="show_sql">false</property>
		<property name="hbm2ddl.auto">update</property>
		<property name="c3p0.min_size">5</property>
		<property name="c3p0.max_size">30</property>
		<property name="c3p0.timeout">120</property>
		<property name="c3p0.idle_test_period">3000</property>
	</session-factory>
</hibernate-configuration>
```