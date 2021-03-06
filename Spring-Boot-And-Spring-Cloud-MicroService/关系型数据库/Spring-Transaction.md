﻿



# Transaction
事务是数据库开发中常见的操作之一，根据数据库的基本理论我们可知事务具有以下四个特性：
- 原子性：一个事务中所有对数据库的操作是一个不可分割的操作序列，要么全做，要么全部做。
- 一致性：数据不会因为事务的执行而遭到破坏。
- 隔离性：一个事务的执行，不受其他事务（进程）的干扰。既并发执行的个事务之间互不干扰。
- 持久性：一个事务一旦提交，它对数据库的改变将是永久的。


编程式事务管理&声明式事务管理:
（一）编程式事务管理 在 Spring 出现以前，编程式事务管理对基于 POJO 的应用来说是唯一选择。用过 Hibernate 
的人都知道，我们需要在代码中显式调用beginTransaction()、commit()、rollback()等事务管理相关的方法，这就是编程
式事务管理。通过 Spring 提供的事务管理 API，我们可以在代码中灵活控制事务的执行。在底层，Spring 
仍然将事务操作委托给底层的持久化框架来执行。
（二）声明式事务管理
1）Spring 的声明式事务管理在底层是建立在 AOP 的基础之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。
2）声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务
规则声明（或通过等价的基于标注的方式），便可以将事务规则应用到业务逻辑中。因为事务管理本身就是一个典型的横切逻辑，正是 AOP 
的用武之地。Spring 开发团队也意识到了这一点，为声明式事务提供了简单而强大的支持。
3）声明式事务管理曾经是 EJB 引以为傲的一个亮点， Spring 让 POJO 在事务管理方面也拥有了和 EJB 
一样的待遇，让开发人员在 EJB 容器之外也用上了强大的声明式事务管理功能，这主要得益于 Spring 依赖注入容器和 Spring AOP 
的支持。依赖注入容器为声明式事务管理提供了基础设施，使得 Bean 对于 Spring 框架而言是可管理的；而 Spring AOP 
则是声明式事务管理的直接实现者。

4）建议在开发中使用声明式事务，不仅因为其简单，更主要是因为这样使得纯业务代码不被污染，极大方便后期的代码维护。和编程式事务相比，声明
式事务唯一不足地方是，后者的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方
法，比如，可以将需要进行事务管理的代码块独立为方法等等。

从@Transactional的代码当中可以看到，可配置的参数主要有：

事务隔离级别:      Isolation isolation() default Isolation.DEFAULT;
事务传播行为:      Propagation propagation() default Propagation.REQUIRED;
回滚、不回滚的异常类:      rollbackFor() noRollbackFor()
超时:      int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;


目前Spring中提供的事务实现方式有两种：编码方式与声明式事务管理方式。基于AOP技术实现的声明式事务管理，实质就是：在方法执行前后进行拦截，然后在目标方法开始之前创建并加入事务，执行完目标方法后根据执行情况提交或回滚事务。声明式事务管理又有两种方式：一种是基于XML配置文件的方式；另一种是在业务方法上添加@Transactional注解，将事务规则应用到业务逻辑中（一般定义在Service层）。Spring事务的抽象的关键是所谓的*事务策略*，一个事务策略的抽象定义在`org.springframework.transaction.PlatformTransactionManager`接口中：
```
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(
            TransactionDefinition definition) throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```


## Reference



