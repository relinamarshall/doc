---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Mybatis

## MyBatis概述

### **1、MyBatis是什么，ORM是什么？**

* MyBatis是一个半ORM(对象关系映射)框架，它内部封装了JDBC，开发时只需要关注SQL语句本身，不需要花费精力去处理加载驱动、创建连接、创建Statement等繁杂的过程。程序员直接编写原生态SQL，可以严格控制SQL执行性能，灵活度高
* MyBatis可以使用XML或注解来配置和映射原生信息，将POJO映射成数据库中的记录，避免了几乎所有的JDBC代码和手动设置参数以及获取结果集
* ORM(Object Relational Mapping)，对象关系映射，是一种为了解决关系型数据库数据与简单Java对象（POJO）的映射关系的技术。简单来说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系型数据库中

### **2、MyBatis优缺点？**

**优点**：与传统的数据库访问技术相比，ORM有以下优点

* 基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在XML里，解除SQL与程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可复用
* 与JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接
* 很好的与各种数据库兼容（因为MyBatis使用JDBC来连接数据库，所以只要JDBC支持的数据库MyBatis都支持）
* 提供映射标签，支持对象与数据库的ORM字段关系映射；提供对象关系映射标签，支持对象关系组件维护
* 能够与Spring很好的即成

**缺点**

* SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求
* SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库

### **3、Hibernate和MyBatis区别，适用场景？**

**相同点**

* 都是对JDBC的封装，都是持久层的框架，都是用于DAO层（ 介于业务逻辑层和数据库之间,进行数据的访问和操作 ）的开发

**不同点**

* 映射关系
  * MyBatis是一个半自动映射的框架，配置Java对象与SQL语句执行结果的对应关系，多表关联关系配置简单
  * Hibernate是一个全表映射的框架，配置Java对象与数据库表的对应关系，多表关联关系配置复杂
* SQL优化和移植性
  * MyBatis需要手动编写SQL，支持动态SQL、处理列表、动态生成表名、支持存储过程。开发工作量相对大些。直接使用SQL语句操作数据库，不支持数据库无关性，但SQL语句优化容易
  * Hibernate对SQL语句封装，提供了日志、缓存、级联（级联比MyBatis强大）等特性，此外还提供HQL(Hibernate Query Language) 操作数据库，数据库无关系性支持好，但会消耗性能。如果项目需要支持多种数据库，代码开发量少，但SQL语句优化困难

**适用场景**

* MyBatis专注于SQL本身，是一个足够灵活的DAO层解决方案
* 对性能的要求很高，或者需求变化较多的项目，如互联网项目，MyBatis将是不错的选择
* Hibernate是重量级框架，学习使用门槛高，适合于需求相对稳定，中小型的项目，比如：办公自动化系统
* MyBatis是轻量级框架，学习使用门槛低，适合于需求变化频繁，大型的项目，比如：互联网电子商务系统

**总结**

* MyBatis是一个小巧，方便、高效、简单、直接、半自动化的持久层框架
* Hibernate是一个强大、方便、高效、复杂、间接、全自动化的持久层框架

### **4、为什么说Mybatis是半自动ORM映射工具,与全自动的区别在哪?**

* Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的
* 而MaBatis在查询关联对象或关联集合对象时，需要手动编写SQL来完成，所以，被称之为半自动化ORM映射工具

### **5、传统JDBC开发存在什么问题？**

* 频繁创建数据库连接对象、释放，容易造成系统资源浪费，影响系统性能。可以使用连接池解决这个问题，但是使用JDBC需要自己实现连接池
* SQL语句定义、参数设置，结果及处理存在硬编码。实际项目中SQL语句变化的可能性较大，一旦发生变法，需要修改Java代码，系统需要重新编译，重新发布，不好维护
* 向SQL语句传参数麻烦，因为SQL语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应，修改SQL还需要修改代码，系统不易维护
* 结果集处理存在重复代码，处理麻烦，如果可以映射成Java对象会比较方便

### **6、JDBC编程有哪些不足之处，MyBatis是如何解决的？**

* 1、数据连接创建、释放频繁，导致系统资源浪费从而影响系统性能，如果使用数据库连接池可解决此问题
  * **解决**：在MyBatis-config.xml中配置数据连接池，使用连接池管理数据库连接
* 2、SQL语句写在代码中造成代码不易维护，实际应用SQL变化较大，SQL变动需要改变Java代码
  * **解决**：将SQL语句配置在XXXMapper.xml文件中与Java代码分离
* 3、向SQL语句中传参麻烦，因为SQL语句Where条件不确定，占位符需要和参数一一对应
  * **解决**：MyBatis自动将Java对象映射至SQL语句，同时MyBatis还支持动态SQL的编写
