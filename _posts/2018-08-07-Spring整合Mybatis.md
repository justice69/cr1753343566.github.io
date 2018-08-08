---
layout: post                  
title: "Spring整合Mybatis"             
date: 2018-08-07               
tag:  ssm框架
---
## Spring整合Mybatis的俩种数据库操作方式

1. 使用SqlSessionTemplate实现对数据库的操作
2. 使用MapperFactoryBean实现对数据库的操作

不管使用以上哪种方式，我们都要在Spring的配置文件种定义 ***SqlSessionFactoryBean*** 类为整合应用提供SqlSession对象资源

配置数据源我们这里采用dbcp连接池作为数据源，配置xml文件如下：

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/smbms?
                        useUnicode=true&amp;characterEncoding=utf-8"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>

接下来我们配置SqlSessionFactoryBean，它的作用就是为整合应用提供SqlSession对象资源

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--引用数据源组件-->
        <property name="dataSource" ref="dataSource"></property>
        <!--引用mybatis-config配置文件中的配置-->
        <!--<property name="configLocation" value="classpath:mybatis-config.xml"></property>-->
        <property name="typeAliasesPackage" value="com.ssm.pojo"></property>
        <!--配置SQL映射文件-->
        <property name="mapperLocations">
            <list>
                <value>classpath:com/ssm/dao/**/*.xml</value>
            </list>
        </property>
    </bean>

在上面的代码中，引用mybatis-config配置文件中的配置,配置信息就是给实体类设置别名，也就是将它的别名设置为类名，省去前面的包名，这个操作可以在上面的代码中直接进行配置，也就是

    <property name="typeAliasesPackage" value="com.ssm.pojo"></property>

这行代码可以替代mybatis-config.xml文件中的配置信息，所谓Spring整合Mybatis，就是接管mybatis的SqlSession，在使用SqlSessionTemplate管理mybaatis的SqlSession的时候，配置SQL映射文件是必不可少的一步，它将负责调用SQL映射语句，实现对数据库的访问

- 使用SqlSessionTemplate整合mybatis

配置SqlSessionTemplate

    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"></constructor-arg>
    </bean>

通过SqlSessionTemplate的构造器注入SqlSessionFactory

接下来就要配置Mapper，在Mapper.xml文件中写好SQL语句，并在Spring的配置文件中配置好Dao和service

    <!-- 配置Dao -->
    <bean id="userMapper" class="cn.smbms.dao.user.UserMapperImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>
    <!-- 配置业务Bean -->
    <bean id="userService" class="cn.smbms.service.user.UserServiceImpl">
        <property name="userMapper" ref="userMapper" />
    </bean>

配置SqlSessionTemplate是为了注入Dao的实现类去调用getMapper方法，返回数据访问结果，配置Dao是为了注入给Service，service的实现类通过它的set方法，来得到Dao，调用Dao实现类的方法，传入参数，得到数据，也就是得到Dao的实现类的返回结果，接下来介绍通过 ***使用MapperFactoryBean实现整合*** 

- 使用MapperFactoryBean实现整合

采用数据映射器（MapperFactoryBean）的方法完成对数据库的操作，首先对Spring进行配置

根据Mapper接口获取Mapper对象，它封装了原有的SqlSession.getMapper()功能的实现，也就是说，我们可以跳过dao的实现类去调用getMapper这一步，直接在service中注入dao对象。

    <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="mapperInterface" value="com.ssm.dao.user.UserMapper"></property>
        <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
    </bean>

它指定了映射器只能为接口类型，并注入SqlSessionFactory以提供SqlSessionTemplate实例，在Service的实现类中通过set方法注入dao，进行操作

通过数映射器的方式完成对数据库的操作可以在配置文件中省略SQL映射文件的配置，因为映射器对应的SQL映射文件与映射器的类路径相同，可以被MapperFactoryBean自动解析，但是如果映射器很多，响应的配置项就会很多，这时候我们可以通过MapperScannerConfiguer来简化配置项

