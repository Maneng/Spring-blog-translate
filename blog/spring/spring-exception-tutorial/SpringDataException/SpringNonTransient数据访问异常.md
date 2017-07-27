# Guide to Spring NonTransientDataAccessException

**目录**  

- [Guide to Spring NonTransientDataAccessException](#guide-to-spring-nontransientdataaccessexception)
  - [**1.概述**](#1%E6%A6%82%E8%BF%B0)
  - [**2.The Base Exception Class**](#2the-base-exception-class)
  - [**3. DataIntegrityViolationException**](#3-dataintegrityviolationexception)
    - [3.1. DuplicateKeyException](#31-duplicatekeyexception)
  - [**4. DataRetrievalFailureException**](#4-dataretrievalfailureexception)
    - [4.1 IncorrectResultSetColumnCountException](#41-incorrectresultsetcolumncountexception)
    - [4.2 IncorrectResultSizeDataAccessException](#42-incorrectresultsizedataaccessexception)
  - [**5. DataSourceLookupFailureException**](#5-datasourcelookupfailureexception)
  - [**6. InvalidDataAccessResourceUsageException**](#6-invaliddataaccessresourceusageexception)
    - [6.1 BadSqlGrammarException](#61-badsqlgrammarexception)
  - [**7. CannotGetJdbcConnectionException**](#7-cannotgetjdbcconnectionexception)
  - [**8. Conclusion**](#8-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->




## **1.概述**

在这个快速教程中，我们将介绍常见的**NonTransientDataAccessException**的最重要类型，并用示例来说明它们。

## **2.The Base Exception Class**

这个主异常类的子类表示被认为是非瞬时的或永久的与数据访问有关的异常。
简单地说，这意味着，直到根本原因被解决——否则所有导致异常的方法的未来尝试都将失败。

## **3. DataIntegrityViolationException**

尝试修改数据时会引发**NonTransientDataAccessException**的这个子类型导致违反完整性约束

在我们的Foo类的例子中，name列被定义为不允许空值：

``` java
@Column(nullable = false)
private String name;
```


如果我们尝试保存一个实例而不设置名称的值，那么我们可以期待抛出一个**DataIntegrityViolationException**：

``` java
@Test(expected = DataIntegrityViolationException.class)
public void whenSavingNullValue_thenDataIntegrityException() {
    Foo fooEntity = new Foo();
    fooService.create(fooEntity);
}
```
### 3.1. DuplicateKeyException
**DataIntegrityViolationException**的子类之一是**DuplicateKeyException**，当尝试使用已存在的主键或已经存在于具有唯一约束的列中的值（例如尝试插入两行）尝试保存记录时，会抛出此异常在foo表中，id为1：

``` java
@Test(expected = DuplicateKeyException.class)
public void whenSavingDuplicateKeyValues_thenDuplicateKeyException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.execute("insert into foo(id,name) values (1,'a')");
    jdbcTemplate.execute("insert into foo(id,name) values (1,'b')");
}
```


## **4. DataRetrievalFailureException**

当检索数据出现问题时，例如查找具有数据库中不存在的标识符的对象时，会抛出此异常。

例如，我们将使用JdbcTemplate类，该类具有抛出此异常的方法：


``` java
@Test(expected = DataRetrievalFailureException.class)
public void whenRetrievingNonExistentValue_thenDataRetrievalException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
     
    jdbcTemplate.queryForObject("select * from foo where id = 3", Integer.class);
}
```

### 4.1 IncorrectResultSetColumnCountException

当尝试从表中检索多个列而不创建适当的**RowMapper**时，会抛出此异常子类：

``` java
@Test(expected = IncorrectResultSetColumnCountException.class)
public void whenRetrievingMultipleColumns_thenIncorrectResultSetColumnCountException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
 
    jdbcTemplate.execute("insert into foo(id,name) values (1,'a')");
    jdbcTemplate.queryForList("select id,name from foo where id=1", Foo.class);
```

### 4.2 IncorrectResultSizeDataAccessException
当许多检索的记录与预期记录不同时，抛出此异常，例如，当期望单个整数值，但为查询检索两行时：


``` java
@Test(expected = IncorrectResultSizeDataAccessException.class)
public void whenRetrievingMultipleValues_thenIncorrectResultSizeException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
 
    jdbcTemplate.execute("insert into foo(name) values ('a')");
    jdbcTemplate.execute("insert into foo(name) values ('a')");
 
    jdbcTemplate.queryForObject("select id from foo where name='a'", Integer.class);
}
```


## **5. DataSourceLookupFailureException**

无法获取指定的数据源时抛出此异常。例如，我们将使用**JndiDataSourceLookup**类来查找不存在的数据源：

``` java
@Test(expected = DataSourceLookupFailureException.class)
public void whenLookupNonExistentDataSource_thenDataSourceLookupFailureException() {
    JndiDataSourceLookup dsLookup = new JndiDataSourceLookup();
    dsLookup.setResourceRef(true);
    DataSource dataSource = dsLookup.getDataSource("java:comp/env/jdbc/example_db");
}
```


## **6. InvalidDataAccessResourceUsageException**


当资源访问不正确时，例如当用户缺少SELECT权限时，会抛出此异常。 为了测试这个异常，我们需要撤消用户的SELECT权限，然后运行一个SELECT查询：


``` java
@Test(expected = InvalidDataAccessResourceUsageException.class)
public void whenRetrievingDataUserNoSelectRights_thenInvalidResourceUsageException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.execute("revoke select from tutorialuser");
 
    try {
        fooService.findAll();
    } finally {
        jdbcTemplate.execute("grant select to tutorialuser");
    }
}
```
请注意，我们正在恢复finally块中用户的权限。

### 6.1 BadSqlGrammarException

**InvalidDataAccessResourceUsageException**的一个非常常见的子类型是**BadSqlGrammarException**，当尝试运行具有无效SQL的查询时，它将抛出：

``` java
@Test(expected = BadSqlGrammarException.class)
public void whenIncorrectSql_thenBadSqlGrammarException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.queryForObject("select * fro foo where id=3", Integer.class);
}
```
## **7. CannotGetJdbcConnectionException**

当连接尝试通过JDBC失败时，例如数据库url不正确时，会抛出此异常。如果我们写下如下的URL：

``` java
jdbc.url=jdbc:mysql:3306://localhost/spring_hibernate4_exceptions?createDatabaseIfNotExist=true
```
然后在尝试执行语句时将抛出**CannotGetJdbcConnectionException**：


``` java
@Test(expected = CannotGetJdbcConnectionException.class)
public void whenJdbcUrlIncorrect_thenCannotGetJdbcConnectionException() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(restDataSource);
    jdbcTemplate.execute("select * from foo");
}
```
## **8. Conclusion**


在这个非常到点的教程中，我们看了一些**NonTransientDataAccessException**类最常见的子类型。

在[Github](https://github.com/eugenp/tutorials/tree/master/spring-all)项目中可以找到一些这些例外示例的实现 
这是一个基于Eclipse的项目，所以应该很容易导入和运行。