* 4、对结果集解析麻烦，SQL变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成POJO对象解析比较方便
  * **解决**：MyBatis自动将SQL执行结果映射至Java对象

***

## MyBatis架构 <a href="#usercontentmybatis-jia-gou" id="usercontentmybatis-jia-gou"></a>

### **1、MyBatis使用过程是什么样的，生命周期呢？**

**使用过程**

*   1、创建SqlSessionFactory

    ```java
    // 可以从配置或者直接编码来创建SqlSessionFactory
    String resource = "org/mybatis/example/mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    ```
*   2、通过SqlSessionFactory创建SqlSession

    ```java
    // SqlSession（会话）可以理解为程序和数据库之间的桥梁
    SqlSession session = sqlSessionFactory.openSession();
    ```
*   3、通过SqlSession执行数据库操作

    ```java
    // 可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
    Blog blog = (Blog)session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
    // 更常用的方式是先获取Mapper(映射)，然后再执行SQL语句
    BlogMapper mapper = session.getMapper(BlogMapper.class);
    Blog blog = mapper.selectBlog(101);
    ```
*   4、调用session.commit()提交事务

    ```java
    // 如果是更新、删除语句，我们还需要提交一下事务。
    session.commit()
    ```
* 5、调用session.close()关闭会话

**生命周期**： 上面提到了几个MyBatis的组件，一般说的MyBatis生命周期就是这些组件的生命周期。

*   SqlSessionFactoryBuilder

    一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的生命周期只存在于方法的内部。
*   SqlSessionFactory

    SqlSessionFactory 是用来创建SqlSession的，相当于一个数据库连接池，每次创建SqlSessionFactory都会使用数据库资源，多次创建和销毁是对资源的浪费。所以SqlSessionFactory是应用级的生命周期，而且应该是单例的。
*   SqlSession

    SqlSession相当于JDBC中的Connection，SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的生命周期是一次请求或一个方法。
*   Mapper

    映射器是一些绑定映射语句的接口。映射器接口的实例是从 SqlSession 中获得的，它的生命周期在sqlsession事务方法之内，一般会控制在方法级。

<figure><img src="../.gitbook/assets/image (96).png" alt=""><figcaption><p>生命周期</p></figcaption></figure>

**总结**

* 当然，万物皆可集成Spring，MyBatis通常也是和Spring集成使用，Spring可以帮助我们创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到我们的 bean 中，我们不需要关心它们的创建过程和生命周期，那就是另外的故事了。

### **2、说说MyBatis工作原理？**

**MyBatis的工作原理如下图**

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>MyBatis工作原理</p></figcaption></figure>

* 1、读取MyBatis配置文件：mybatis-config.xml为MyBatis的全局配置文件，配置了MyBatis的运行环境等信息，列如数据库连接信息
* 2、加载映射文件，映射文件即SQL映射文件，该文件配置了操作数据库的SQl语句，需要在MyBatis配置文件mybatis-config.xml中加载；mybatis-config.xml文件可以加载多个映射文件，每个文件对应数据库中的一张表
* 3、构造会话工厂：通过MyBatis的环境等配置信息构建会话工厂SqlSessionFactory
* 4、创建会话对象：有会话工厂创建SqlSession对象，该对象中包含了执行Sql语句的所有方法
* 5、Executor执行器：MyBatis底层定义了一个Executor接口来操作数据库，他将根据SqlSession传递的参数动态地生成需要执行的SQL语句，同时负责查询缓存的维护
* 6、MappedStatement对象：在Executor接口的执行方法中有一个MappedStatement类型的参数，该参数是对映射信息的封装，用于存储要映射的SQL语句的ID、参数等信息
* 7、输入参数映射：输入参数类型可以是Map，List等集合类型，也可以是基本数据类型和POJO类型；输入参数映射过程类似于JDBC对PreparedStatement对象设置参数的过程（类似传统JDBC开发给SQL设置参数的过程）
* 8、输出结果映射：输出结果类型可以是Map，List等集合类型，也可以是基本数据类型和POJO类型，输出结果映射过程类似于JDBC对结果集的解析过程

### **3、Mybatis的功能架构是怎样的？**

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption><p>MyBatis架构</p></figcaption></figure>

**我们可以把MyBatis的功能架构分为三层：**

* API接口层：提供给外部使用的接口API，开发人员通过这些本地API来操作数据库，接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理
* 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等，它主要的目的是根据调用的请求完成一次数据库操作
* 基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的，将它们抽取出来作为最基础的组件，为上层的数据处理层提供最基础的支撑

### **4、MyBatis框架架构设计是怎么样的？**

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption><p>MyBatis框架架构</p></figcaption></figure>