MapperScannerConfiguer会自动扫描包下的Mapper接口，并将它们直接注册为MapperFactoryBean,配置如下：

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.ssm.dao"></property>
    </bean>
    <context:component-scan base-package="com.ssm.service"></context:component-scan>

配置好之后，操作就简单多了，直接在Service的实现类中定义dao的实现类，通过@Autowired自动注入，最大限度的减少了DAO组件与业务代码的编码和配置工作，接下来在dao包下进行功能的增加，直接在Service中通过属性@Autowired的方式进行注入，一次配置，将整个包下的配置简化

我们来对三种方式进行对比，看怎么一步一步的实现代码的不断简化

**使用SqlSessionTemplate实现整合**

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <!-- <property name="url">
            <value><![CDATA[jdbc:mysql://127.0.0.1:3306/smbms?
                    useUnicode=true&characterEncoding=utf-8]]></value>
        </property> -->
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/smbms?
                        useUnicode=true&amp;characterEncoding=utf-8" />
        <property name="username" value="root" />
        <property name="password" value="root" />
    </bean>
    <!-- 配置SqlSessionFactoryBean -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 引用数据源组件 -->
        <property name="dataSource" ref="dataSource" />
        <!-- 引用MyBatis配置文件中的配置 -->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <!-- 配置SQL映射文件信息 -->
        <property name="mapperLocations">
            <list>
                <value>classpath:cn/smbms/dao/**/*.xml</value>
            </list>
        </property>
    </bean>
    <!-- 配置DAO -->
    <bean id="userMapper" class="cn.smbms.dao.user.UserMapperImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>
    <!-- 配置业务Bean -->
    <bean id="userService" class="cn.smbms.service.user.UserServiceImpl">
        <property name="userMapper" ref="userMapper" />
    </bean>


**使用MapperFactoryBean实现整合**

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <!-- <property name="url">
            <value><![CDATA[jdbc:mysql://127.0.0.1:3306/smbms?
                    useUnicode=true&characterEncoding=utf-8]]></value>
        </property> -->
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/smbms?
                        useUnicode=true&amp;characterEncoding=utf-8" />
        <property name="username" value="root" />
        <property name="password" value="root" />
    </bean>

    <!-- 配置SqlSessionFactoryBean -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 引用数据源组件 -->
        <property name="dataSource" ref="dataSource" />
        <!-- 引用MyBatis配置文件中的配置 -->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <!-- 配置SQL映射文件信息 -->
        <!-- <property name="mapperLocations">
            <list>
                <value>classpath:cn/smbms/dao/**/*.xml</value>
            </list>
        </property> -->
    </bean>
    <!-- 配置DAO -->
    <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="mapperInterface" value="cn.smbms.dao.user.UserMapper" />
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    </bean>
    <!-- 配置业务Bean -->
    <bean id="userService" class="cn.smbms.service.user.UserServiceImpl">
        <property name="userMapper" ref="userMapper" />
    </bean>

**配置MapperCongiguer简化**

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <!-- <property name="url">
            <value><![CDATA[jdbc:mysql://127.0.0.1:3306/smbms?
                    useUnicode=true&characterEncoding=utf-8]]></value>
        </property> -->
        <property name="url" value="jdbc:mysql://127.0.0.1:3306/smbms?
                        useUnicode=true&amp;characterEncoding=utf-8" />
        <property name="username" value="root" />
        <property name="password" value="root" />
    </bean>

    <!-- 配置SqlSessionFactoryBean -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 引用数据源组件 -->
        <property name="dataSource" ref="dataSource" />
        <!-- 引用MyBatis配置文件中的配置 -->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
    </bean>
    <!-- 配置DAO -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.smbms.dao" />
    </bean>
    <context:component-scan base-package="cn.smbms.service" />
    

进化就是一步步简化代码，从SQL映射文件的配置和dao实现类中进行getMapper方法简化到使用数据映射器，根据Mapper接口获取Mapper对象，封装了原有的SqlSession.getMapper()功能的实现，从而省去了dao实现类，再加入MapperScannerConfiguer来简化配置项，自动扫描包下的Mapper接口并自动注册为MapperFactoryBean在Service中直接@Autowired注入。




