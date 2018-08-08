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


<table>
<tr style="text-align:center">
<td>使用SqlSessionTemplate实现整合</td>
<td>使用MapperFactoryBean实现整合</td>
<td>配置MapperCongiguer简化</td>
</tr>
<td>

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
</td>
<td>

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
</td>
<td>

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
</td>
<tr>
</table>

所以进化的操作就在与从SQL映射文件的配置和dao实现类中进行getMapper方法简化使用数据映射器，根据Mapper接口获取Mapper对象，封装了原有的SqlSession.getMapper()功能的实现，从而省去了dao实现类，再加入MapperScannerConfiguer来简化配置项，自动扫描包下的Mapper接口并自动注册为MapperFactoryBean在Service中直接@Autowired注入。

Spring实现事务管理：对mybatis操作数据库进行事务控制，spring使用jdbc的事务控制类

事物应该在业务逻辑层控制，Spring通过AOP的方式实现实现，Spring提供了声明式事物支持

声明式事物-对什么方法，采用什么样的事物策略

配置步骤：

1. 导入tx和aop命名空间
2. 定义事务管理器bean，并为其注入数据源Bean
3. 通过<tx:advice>配置事务增强，绑定事务管理器并针对不同方法定义事物规则
4. 配置切面，将事务增强与方法切入点结合

在Spring的配置文件中进行配置：

     <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <tx:advice id="txAdvice">
        <tx:attributes>
            <tx:method name="find*" propagation="SUPPORTS" />
            <tx:method name="add*" propagation="REQUIRED" />
            <tx:method name="del*" propagation="REQUIRED" />
            <tx:method name="update*" propagation="REQUIRED" />
            <tx:method name="*" propagation="REQUIRED" />
        </tx:attributes>
    </tx:advice>
    <!-- 定义切面 -->
    <aop:config>
        <aop:pointcut id="serviceMethod"
            expression="execution(* cn.smbms.service..*.*(..))" />
        <aop:advisor advice-ref="txAdvice" pointcut-ref="serviceMethod" />
    </aop:config>

这样就将事务切入到了service包下的所有方法中，不同的操作通过propagation进行不同的事物控制，当然可以通过注解的方式灵活的在service层进行事物管理，在spring的配置文件中修改配置如下，通过@进行配置

     <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <tx:annotation-driven />