* 这张图从上往下看；MyBatis的初始化，会从mybatis-config.xml配置文件，解析构造成Configguration这个类，途中的红框
* 1、加载配置：配置来源于两个地方，一处是配置文件，一处是Java代码的注解，将SQL的配置信息加载成为一个个MappedStatement对象（包括了传入参数映射配置、执行的SQL语句、结果映射配置），存储在内存中
* 2、SQL解析：当API接口层接收到调用请求时，会接收到传入SQL的ID和传入对象（可以是Map、JavaBean或者基本数据类型），MyBatis会根据SQL的ID找到对应的MappedStatement，然后根据传入参数对象对MappedStatement进行解析，解析后可以得到最终要执行的SQL语句和参数
* 3、SQL执行：将最终得到的SQL和参数拿到数据库进行执行，得到操作数据库的结果
* 4、结果映射：将操作数据库的结果按照映射的配置进行转换，可以转换成HashMap、JavaBean或者基本数据类型，并将最终结果返回

### **5、什么是DBMS?**

* DBMS：数据库管理系统（database management system）是一种操纵和管理数据库的大型软件，用于建立、使用和维护数据库，简称DBMS。它对数据库进行统一的管理和控制，以保证数据库的安全性和完整性；用户通过DBMS访问数据库中的数据，数据库管理员也通过DBMS进行数据库的维护工作；它可以使多个应用程序和用户用不同的方法在同时刻或不同时刻去建立，修改和查询数据库；DBMS提供数据定义语言DDL(Data Definition Language) 与数据库操作语言DML（Data Manipulation language），供用户定义数据库的模式结构与权限约束，实现对数据追加权、删除等操作

### **6、为什么需要预编译？**

**定义**

* SQL预编译指的是数据库驱动在发送SQL语句和参数给DBMS之前对SQL语句进行编译，这样DBMS执行SQL时，就不需要重新编译

**为什么需要预编译**

* JDBC中使用对象PreparedStatement来抽象预编译语句，使用预编译；与编译阶段可以优化SQL的执行；预编译之后的SQL多数情况下可以直接执行，DBMS不需要再次编译，越复杂的SQL，编译的复杂度越大，预编译阶段可以合并多次操作为一个操作；同时预编译语句对象可以重复利用；把一个SQL预编译后产生的PreparedStatement对象缓存下来，下次对于同一个SQL，可以直接使用这个缓存的PreparedState对象，MyBatis默认情况下，将对所有的SQL进行预编译
* 还有一个重要原因，防止SQL注入，后面注入的参数将不会再次触发 SQL 编译。也就是说，对于后面注入的参数，系统将不会认为它会是一个 SQL 命令，而默认其是一个参数，参数中的 or 或 and 等（SQL 注入常用技俩）就不是 SQL 语法保留字了

### **7、MyBatis有哪些执行器Executor，它们之间区别是什么？**

**MyBatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutory、BatchExecutor**

* **SimpleExecutor**：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象
* **ReuseExecutor**：执行update或select，以sql作为key查找Statement对象，存在就是用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map\<String，Statement>内，供下一次使用。简言之，就是重复使用Statement对象
* **BatchExecutor**：执行update(没有select，JDBC批处理不支持select)，将所有Sql都添加到批处理中（addBatch()）,等待统一执行（executeBatch()）,它缓存了多个Statement对象，每个Statement对象都是addBacth()完毕后，等待逐一执行executeBatch()批处理；与JDBC批处理相同。

`作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内`

### **8、MyBatis如何指定使用哪一种Executor执行器？**

* 在MyBatis配置文件中，在设置（setting）可以指定默认的ExecutorType执行器类型，也可以手动DefaultSqlSessionFactory的创建SqlSession的方法传递ExecutorType类型参数，如SqlSession.openSession(ExecutorType)
* 配置默认的执行器，SIMPLE就是普通的执行器；REUSE执行器会重用预处理语句(preparedstatement)； BATCH 执行器不仅重用语句还会执行批量更新

### **9、MyBatis是否支持延迟加载，如果支持，他的实现原理是什么？**

* MyBatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的是一对一，collection指的是一对多查询；在MyBatis配置文件中，可以配置是否启用延迟加载**lazyLoadingEnabled=true|false**
* 他的原理，使用CGLIB创建目标对象的代理对象，当调用目标方法是，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的SQL，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用，这就是延迟加载的基本原理
* 当然，不光是MyBatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的

***

## MyBatis映射器 <a href="#usercontentmybatis-ying-she-qi" id="usercontentmybatis-ying-she-qi"></a>

### **1、#{}和${}区别？**