- [使用spring中的@Transactional注解时，可能需要注意的地方](http://www.tuicool.com/articles/AjiInmq)




# 事务的传播级别


事务的传播行为: 所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：


1、TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。


2、TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。


3、TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。


4、TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。


5、TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。


6、TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。


7、TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。


这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。而 PROPAGATION_NESTED是 Spring 所特有的。以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。如果熟悉 JDBC 中的保存点（SavePoint）的概念，那嵌套事务就很容易理解了，其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。


是否需要创建事务，是由事务传播行为控制的。读数据不需要或只为其指定只读事务，而数据的插入、修改、删除就需要进行事务管理了，这是由事务的隔离级别控制的。事务具有7个传播级别和4个隔离级别，传播级别定义的是事务创建的时机，隔离级别定义的是对并发事务数据读取的控制。
- PROPAGATION_REQUIRED:默认的Spring事务传播级别，使用该级别的特点是：如果上下文中已经存在事务，那么就加入到事务中执行；如果当前上下文中不存在事务，则新建事务执行。所以这个级别通常能满足处理大多数的业务场景。


- PROPAGATION_SUPPORTS:从字面意思就知道，supports，支持，该传播级别的特点是：如果上下文存在事务，则支持事务加入事务，如果没有事务，则使用非事务的方式执行。所以
说，并非所有的包含在TransactionTemplate.execute方法中的代码都会有事务支持。这个通常是用来处理那些并非原子性的非核心业
务逻辑操作。应用场景较少。



- PROPAGATION_MANDATORY:该级别的事务要求上下文中必须要存在事务，否则就会抛出异常！配置该方式的传播级别是有效的控制上下文调用代码遗漏添加事务控制的保证手段。比如一段代码不能单独被调用执行，但是一旦被调用，就必须有事务包含的情况，就可以使用这个传播级别。



- PROPAGATION_REQUIRES_NEW:从字面即可知道，new，每次都要一个新事务，该传播级别的特点是：每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。那么当执行到ServiceB.methodB的时候，ServiceA.methodA所在的事务就会挂起，ServiceB.methodB会起一个新的事务，等待ServiceB.methodB的事务完成以后， 他才继续执行。他与PROPAGATION_REQUIRED 的事务区别在于事务的回滚程度了。因为ServiceB.methodB是新起一个事务，那么就是存在两个不同的事务。如果ServiceB.methodB已经提交，那么ServiceA.methodA失败回滚，ServiceB.methodB是不会回滚的。如果ServiceB.methodB失败回滚， 如果他抛出的异常被ServiceA.methodA捕获，ServiceA.methodA事务仍然可能提交。这是一个很有用的传播级别，举一个应用场景：现在有一个发送100个红包的操作，在发送之前，要做一些系统的初始化、验证、数据记录操作，然后发送100封红包，然后再记录发送日志，发送日志要求100%的准确，如果日志不准确，那么整个父事务逻辑需要回滚。怎么处理整个业务需求呢？就是通过这个PROPAGATION_REQUIRES_NEW级别的事务传播控制就可以完成。发送红包的子事务不会直接影响到父事务的提交和回滚。


- PROPAGATION_NOT_SUPPORTED:这个也可以从字面得知，not supported，不支持，当前级别的特点是：若上下文中存在事务，则挂起事务，执行当前逻辑，结束后恢复上下文的事务。这个级别有什么好处？可以帮助你将事务尽可能的缩小。我们知道一个事务越大，它存在的风险也就越多。所以在处理事务的过程中，要保证尽可能的缩小范
围。比如一段代码，是每次逻辑操作都必须调用的，比如循环1000次的某个非核心业务逻辑操作。这样的代码如果包在事务中，势必造成事务太大，导致出现一
些难以考虑周全的异常情况，所以事务的这个传播级别就派上用场了。用当前级别的事务模板包含起来就可以了。


- PROPAGATION_NEVER:该事务更严格，上面一个事务传播级别只是不支持而已，有事务就挂起，而PROPAGATION_NEVER传播级别要求上下文中不能存在事务，一旦有事务，就抛出runtime异常，强制停止执行！这个级别上辈子跟事务有仇。



- PROPAGATION_NESTED:如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。从字面也可知道，nested，嵌套级别事务。该传播级别的特征是：如果上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务。那么什么是嵌套事务呢？很多人都不理解，我看过一些博客，都是有些理解偏差。
嵌套是子事务嵌套在父事务中执行，子事务是父事务的一部分，在进入子事务之前，父事务建立一个回滚点，叫save point，然后执行子事务，这个子事务的执行也算是父事务的一部分，然后子事务执行结束，父事务继续执行。重点就在于那个save point。看几个问题就明了了：
   - 如果子事务回滚，会发生什么？
父事务会回滚到进入子事务前建立的save point，然后尝试其他的事务或者其他的业务逻辑，父事务之前的操作不会受到影响，更不会自动回滚。
   - 如果父事务回滚，会发生什么？

父事务回滚，子事务也会跟着回滚！为什么呢，因为父事务结束之前，子事务是不会提交的，我们说子事务是父事务的一部分，正是这个道理。
   - 事务的提交，是什么情况？

是父事务先提交，然后子事务提交，还是子事务先提交，父事务再提交？答案是第二种情况，还是那句话，子事务是父事务的一部分，由父事务统一提交。





### Required
![](http://docs.spring.io/autorepo/docs/spring/4.2.x/spring-framework-reference/html/images/tx_prop_required.png)




### RequiresNew
![](http://docs.spring.io/autorepo/docs/spring/4.2.x/spring-framework-reference/html/images/tx_prop_requires_new.png)


### Nested
PROPAGATION_NESTED uses a single physical transaction with multiple savepoints that it can roll back to. Such partial rollbacks allow an inner transaction scope to trigger a rollback for its scope, with the outer transaction being able to continue the physical transaction despite some operations having been rolled back. This setting is typically mapped onto JDBC savepoints, so will only work with JDBC resource transactions. See Spring’sDataSourceTransactionManager.


PROPAGATION_NESTED: 嵌套事务类型，是相对上面提到的六种情况（上面的六种应该称为平面事务类型），打个比方我现在有一个事务主要有一下几部分：
      1，从A用户帐户里面减去100元钱
      2，往B用户帐户里面添加100元钱
这样看和以前不同的事务可能没有什么区别，那我现在有点特殊的要求就是，A用户有3个帐户，B用户有2个帐户，现在我的要求就是只要再A用户的3个帐户里面任意一个减去100元，往B用户的两个帐户中任意一个里面增加100元就可以了！一旦你有这样的要求那嵌套事务类型就非常适合你！我们可以这样理解，
       一：将“从A用户帐户里面减去100元钱” 和 “往B用户帐户里面增加100元钱”我们暂时认为是一级事务操作
       二：将从A用户的3个帐户的任意一个帐户里面减钱看做是“从A用户帐户里面减去100元钱”这个一级事务的子事务（二级事务），同样把后面存钱的看成是另一个的二级事务。
      问题一：当二级事务被rollback一级事务会不会被rollback？
      答案是不会的，二级事务的rollback只针对自己。
      问题二：什么时候这个一级事务会commit，什么时候会被rollback呢？
      我们主要看二级里面出现的情况，当所有的二级事务被commit了并且一级事务没有失败的操作，那整个事务就算是一个成功的事务，这种情况整个事务会被commit。当任意一个二级事务没有被commit那整个事务就是失败的，整个事务会被roolback。还是拿上面的例子来说明吧！如果我在a的三个帐户里面减钱的操作都被二级事务给rollback了，也就是3个帐户里面都没有减钱成功，整个事务就失败了就会被rollback。如果A用户帐户三个帐户里面有一个可以扣钱而且B用户的两个帐户里面也有一个帐户可以增加钱，那整个事务就算成功的，会被 commit。
看了一下觉得上面的例子好像不是很深刻，看这个情况（A用户的3个帐户都是有信用额度的，也就是说可以超支，但是超支有金额限制）。


## 事务的隔离级别


数据隔离级别分为不同的4种：
（1）READ_UNCOMMITTED
保证了读取过程中不会读取到非法数据。

（2）READ_COMMITTED
大多数主流数据库的默认事务等级，保证了一个事务不会读到另一个并行事务已修改但未提交的数据，避免了“脏读取”。该级别适用于大多数系统。所谓的脏读，其实就是读到了别的事务回滚前的脏数据。比如事务B执行过程中修改了数据X，在未提交前，事务A读取了X，而事务B却回滚了，这样事务A就形成了脏读。

（3）REPEATABLE_READ
保证了一个事务不会修改已经由另一个事务读取但未提交（回滚）的数据。避免了“脏读取”和“不可重复读取”的情况，但是带来了更多的性能损失。不可重复读字面含义已经很明了了，比如事务A首先读取了一条数据，然后执行逻辑的时候，事务B将这条数据改变了，然后事务A再次读取的时候，发现数据不匹配了，就是所谓的不可重复读了。

（4）SERIALIZABLE
最严格的级别，事务串行执行，资源消耗最大。小的时候数手指，第一次数十10个，第二次数是11个，怎么回事？产生幻觉了？幻读也是这样子，事务A首先根据条件索引得到10条数据，然后事务B改变了数据库一条数据，导致也符合事务A当时的搜索条件，这样事务A再次搜索发现有11条数据了，就产生了幻读。


事务隔离级别对照关系表：
|                      | **脏读** | **不可重复读** | **幻读** |
| -------------------- | ------ | --------- | ------ |
| **SERIALIZABLE**     | 不会     | 不会        | 不会     |
| **REPEATABLE_READ**  | 不会     | 不会        | 会      |
| **READ_COMMITTED**   | 不会     | 会         | 会      |
| **READ_UNCOMMITTED** | 会      | 会         | 会      |


所以最安全的，是Serializable，但是伴随而来也是高昂的性能开销。另外，事务常用的两个属性：① **readonly**，设置事务为只读以提升性能；②**timeout**，设置事务的超时时间，一般用于防止大事务的发生。


## Reference

- [Spring的事务传播性与隔离级别 ](http://blog.csdn.net/yang1982_0907/article/details/44408809) 




# Declarative Transaction Management:声明式事务管理


![](http://docs.spring.io/autorepo/docs/spring/4.2.x/spring-framework-reference/html/images/tx.png)


声明式事务的首要操作就是添加`@EnableTransactionManagement`到配置类中，然后在需要实现事务控制的类上添加`@Transactional`注解。声明式事务主要是基于AOP代理实现，并且由定义在XML或者注解中的元数据所驱动。一个AOP和事务元信息的组合会使用`TransactionIntercetor`与`PlatformTransactionManager`的组合来实现在方法调用前后的事务控制。
首先我们定义一个接口及其实现：
```// the service interface that we want to make transactional

package x.y.service;

@Transactional
public interface FooService {

    Foo getFoo(String fooName);

    Foo getFoo(String fooName, String barName);

    void insertFoo(Foo foo);

    void updateFoo(Foo foo);

}//在实现的类中也可以设置对于父类或者父接口的Transactional的复写
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {

    public Foo getFoo(String fooName) {
        // do something
    }

    ...
    
    // these settings have precedence for this method
    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
    public void updateFoo(Foo foo) {
        // do something
    }
}


```
这里假设在FooService的get*方法中执行只读事务策略，即如果发现有写入或者删除直接回滚，而insertFoo与updateFoo方法则运行在读写策略的事务上下文中。基本的XML中的配置如下：
```
<!-- from the file 'context.xml' -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!-- this is the service object that we want to make transactional -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>


   <!--通用事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!-- 指定事务策略,声明一个通知，用以指出要管理哪些事务方法及如何管理 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- the transactional semantics... -->
        <tx:attributes>
            <!-- all methods starting with 'get' are read-only -->
            <tx:method name="get*" read-only="true"/>
            <!-- other methods use the default transaction settings (see below) -->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>


    <!-- 声明一个config，用以将事务策略和业务类关联起来-->
    <aop:config>
        <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
    </aop:config>


    <!-- don't forget the DataSource -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
        <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
        <property name="username" value="scott"/>
        <property name="password" value="tiger"/>
    </bean>




    <!-- other <bean/> definitions here -->


</beans>
```
这里tx:method的详细配置项如下：
| 属性              | 说明                                       |
| --------------- | ---------------------------------------- |
| name            | 方法名的匹配模式，通知根据该模式寻找匹配的方法。  该属性可以使用asterisk (*)通配符 |
| propagation     | 设定事务定义所用的传播级别                            |
| isolation       | 设定事务的隔离级别                                |
| timeout         | 指定事务的超时（单位为秒）                            |
| read-only       | 该属性为true指示事务是只读的（典型地，  对于只执行查询的事务你会将该属性设为true，  如果出现了更新、插入或是删除语句时只读事务就会失败） |
| no-rollback-for | 以逗号分隔的异常类的列表，目标方法可以抛出  这些异常而不会导致通知执行回滚   |
| rollback-for    | 以逗号分隔的异常类的列表，当目标方法抛出这些  异常时会导致通知执行回滚。默认情况下，该列表为空，  因此不在no-rollback-for列表中的任何运行  时异常都会导致回滚 |


如果你希望针对所有的Service类都包裹在事务中，则：
```<aop:config>
    <aop:pointcut id="fooServiceMethods" expression="execution(* x.y.service.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceMethods"/>
</aop:config>
```
## Multiple TransactionalManager
```public class TransactionalService {

    @Transactional("order")
    public void setSomething(String name) { ... }

    @Transactional("account")
    public void doSomething() { ... }
}
```

```<tx:annotation-driven/>

    <bean id="transactionManager1" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        ...
        <qualifier value="order"/>
    </bean>

    <bean id="transactionManager2" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        ...
        <qualifier value="account"/>
    </bean>
```
## 事务回滚
一般来说，声明式事务都是利用抛出异常进行回滚，在`tx:advice`的配置中，也可以对不同的方法指定不同的回滚类：
```<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
    <tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice><tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="updateStock" no-rollback-for="InstrumentNotFoundException"/>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice><tx:advice id="txAdvice">
    <tx:attributes>
    <tx:method name="*" rollback-for="Throwable" no-rollback-for="InstrumentNotFoundException"/>
    </tx:attributes>
</tx:advice>
```
同时也可以在代码中指明特定的回滚规则，譬如：
```public void resolvePosition() {
    try {
        // some business logic...
    } catch (NoProductInStockException ex) {
        // trigger rollback programmatically
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```
# 编程式事务
## PlatformTransactionManager
```DefaultTransactionDefinition def = new DefaultTransactionDefinition();
// explicitly setting the transaction name is something that can only be done programmatically
def.setName("SomeTxName");
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

TransactionStatus status = txManager.getTransaction(def);
try {
    // execute your business logic here
}
catch (MyException ex) {
    txManager.rollback(status);
    throw ex;
}
txManager.commit(status);
```
## TransactionTemplate
如果使用TransactionTemplate的话，那么整个应用的代码必须包裹在一个事务的上下文中，如下所示。一般来说用匿名内部类来实现一个`TransactionCallback`中的`execute(..)`方法。
```public class SimpleService implements Service {

    // single TransactionTemplate shared amongst all methods in this instance
    private final TransactionTemplate transactionTemplate;

    // use constructor-injection to supply the PlatformTransactionManager
    public SimpleService(PlatformTransactionManager transactionManager) {
        Assert.notNull(transactionManager, "The 'transactionManager' argument must not be null.");
        this.transactionTemplate = new TransactionTemplate(transactionManager);
    }

    public Object someServiceMethod() {
        return transactionTemplate.execute(new TransactionCallback() {
            // the code in this method executes in a transactional context
            public Object doInTransaction(TransactionStatus status) {
                updateOperation1();
                return resultOfUpdateOperation2();
            }
        });
    }
}
```
如果你不希望有返回值，可以用如下:
```transactionTemplate.execute(new TransactionCallbackWithoutResult() {
    protected void doInTransactionWithoutResult(TransactionStatus status) {
        updateOperation1();
        updateOperation2();
    }
});
```
也可以手动地控制回滚：
```transactionTemplate.execute(new TransactionCallbackWithoutResult() {

    protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
            updateOperation1();
            updateOperation2();
        } catch (SomeBusinessExeption ex) {
            status.setRollbackOnly();
        }
    }
});
```
### Settings
```public class SimpleService implements Service {

    private final TransactionTemplate transactionTemplate;

    public SimpleService(PlatformTransactionManager transactionManager) {
        Assert.notNull(transactionManager, "The 'transactionManager' argument must not be null.");
        this.transactionTemplate = new TransactionTemplate(transactionManager);

        // the transaction settings can be set here explicitly if so desired
        this.transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);
        this.transactionTemplate.setTimeout(30); // 30 seconds
        // and so forth...
    }
}
```
或者直接配置在XML中：
```<bean id="sharedTransactionTemplate"
        class="org.springframework.transaction.support.TransactionTemplate">
    <property name="isolationLevelName" value="ISOLATION_READ_UNCOMMITTED"/>
    <property name="timeout" value="30"/>
</bean>"
```


# Distributed Transaction

> 
- [分布式事务笔记 ](http://www.yangguo.info/2016/05/23/%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E7%AC%94%E8%AE%B0/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io) 