在service中，将需要进行事物管理的service的实现类注入@Transactional并在具体要进行事务管理的地方设置其事务，如：

    @Transactional
    @Service("userService")
    public class UserServiceImpl implements UserService {
    @Autowired // @Resource
    private UserMapper userMapper;

    @Override
    @Transactional(propagation = Propagation.SUPPORTS)
    public List<User> findUsersWithConditions(User user) {
        try {
            return userMapper.getUserList(user);
        } catch (RuntimeException e) {
            e.printStackTrace();
            throw e;
        }
    }

- 数据库事务：

    数据库事务满足的特性：原子性、一致性、隔离性、持久性

1. 原子性：表示组成一个事务的多个数据库操作是一个不可分割的源自1单元，只有所有操作执行成功，整个事务才提交，任何一个数据库操作失败，已经执行的任何操作都撤销，让数据库返回初始状态
2. 一致性：事务操作成功后，数据库所处的状态和他的业务规则是一致的，即数据不会被破坏，拿银行转账来说，如果A向B转账100，不管操作成功与否，A账户和B账户的存款总额不变
3. 隔离性：在并发数据操作时，不同的事物有各自的数据空间，他们的操作不会对对方产生干扰，数据库规定了多种事物隔离级别，不同的隔离级别对应不同的干扰程度，隔离级别越高，数据一致性越好，但并发性越弱
4. 持久性：一旦事物提交后，事务中所有数据操作都必须被持久化到数据库中，即使在提交事务后，数据库马上崩溃，在数据库重启时，也必须保证能够通过某种机制恢复数据

事务的一致性是最终目标，其他特性都是为达到一致性而采取的措施、要求或手段

数据库管理系统采用数据库锁的机制保证事务的隔离性，当多个事务试图对相同的数据进行操作时，只有持有锁的事务才能操作数据，其他事务等到事务完成后，才能对数据库进行操作，Oracle数据库采用数据版本的机制，在回滚段为数据的每个变化都保存一个版本，使数据的更改不影响数据的读取

- 数据并发问题

1. 脏读：A事务读取B事务尚未提交的更改数据，并在这个数据基础上进行操作。如果恰巧B事务回滚，那么A事务读到的数据根本是不被承认的，A事务读到的数据也称为 **脏读** Oracle数据库不会发生脏读的情况
2. 不可重复读： A事务读取了B事务已经提交的更改数据，如在银行业务中，A在某个时间段查询余额为1000元，B事务随后取出100，余额变为900，A事务再次查询余额，发现查询到的余额不一致
3. 幻象读：A事务读取B事务提交的新增数据，这时A事务将出现幻象读的问题，这个问题主要发生在统计数据的事务中，如银行在一个事务中俩次统计存款账户的总金额，在俩次统计过程中，刚好增加了一个存款账户，并存入100，这时，俩次统计的总金额将不一致
4. 第一类丢失更新：A事务撤销时，把已经提交的B事务的更新数据覆盖了 如A事务查询账户余额为1000元，B事务查询账户余额为1000元，然后B事务汇入100元，把余额改为了1100元，并提交了事务，此时事务A进行取款，取出100，将把A事务查到的1000元减去100元，余额改为900元，然后A事务撤销，余额将恢复到1000元，就把B事务的更新丢失。
5. 第二类丢失更新：事务A覆盖事务B已经提交的数据，造成B事务所做操作丢失 如事务A查询账户余额1000元，B查询账户余额1000元，然后事务B取出100元，把余额改为了900元并提交，此时事务A汇入100元，并提交，数据库会把余额改为1100元，此时B事务的操作丢失

数据并发产生的问题通过锁机制解决

按照锁定的对象的不同，一般可以分为表锁定和行锁定，表锁定对整张表进行锁定，而行锁定对特定的某一行进行锁定，从并发事务锁定的关系上来看，可以分为共享锁定和独占锁定，共享锁定会防止独占锁定，但允许其他的共享锁定，而独占锁定既防止独占锁定，也防止共享锁定

为了更改数据，数据库必须在进行更改的行上施加行独占锁定，

事务隔离级别：用户制定会话的隔离级别，数据库就会分析事务中的SQL语句，然后自动为事务操作的数据资源添加适合的锁，数据库还会维护这些锁

<p><img src="/images/Blog/shiwu.PNG" ></p>

***事务隔离级别越高数据库并发性越差***

当我们整合Spring和Mybatis的时候，我们使用的事务管理器为

    org.springframework.jdbc.datasource.DataSourceTransactionManager

它是PlatformTransactionManager接口的实现类，当采用不同的框架进行持久化的时候，它提供不同的事物管理器

事物传播行为：当我们调用一个基于Spring的service接口方法时，它将运行于Spring管理的事务环境中，Service接口方法可能会在内部调用其他的Service接口方法以共同完成一个完整的业务操作，因此就会产生服务器接口方法嵌套调用的情况，Spring通过事物传播行为控制当前的事物如何传播到被嵌套调用的目标服务接口方法中

<p><img src="/images/Blog/chuanbo.PNG" ></p>

TransactionDefinition定义了Spring兼容的事务属性，这些属性对事物管理控制的若干方面进行配置

1. propagation：事物传播机制  

    可选 REQUIRED（默认值）
    REQUIRES_NEW 、MANDATORY、NESTED 
    SUPPORTS
    NOT_SUPPORTED、NEVER

2. isolation：事务隔离等级

    DEFAULT（默认值）
    READ_COMMITTED
    READ_UNCOMMITTED
    REPEATABLE_READ
    SERIALIZABLE

3. timeout:事务超时时间，允许事务运行的最长时间，以秒为单位。默认值为-1，表示不超时

4. read-only：事务是否为只读，默认值为false 可以用在查询操作

5. rollback-for：设定能够触发回滚的异常类型

    Spring默认只在抛出runtime exception时才标识事务回滚
    可以通过全限定类名指定需要回滚事务的异常，多个类名用逗号隔开

6. no-rollback-for：设定不触发回滚的异常类型

    Spring默认checked Exception不会触发事务回滚
    可以通过全限定类名指定不需回滚事务的异常，多个类名用英文逗号隔开

在使用注解实现事务支持的时候，在要进行事务管理的方法前使用 ***@Transactional***为方法添加事务支持如

    @Transactional(propagation = Propagation.SUPPORTS)
    public List<User> findUsersWithConditions(User user) {
        // 省略实现代码
    }

在括号中进行属性赋值，来进行事务管理

- 拓展

    死锁：在数据库中有两种基本的锁类型：排它锁（Exclusive Locks，即X锁）和共享锁（Share Locks，即S锁）。当数据对象被加上排它锁时，其他的事务不能对它读取和修改。加了共享锁的数据对象可以被其他事务读取，但不能修改。数据库利用这两种基本的锁类型来对数据库的事务进行并发控制，当出现如下情况时称为死锁：

1. 事务之间对资源访问顺序的交替

    一个用户A 访问表A（锁住了表A），然后又访问表B；另一个用户B 访问表B（锁住了表B），然后企图访问表A；这时用户A由于用户B已经锁住表B，它必须等待用户B释放表B才能继续，同样用户B要等用户A释放表A才能继续，这就死锁就产生了。

    **解决办法：** 是由于程序的BUG产生的，除了调整的程序的逻辑没有其它的办法。仔细分析程序的逻辑，对于数据库的多表操作时，尽量按照相同的顺序进行处理，尽量避免同时锁定两个资源，如操作A和B两张表时，总是按先A后B的顺序处理， 必须同时锁定两个资源时，要保证在任何时刻都应该按照相同的顺序来锁定资源

2. 并发修改同一记录

    用户A查询一条纪录，然后修改该条纪录；这时用户B修改该条纪录，这时用户A的事务里锁的性质由查询的共享锁企图上升到独占锁，而用户B里的独占锁由于A有共享锁存在所以必须等A释放掉共享锁，而A由于B的独占锁而无法上升的独占锁也就不可能释放共享锁，于是出现了死锁。这种死锁由于比较隐蔽，但在稍大点的项目中经常发生。

    一般更新模式由一个事务组成，此事务读取记录，获取资源（页或行）的共享 (S) 锁，然后修改行，此操作要求锁转换为排它 (X) 锁。如果两个事务获得了资源上的 共享模式锁，然后试图同时更新数据，则一个事务尝试将锁转换为排它 (X) 锁。共享模式到排它锁的转换必须等待一段时间，因为一个事务的排它锁与其它事务的共享模式锁不兼容；发生锁等待。第二个事务试图获取排它 (X) 锁以进行更新。由于两个事务都要转换为排它 (X) 锁，并且每个事务都等待另一个事务释放共享模式锁，因此发生死锁。

    **解决方法：** 
    1. 使用乐观锁进行控制。乐观锁大多是基于数据版本（Version）记录机制实现。即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个“version”字段来实现。读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。乐观锁机制避免了长事务中的数据库加锁开销（用户A和用户B操作过程中，都没有对数据库数据加锁），大大提升了大并发量下的系统整体性能表现。Hibernate 在其数据访问引擎中内置了乐观锁实现。需要注意的是，由于乐观锁机制是在我们的系统中实现，来自外部系统的用户更新操作不受我们系统的控制，因此可能会造成脏数据被更新到数据库中。 
    2. 使用悲观锁进行控制。悲观锁大多数情况下依靠数据库的锁机制实现，如Oracle的Select … for update语句，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销，特别是对长事务而言，这样的开销往往无法承受。如一个金融系统，当某个操作员读取用户的数据，并在读出的用户数据的基础上进行修改时（如更改用户账户余额），如果采用悲观锁机制，也就意味着整个操作过程中（从操作员读出数据、开始修改直至提交修改结果的全过程，甚至还包括操作员中途去煮咖啡的时间），数据库记录始终处于加锁状态，可以想见，如果面对成百上千个并发，这样的情况将导致灾难性的后果。所以，采用悲观锁进行控制时一定要考虑清楚。 