* \#{}是占位符，预编译处理；${}是拼接符，字符串替换，没有预编译处理
* MyBatis在处理#{}时，#{}传入参数是以字符串传入，会将SQL中的#{}代替为？号，调用PreparedStatement的set方法来赋值
* \#{}可以有效的防止SQL注入，提高系统安全性；${}不能防止SQL注入
* \#{}对应的变量会自动加上单引号；${}对应变量不会加上单引号
* \#{变量名}会转化为jdbc的类型；${变量名}不进行数据类型匹配，直接替换
* \#{}和${}的默认值；在不同版本中可能有会一些差异
* \#{}的变量替换是在DBMS中；${}的变量替换是在DBMS外

### **2、模糊查询like语句该怎么写？**

* `%${question}%`可能会引起SQL注入，不推荐
* `"%"#{}"%"`注意：因为#{...}解析成SQL语句时候，会在变量外侧自动加上单引号' '，所以这里%需要使用双引号" "，不能使用单引号' '，不然会查不到任何结果
* CONCAT('%',#{question},'%') 使用concat()函数(推荐)
*   使用bind标签(不推荐)

    ```xml
    <select id="likeUsername" resultType="User">
    	<bind name="pattern" value="'%' + username + '%'" />
    	select id,sex,age,username,password from person where username LIKE #{pattern}
    </select>
    ```

### **3、在mapper中如何传递多个参数？**

*   顺序传参方式

    ```xml
    public User selectUser(String name, int deptId);

    <select id="selectUser" resultMap="UserResultMap">
    	select * from user 
        where user_name = #{param1} and dept_id = #{param2}
    </select>
    ```

    * \#{}里面的数字代表传入参数的顺序
    * 这种方法不建议使用，SQL层表达不直观，且一旦顺序调整容易出错
*   @Param注解传参方式

    ```xml
    public User selectUser(@Param("userName") String name, int @Param("deptId")
    deptId);

    <select id="selectUser" resultMap="UserResultMap">
    	select * from user 
        where user_name = #{userName} and dept_id = #{deptId}
    </select>
    ```

    * \#{}里面的名称对应的是注解@Param括号里面修饰的名词
    * 这种方法参数不多的情况还是比较直观的（推荐使用）
*   Map传参方式

    ```xml
    public User selectUser(Map<String, Object> params);

    <select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
    	select * from user
        where user_name = #{userName} and dept_id = #{deptId}
    </select>
    ```

    * \#{}里面的名称对应的是Map里面的Key名称
    * 这种方法是和传递多个参数，且参数易变能灵活传递的情况（推荐使用）
*   Java Bean传参方式

    ```xml
    public User selectUser(User user);

    <select id="selectUser" parameterType="User" resultMap="UserResultMap">
    	select * from user
    	where user_name = #{userName} and dept_id = #{deptId}
    </select>
    ```
* \#{}里面的名称对应的是User类里面的成员属性
  * 这种方法直观，需要建一个实体类，扩展不易，需要如加属性，但代码可读性强，业务逻辑处理方便（推荐使用）

### **4、MyBatis如何执行批量操作？**

* 使用ForEach标签
  * foreach的主要用在构建in条件中，它可以在SQL语句中进行迭代一个集合。foreach标签的构建主要有item，index，collection，open，separator，close
    * item：表示集合中每一个元素进行迭代时的别名，随便起的变量名
    * index： 指定一个名字，用于表示在迭代过程中，每次迭代到的位置，不常用
    * open： 表示该语句以什么开始，常用\*\*"（"\*\*
    * separator： 表示在每次迭代之间以什么符号作为分隔符，]常用\*\*","\*\*
    * close： 表示以什么结束，常用\*\*"）"\*\*
  *   在使用foreach的时候最关键的也是最容易出错的就是collection属性，该属性是必须指定的，但是在不同情况下，该属性的值是不一样的，主要有以下3中情况

      * 如果传入的是单参数且参数类型是一个List的时候，collection的属性值为List
      * 如果传入的是单参数且参数类型是一个Array数组的时候，collection的属性值为Array
      * 如果传入的参数是多个的时候，就需要吧它们封装成一个Map了，当然单参数也可以封装成Map，实际上如果你在传入参数的时候，在MyBatis里面也是会把它封装成一个Map的，Map的Key就是参数名，所以这时候collection属性值就是传入的List或Array对象在自己封装的Map里面的Value

      ```xml
      <!-- MySQL下批量保存，可以foreach遍历 mysql支持values(),(),()语法 --> //推荐使用
      <insert id="addEmpsBatch">
          INSERT INTO emp(ename,gender,email,did)
          VALUES
          <foreach collection="emps" item="emp" separator=",">
              (#{emp.eName},#{emp.gender},#{emp.email},#{emp.dept.id})
          </foreach>
      </insert>

      <!-- 这种方式需要数据库连接属性allowMultiQueries=true的支持
      如jdbc.url=jdbc:mysql://localhost:3306/mybatis?allowMultiQueries=true -->  
      <insert id="addEmpsBatch">
          <foreach collection="emps" item="emp" separator=";">       
              INSERT INTO emp(ename,gender,email,did)
              VALUES(#{emp.eName},#{emp.gender},#{emp.email},#{emp.dept.id})
          </foreach>
      </insert>
      ```
* 使用ExecutorType.BATCH
  *   MyBatis内置的ExecutorType有3种，默认为Simple，该模式下它为每个语句的执行创建一个新的预处理语句，单条提交SQL；而Batch模式重复使用已经预处理的语句，并且批量执行所有更新语句，显然Batch性能将更优；但Batch模式也有自己的问题，比如在Insert操作时，在事务没有提交之前，是没有办法获取到自增的ID，在某些情况下不符合业务的需求

      ```java
      //批量保存方法测试
      @Test  
      public void testBatch() throws IOException{
          String resource = "org/mybatis/example/mybatis-config.xml";
          InputStream inputStream = Resources.getResourceAsStream(resource);
          SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
          //可以执行批量操作的sqlSession
          SqlSession openSession = sqlSessionFactory.openSession(ExecutorType.BATCH);

          //批量保存执行前时间
          long start = System.currentTimeMillis();
          try {
              EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
              for (int i = 0; i < 1000; i++) {
                  mapper.addEmp(new Employee(UUID.randomUUID().toString().substring(0, 5), "b", "1"));
              }

              openSession.commit();
              long end = System.currentTimeMillis();
              //批量保存执行后的时间
              System.out.println("执行时长" + (end - start));
              //批量 预编译sql一次==》设置参数==》10000次==》执行1次   677
              //非批量  （预编译=设置参数=执行 ）==》10000次   1121

          } finally {
              openSession.close();
          }
      }
      ```



      ```java
      public interface EmployeeMapper {   
          //批量保存员工
          Long addEmp(Employee employee);
      }
      ```

      ```xml
      <mapper namespace="EmployeeMapper"
           <!--批量保存员工 -->
          <insert id="addEmp">
              insert into employee(lastName,email,gender)
              values(#{lastName},#{email},#{gender})
          </insert>
      </mapper>
      ```

### **5、如何获取生成的主键？**

*   新增标签中添加：KeyProperty="ID" 和 useGeneratedKeys="true"，即可

    ```xml
    <insert id="insertGetId" parameterType="com.wen.pojo.User" keyColumn="id" useGeneratedKeys="true" keyProperty="id">
        insert into user(`name`, pwd, perms)
        values (#{name}, #{pwd}, #{perms})
    </insert>
    ```
*   SelectKey标签

    ```xml
    <insert id="insertGetId2" parameterType="com.wen.pojo.User">
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into user(`name`, pwd, perms)
        values (#{name}, #{pwd}, #{perms})
    </insert>
    ```

### **6、当实体类中的属性名和表中的字段名不一样，怎么办？**

*   通过在查询SQL语句中定义字段名的别名，让别名和实体类的属性名一致

    ```xml
    <select id="getOrder" parameterType="int"
    resultType="Order">
    	select order_id id, order_no orderno ,order_price price form orders
    	where order_id=#{id};
    </select>
    ```
*   通过\<resultMap>来映射字段名和实体类属性名的一一对应关系

    ```xml
    <select id="getOrder" parameterType="int" resultMap="orderResultMap">
    	select * from orders where order_id=#{id}
    </select>

    <resultMap type="Order" id="orderResultMap">
        <!–用id属性来映射主键字段–>
        <id property="id" column="order_id">
        <!–用result属性来映射非主键字段，property为实体类属性名，column为数据库表中的属性–>
        <result property ="orderno" column ="order_no"/>
        <result property="price" column="order_price" />
    </reslutMap>
    ```

### **7、Mapper编写有哪几种方式？**

* 接口实现类继承SqlSessionDaoSupport；使用此方法需要编写Mapper接口，Mapper接口实现类、Mapper.xml文件
  *   在mybatis-config.xml中配置mapper.xml的位置

      ```xml
      <mappers>
          <mapper resource="mapper.xml 文件的地址" />
          <mapper resource="mapper.xml 文件的地址" />
      </mappers>
      ```
  * 定义Mapper接口
  * 实现类集成SqlSessionDaoSupport
    * Mapper方法中可以this.getSqlSession()进行数据操作
  *   Spring配置

      ```xml
      <bean id=" " class="mapper 接口的实现">
      	<property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
      </bean>
      ```
* 使用org.mybatis.spring.mapper.MapperFactoryBean
  * 在myabtis-config.xml中配置mapper.xml的位置，如果mapper.xml和mapper接口的名称相同且在同一个目录，这里可以不用配置
  * 定义Mapper接口
    * mapper.xml中的namespace为mapper接口的地址
    * mapper接口中的方法名和mapper.xml中的定义的statement的Id保持一致
  *   Spring中定义

      ```xml
      <bean id="" class="org.mybatis.spring.mapper.MapperFactoryBean">
          <property name="mapperInterface" value="mapper 接口地址" />
          <property name="sqlSessionFactory" ref="sqlSessionFactory" />
      </bean>
      ```
* 使用mapper扫描器
  * mapper.xml文件编写
    * mapper.xml中的namespace为mapper接口的地址
    * mapper接口中的方法名和mapper.xml中的定义的statement的Id保持一致
  * 定义mapper接口
    * 注意mapper.xml的文件名和mapper的接口名保持一致，且放在同一个目录
  *   配置mapper扫描器

      ```xml
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
          <property name="basePackage" value="mapper 接口包地址"></property>
          <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
      </bean>
      ```
  * 使用扫描器后从Spring容器中获取mapper的实现对象

### **8、什么是MyBatis的接口绑定，有哪些实现方式？**

* 接口绑定，就是在MyBatis中任意定义接口，然后把接口里面的方法和SQL语句绑定，直接调用接口方法就行，这样比起原来SqlSession提供的方法，可以有更加灵活的选择和设置
* 接口绑定有两种实现方式
  * 通过注解绑定，就是在接口的方法上面加上@Selelct、@Update等注解，把里面Sql语句来绑定
  * 通过Xml里面写Sql来绑定，在这种情况下，要制定xml映射文件里面的namespace必须为接口的全路径名。当Sql语句比较简单时候，用注解绑定，当Sql语句复杂时候，用Xml方式绑定，一般用Xml绑定的比较多

### **9、使用MyBatis的Mapper接口调用时有哪些要求？**

* Mapper接口方法名和mapper.xml中定义的每个SQL的ID相同
* Mapper接口方法的输入参数类型和Mapper.xml中定义的每个SQL的parameterType的类型相同
* Mapper接口方法的输出类型和Mapper.xml中定义的每个SQL的resultType的类型相同
* Mapper.xml文件中namespace即是mapper接口的类路径

### **10、Dao接口工作原理是什么，Dao接口里的方法，参数不同是时，方法能重载吗？**

* Dao接口的工作原理JDK动态代理，MyBatis运行会使用JDK动态代理为Dao接口生成代理的proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的SQL，然后将SQL执行结果返回
* Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略，重载后还会报错

### **11、MyBatis的Xml映射文件中，不同的Xml映射文件，Id是否可以重复？**

* 不同的Xml映射文件；如果配置了namespace，那么ID可以重复；如果没有配置namespace，那么Id不能重复，毕竟namespace不是必须的，只是最佳实践而已；但随着namespace重要性，好像必须要配置了
* 原因就是namespace+id是作为Map\<String,MappedStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据交互覆盖；有了namespace，自然Id就可以重复，namespace不同，namespace+id自然也就不同

### **12、简述MyBatis的Xml映射文件和MyBatis内部数据结构之间的映射关系？**

* MyBatis将所有Xml配置信息都封装到All-In-One重量级对象Configuration内部
* 在Xml映射文件中，\<parameterMap>标签会被解析为ParameterMap对象，其每个子元素会被解析为ParameterMapping对象；\<resultMap>标签会被解析为ResultMap对象，其每个子元素会被解析为ResultMapping对象
* 每一个\<select>、\<insert>、\<update>、\<delect>标签均会被解析为MappedStatement对象，标签内的Sql会被解析为BoundSql对象

### **13、MyBatis如何将SQL执行结果封装为目标对象并返回的，都有哪些映射形式？**

* 使用\<resultMap>标签，注意定义列名和对象属性名之间的映射关系
* 使用Sql列的别名功能，将列明书写为对象属性名，比如T\_NAME as name ，对象属性名一般是name，小写，但是列名不区分大小写，MyBatis会忽略列名大小写

### **14、Xml映射文件中，除了常见的select|insert|update|delete标签之外，还有那些标签？**

* 还有很多其他标签，\<resultMap>、\<parameterMap>、\<sql>、\<include>、\<selectKey>，加上动态sql的9个标签trim|where|set|foreach|if|choose|when|otherwise|bind等，其中\<sql>为SQL片段标签，通过\<include>标签引入SQL片段，\<selectKey>为不支持自增的主键生成策略标签

### **15、MyBatis映射文件中，如果A标签通过include引用B标签的内容，请问，B标签能否定义在A标签的后面，还是说必须定义在A标签的前面？**

* 虽然MyBatis解析Xml映射文件是按照顺序解析的，但是，被引用的B标签依然可以定义在任何地方，MyBatis都可以正确识别
* 原理是，MyBatis解析A标签，发现A标签引用了B标签，但是B标签尚未解析到，尚不存在，此时，MyBatis会将A标签标记为为解析状态，然后继续解析余下的标签，包含B标签，待所有标签解析完毕，MyBtis会重新解析那些被标记为未解析的标签，此时在解析A标签时，B标签已经存在，A标签也就可以正常解析完成了

### **16、MyBatis能执行一对多，一对一的联系查询吗，有哪些实现方法？**

* 能，不止一对多，一对一还可以多对多，一对多
* 实现方式
  * 单独发送一个SQL去嵌套查询关联对象，赋给主对象，然后返回主对象
  * 使用关联查询，似JOIN查询，一部分是A对象的属性值，另一部分是关联对象B的属性值，好处是只要发送一个属性值，就可以把主对象和关联对象查出来
  * 子查询
* Mapper.xml层面中实现关联查询有两种方式
* 配置文件方式
  * 一对一：在resultMap标签中使用association标签
  * 一对多：在resultMap标签中使用collection标签
*   注解方式

    * 在@Results注解中的@Result注解中使用@One注解
    * 在@Result注解中的@Result注解中使用@Many注解

    ```java
    @Results({
        @Result(column = "id", property = "id"),
        @Result(column = "tid", property = "teacher", one = @One(
            select = "com.wen.mapper.TeacherMapper.getOne"
        ))
    })
    @Select("select * from student")
    public List<Student> getAll();

    @Select("select * from teacher where id = #{tid}")
    public Teacher getOne( int tid);
    ```

### **17、MyBatis是否可以映射Enum枚举类？**

* MyBtis可以映射枚举类，不单可以映射枚举类，MyBatis可以映射任何对象到表的一列上；映射方式为自定义一个TypeHandler，实现TypeHandler的setParameter()和getResult()接口方法
* TypeHandler有两个作用，一是完成从JavaType至JdbcType的转换，二是完成JdbcType至Java的转换，体现为setParameter()和getResult()两个方法，分别代表设置SQL问好占位符参数和获取列查询结果

### **18、MyBatis动态SQL是做什么的，都有哪些动态sql，简单描述动态SQL的执行原理？**

* MyBatis动态SQL可以在Xml映射文件内，一标签的形式编写动态SQL，完成逻辑判断和动态拼接SQL的功能，MyBatis提供了9中动态SQL标签 trim|where|set|foreach|if|choose|otherwise|bind|when
* 执行原理为，使用OGNL从sql参数对象中计算表达式的值，根据表达式的值动态拼接sql，以此来完成动态sql的功能。

### **19、MyBatis是如何进行分页的，分页插件的原理是什么？**

* MyBatis使用RowBounds对象进行分页，他是正对ResultSet结果集执行的内存分页，而非物理分页，可以在SQL内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页
* 分页插件的基本原理是使用MyBatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的SQL，然后重写SQL，更具dialect方言，添加对应的物理分页语句和物理分页参数
* 距离 select \* from student，拦截SQL重写为：select \* from (select \* from student) t limit 0,10

### **20、简述MyBatis的插件运行原理，以及如何编写一个插件？**

<figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>插件运行原理</p></figcaption></figure>

* Mybatis会话的运行需要ParameterHandler、ResultSetHandler、StatementHandler、Executor这四大对象的配合，插件的原理就是在这四大对象调度的时候，插入一些我我们自己的代码
* Mybatis使用JDK的动态代理，为目标对象生成代理对象。它提供了一个工具类`Plugin`，实现了`InvocationHandler`接口

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption><p>Intercept</p></figcaption></figure>

* 使用`Plugin`生成代理对象，代理对象在调用方法的时候，就会进入invoke方法，在invoke方法中，如果存在签名的拦截方法，插件的intercept方法就会在这里被我们调用，然后就返回结果。如果不存在签名方法，那么将直接反射调用我们要执行的方法

**如何编写一个插件**

编写MyBatis 插件，只需要实现拦截器接口 `Interceptor (org.apache.ibatis. plugin Interceptor ）`，在实现类中对拦截对象和方法进行处理

*   实现Mybatis的Interceptor接口并重写intercept()方法

    这里我们只是在目标对象执行目标方法的前后进行了打印；

    ```java
    public class MyInterceptor implements Interceptor {
        Properties props=null;

        @Override
        public Object intercept(Invocation invocation) throws Throwable {
            System.out.println("before……");
            //如果当前代理的是一个非代理对象，那么就会调用真实拦截对象的方法
            // 如果不是它就会调用下个插件代理对象的invoke方法
            Object obj=invocation.proceed();
            System.out.println("after……");
            return obj;
        }
    }
    ```
*   然后再给插件编写注解，确定要拦截的对象，要拦截的方法

    ```java
    @Intercepts({@Signature(
            type = Executor.class,    //确定要拦截的对象
            method = "update",        //确定要拦截的方法
            args = {MappedStatement.class,Object.class}   //拦截方法的参数
    )})
    public class MyInterceptor implements Interceptor {
        Properties props=null;
        
        @Override
        public Object intercept(Invocation invocation) throws Throwable {
            System.out.println("before……");
            //如果当前代理的是一个非代理对象，那么就会调用真实拦截对象的方法
            // 如果不是它就会调用下个插件代理对象的invoke方法
            Object obj=invocation.proceed();
            System.out.println("after……");
            return obj;
        }
    }
    ```
*   最后，再MyBatis配置文件里面配置插件

    ```xml
    <plugins>
        <plugin interceptor="xxx.MyPlugin">
           <property name="dbType",value="mysql"/>
        </plugin>
    </plugins> 
    ```

### **21、MyBatis的一级、二级缓存**

* 一级缓存：基于PerpetualCache的HashMap本地缓存，其存储作用域为Session，当Session Flush或Close之后，该Session中的所有Cache就将清空，默认打开一级缓存
* 二级缓存与一级缓存其机制相同，默认也是采用PerpetualCache，HashMap存储，不同在于其存储作用域为Mapper(namespace)，并且可以之定义存储源，如Ehcache；默认不开启二级缓存，需要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态)，可在它的映射文件中配置\<cache>
* 对于缓存数据更新机制，当某一个作用域（一级缓存Session / 二级缓存NameSpace）的进行了数据更新操作后，默认该作用域下所有Select中的缓存将被clear

### **22、为什么Mapper接口不需要实现类**

四个字回答：**动态代理**，看一下获取Mapper的过程

<figure><img src="../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption><p>Mapper获取过程</p></figcaption></figure>

*   获取Mapper：我们都知道定义的Mapper接口是没有实现类的，Mapper映射其实是通过动态代理实现的

    ```java
    // 获取Mapper
    BlogMapper mapper = session.getMapper(BlogMapper.class);
    ```
*   七拐八绕地进去看一下，发现获取Mapper的过程，需要先获取MapperProxyFactory——Mapper代理工厂

    ```java
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory)this.knownMappers.get(type);
        if (mapperProxyFactory == null) {
            throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
        } else {
            try {
                return mapperProxyFactory.newInstance(sqlSession);
            } catch (Exception var5) {
                throw new BindingException("Error getting mapper instance. Cause: " + var5, var5);
            }
        }
    }
    ```
*   MapperProxyFactory： MapperProxyFactory的作用是生成MapperProxy（Mapper代理对象）

    ```java
    public class MapperProxyFactory<T> {
      private final Class<T> mapperInterface;
      ……
      protected T newInstance(MapperProxy<T> mapperProxy) {
          return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
      }

      public T newInstance(SqlSession sqlSession) {
          MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
          return this.newInstance(mapperProxy);
      }
    }
    ```
*   MapperProxy： MapperProxy里，通常会生成一个MapperMethod对象，它是通过cachedMapperMethod方法对其进行初始化的，然后执行excute方法

    ```java
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            return Object.class.equals(method.getDeclaringClass()) ? method.invoke(this, args) : this.cachedInvoker(method).invoke(proxy, method, args, this.sqlSession);
        } catch (Throwable var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }
    }
    ```
*   MapperMethod： MapperMethod里的excute方法，会真正去执行sql。这里用到了命令模式，其实绕一圈，最终它还是通过SqlSession的实例去运行对象的sql

    ```java
    public Object execute(SqlSession sqlSession, Object[] args) {
          Object result;
          Object param;
          ……
          case SELECT:
              if (this.method.returnsVoid() && this.method.hasResultHandler()) {
                  this.executeWithResultHandler(sqlSession, args);
                  result = null;
              } else if (this.method.returnsMany()) {
                  result = this.executeForMany(sqlSession, args);
              } else if (this.method.returnsMap()) {
                  result = this.executeForMap(sqlSession, args);
              } else if (this.method.returnsCursor()) {
                  result = this.executeForCursor(sqlSession, args);
              } else {
                  param = this.method.convertArgsToSqlCommandParam(args);
                  result = sqlSession.selectOne(this.command.getName(), param);
                  if (this.method.returnsOptional() && (result == null || !this.method.getReturnType().equals(result.getClass()))) {
                      result = Optional.ofNullable(result);
                  }
              }
              break;
             ……
      }
    ```
