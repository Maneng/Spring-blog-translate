# 介绍JOOQ和Spring


- [**1. 概述**](#1-%E6%A6%82%E8%BF%B0)
- [**2. Maven依赖关系**](#2-maven%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB)
  - [2.1. jOOQ](#21-jooq)
  - [2.2. Spring](#22-spring)
  - [2.3. Database](#23-database)
- [3.代码生成](#3%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90)
  - [3.1.数据库结构](#31%E6%95%B0%E6%8D%AE%E5%BA%93%E7%BB%93%E6%9E%84)
  - [3.2.属性Maven插件](#32%E5%B1%9E%E6%80%A7maven%E6%8F%92%E4%BB%B6)
  - [3.3. SQL Maven插件](#33-sql-maven%E6%8F%92%E4%BB%B6)
  - [3.4. jOOQ Codegen插件](#34-jooq-codegen%E6%8F%92%E4%BB%B6)
  - [3.5.生成代码](#35%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81)
- [4. Spring Configuration](#4-spring-configuration)
  - [4.1. Translating jOOQ Exceptions to Spring](#41-translating-jooq-exceptions-to-spring)
  - [4.2. Configuring Spring](#42-configuring-spring)
- [5. Using jOOQ with Spring](#5-using-jooq-with-spring)
  - [5.1. Inserting Data](#51-inserting-data)
  - [5.2. Updating Data](#52-updating-data)
  - [5.3. Deleting Data](#53-deleting-data)
- [6. 总结](#6-%E6%80%BB%E7%BB%93)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## **1. 概述**

本文将介绍面向Java的面向对象的查询 - jOOQ - 和一个简单的方法来配合Spring框架。

大多数Java应用程序都有某种SQL持久性，并通过诸如JPA之类的更高级别的工具来访问该层。

虽然这很有用，但在某些情况下，您真的需要一个更精细，更细微的工具来获取数据或实际利用基础数据库提供的所有内容。 

jOOQ避免了一些典型的ORM模式，并生成了允许我们构建类型安全查询的代码，并通过干净强大的流畅API对生成的SQL进行完整的控制。

## **2. Maven依赖关系**

在本教程中运行代码需要以下依赖关系：

### 2.1. jOOQ

```xml
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jooq</artifactId>
    <version>3.7.3</version>
</dependency>

```



### 2.2. Spring

我们的示例需要几个Spring依赖关系;然而，为了使事情变得简单，我们只需要在POM文件中明确地包含两个：


```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.2.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.2.5.RELEASE</version>
</dependency>
```


### 2.3. Database

为了简化我们的例子，我们将利用H2嵌入式数据库：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.191</version>
</dependency>
```

## 3.代码生成

### 3.1.数据库结构

我们在本文中介绍我们将要使用的数据库结构。

假设我们需要为发行商创建一个数据库来存储他们管理的书籍和作者的信息，作者可以在那里书写许多书籍，并且书籍可能由许多作者共同撰写。

为了简单起见，我们将只生成三本表：书籍作者，作者作者，以及另一张名为author_book的表格，用于表示作者和书籍之间的多对多关系。作者表有三列：id，first_name和last_name。书表仅包含标题列和ID主键。

存储在intro_schema.sql资源文件中的以下SQL查询将针对我们之前设置的数据库执行，以创建必需的表并使用示例数据填充它们：

```sql
DROP TABLE IF EXISTS author_book, author, book;
 
CREATE TABLE author (
  id             INT          NOT NULL PRIMARY KEY,
  first_name     VARCHAR(50),
  last_name      VARCHAR(50)  NOT NULL
);
 
CREATE TABLE book (
  id             INT          NOT NULL PRIMARY KEY,
  title          VARCHAR(100) NOT NULL
);
 
CREATE TABLE author_book (
  author_id      INT          NOT NULL,
  book_id        INT          NOT NULL,
   
  PRIMARY KEY (author_id, book_id),
  CONSTRAINT fk_ab_author     FOREIGN KEY (author_id)  REFERENCES author (id)  
    ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT fk_ab_book       FOREIGN KEY (book_id)    REFERENCES book   (id)
);
 
INSERT INTO author VALUES
  (1, 'Kathy', 'Sierra'), 
  (2, 'Bert', 'Bates'), 
  (3, 'Bryan', 'Basham');
 
INSERT INTO book VALUES
  (1, 'Head First Java'), 
  (2, 'Head First Servlets and JSP'),
  (3, 'OCA/OCP Java SE 7 Programmer');
 
INSERT INTO author_book VALUES (1, 1), (1, 3), (2, 1);
```

### 3.2.属性Maven插件

我们将使用三种不同的Maven插件来生成jOOQ代码。第一个是Properties Maven插件。

该插件用于从资源文件读取配置数据。这不是必需的，因为数据可以直接添加到POM中，但是外部管理属性是个好主意。

在本节中，我们将在名为intro_config.properties的文件中定义数据库连接的属性，包括JDBC驱动程序类，数据库URL，用户名和密码。外部化这些属性可以轻松切换数据库或仅更改配置数据。


此插件的read-project-properties目标应该绑定到早期阶段，以便配置数据可以准备供其他插件使用。在这种情况下，它必须到初始化阶段：

```xml

<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>properties-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
        <execution>
            <phase>initialize</phase>
            <goals>
                <goal>read-project-properties</goal>
            </goals>
            <configuration>
                <files>
                    <file>src/main/resources/intro_config.properties</file>
                </files>
            </configuration>
        </execution>
    </executions>
</plugin>

```

### 3.3. SQL Maven插件

SQL Maven插件用于执行SQL语句来创建和填充数据库表。它将利用属性Maven插件从intro_config.properties文件中提取的属性，并从intro_schema.sql资源获取SQL语句。

SQL Maven插件配置如下：


```xml

<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>sql-maven-plugin</artifactId>
    <version>1.5</version>
    <executions>
        <execution>
            <phase>initialize</phase>
            <goals>
                <goal>execute</goal>
            </goals>
            <configuration>
                <driver>${db.driver}</driver>
                <url>${db.url}</url>
                <username>${db.username}</username>
                <password>${db.password}</password>
                <srcFiles>
                    <srcFile>src/main/resources/intro_schema.sql</srcFile>
                </srcFiles>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.191</version>
        </dependency>
    </dependencies>
</plugin>
```
请注意，这个插件必须放在POM文件中的Properties Maven插件的后面，因为它们的执行目标都绑定到同一个阶段，Maven将按照列出的顺序执行它们。

### 3.4. jOOQ Codegen插件

jOOQ Codegen插件从数据库表结构生成Java代码。其生成目标应该绑定到生成源阶段，以确保正确的执行顺序。插件元数据如下所示：

```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <version>${org.jooq.version}</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <jdbc>
                    <driver>${db.driver}</driver>
                    <url>${db.url}</url>
                    <user>${db.username}</user>
                    <password>${db.password}</password>
                </jdbc>
                <generator>
                    <target>
                        <packageName>com.baeldung.jooq.introduction.db</packageName>
                        <directory>src/main/java</directory>
                    </target>
                </generator>
            </configuration>
        </execution>
    </executions>
</plugin>
```
### 3.5.生成代码

为了完成源代码生成的过程，我们需要运行Maven生成源阶段。在Eclipse中，我们可以通过右键单击项目并选择Run As - > Maven generate-sources来执行此操作。命令完成后，将生成与作者，书，author_book表（以及其他几个支持类）对应的源文件。

让我们来看一下表类，看看jOOQ的产生。每个类都有一个与该类名相同的静态域，除了名称中的所有字母均大写。以下是从生成的类的定义中获取的代码段


The Author class:

```java

public class Author extends TableImpl<AuthorRecord> {
    public static final Author AUTHOR = new Author();
 
    // other class members
}

```
The Book class:

```java
public class Book extends TableImpl<BookRecord> {
    public static final Book BOOK = new Book();
 
    // other class members
}
```


The AuthorBook class:

```java
public class AuthorBook extends TableImpl<AuthorBookRecord> {
    public static final AuthorBook AUTHOR_BOOK = new AuthorBook();
 
    // other class members
}
```


当使用项目中的其他图层时，这些静态字段引用的实例将用作数据访问对象来表示相应的表。

## 4. Spring Configuration

### 4.1. Translating jOOQ Exceptions to Spring

为了使从jOOQ执行引发的异常与Spring对数据库访问的支持保持一致，我们需要将它们转换为DataAccessException类的子类型。

让我们定义一个ExecuteListener接口的实现来转换异常：

```java
public class ExceptionTranslator extends DefaultExecuteListener {
    public void exception(ExecuteContext context) {
        SQLDialect dialect = context.configuration().dialect();
        SQLExceptionTranslator translator 
          = new SQLErrorCodeSQLExceptionTranslator(dialect.name());
        context.exception(translator
          .translate("Access database using jOOQ", context.sql(), context.sqlException()));
    }
}
```

这个类将被Spring应用程序上下文使用。


### 4.2. Configuring Spring

本节将介绍定义一个PersistenceContext的步骤，该PersistenceContext包含要在Spring应用程序上下文中使用的元数据和bean。

我们开始使用必要的注释到类：
- @Configuration：使类被识别为bean的容器
- @ComponentScan：配置扫描指令，包括值选项来声明一个包名称数组以搜索组件。在本教程中，要搜索的包是由jOOQ Codegen Maven插件生成的包
- @EnableTransactionManagement：启用事务由Spring管理
- @PropertySource：指示要加载的属性文件的位置。本文中的值指向包含数据库的配置数据和方言的文件，它恰好是第4.1节中提到的相同文件。


```java
@Configuration
@ComponentScan({"com.baeldung.jooq.introduction.db.public_.tables"})
@EnableTransactionManagement
@PropertySource("classpath:intro_config.properties")
public class PersistenceContext {
    // Other declarations
}
```


接下来，使用Environment对象获取配置数据，然后用于配置DataSource bean：


```java
@Autowired
private Environment environment;
 
@Bean
public DataSource dataSource() {
    JdbcDataSource dataSource = new JdbcDataSource();
 
    dataSource.setUrl(environment.getRequiredProperty("db.url"));
    dataSource.setUser(environment.getRequiredProperty("db.username"));
    dataSource.setPassword(environment.getRequiredProperty("db.password"));
    return dataSource; 
}
```

现在我们定义几个bean来处理数据库访问操作：


```java


@Bean
public TransactionAwareDataSourceProxy transactionAwareDataSource() {
    return new TransactionAwareDataSourceProxy(dataSource());
}
 
@Bean
public DataSourceTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSource());
}
 
@Bean
public DataSourceConnectionProvider connectionProvider() {
    return new DataSourceConnectionProvider(transactionAwareDataSource());
}
 
@Bean
public ExceptionTranslator exceptionTransformer() {
    return new ExceptionTranslator();
}
     
@Bean
public DefaultDSLContext dsl() {
    return new DefaultDSLContext(configuration());
}

```


最后，我们提供一个jOOQ配置实现，并将其声明为一个Spring bean供DSLContext类使用：

```java

@Bean
public DefaultConfiguration configuration() {
    DefaultConfiguration jooqConfiguration = new DefaultConfiguration();
    jooqConfiguration.set(connectionProvider());
    jooqConfiguration.set(new DefaultExecuteListenerProvider(exceptionTransformer()));
 
    String sqlDialectName = environment.getRequiredProperty("jooq.sql.dialect");
    SQLDialect dialect = SQLDialect.valueOf(sqlDialectName);
    jooqConfiguration.set(dialect);
 
    return jooqConfiguration;
}
```

## 5. Using jOOQ with Spring

本节演示了在通用数据库访问查询中使用jOOQ。对于每种类型的“写入”操作，包括插入，更新和删除数据，都有两个测试，一个用于提交，一个用于回滚。当选择数据以验证“写入”查询时，会显示使用“读取”操作。

我们将首先声明一个自动连线的DSLContext对象，并且所有测试方法都使用jOOQ生成的类的实例：

```java

@Autowired
private DSLContext dsl;
 
Author author = Author.AUTHOR;
Book book = Book.BOOK;
AuthorBook authorBook = AuthorBook.AUTHOR_BOOK;
```

### 5.1. Inserting Data

第一步是将数据插入表中:


```java
dsl.insertInto(author)
  .set(author.ID, 4)
  .set(author.FIRST_NAME, "Herbert")
  .set(author.LAST_NAME, "Schildt")
  .execute();
dsl.insertInto(book)
  .set(book.ID, 4)
  .set(book.TITLE, "A Beginner's Guide")
  .execute();
dsl.insertInto(authorBook)
  .set(authorBook.AUTHOR_ID, 4)
  .set(authorBook.BOOK_ID, 4)
  .execute();
```

一个SELECT查询来提取数据：


```java
Result<Record3<Integer, String, Integer>> result = dsl
  .select(author.ID, author.LAST_NAME, DSL.count())
  .from(author)
  .join(authorBook)
  .on(author.ID.equal(authorBook.AUTHOR_ID))
  .join(book)
  .on(authorBook.BOOK_ID.equal(book.ID))
  .groupBy(author.LAST_NAME)
  .fetch();
```


上述查询产生以下输出：

```$xslt
+----+---------+-----+
|  ID|LAST_NAME|count|
+----+---------+-----+
|   1|Sierra   |    2|
|   2|Bates    |    1|
|   4|Schildt  |    1|
+----+---------+-----+
```

结果由Assert API证实：

```java
assertEquals(3, result.size());
assertEquals("Sierra", result.getValue(0, author.LAST_NAME));
assertEquals(Integer.valueOf(2), result.getValue(0, DSL.count()));
assertEquals("Schildt", result.getValue(2, author.LAST_NAME));
assertEquals(Integer.valueOf(1), result.getValue(2, DSL.count()));
```

当由于无效的查询而发生故障时，抛出异常并且事务回滚。在以下示例中，INSERT查询违反外键约束，导致异常：



```java

@Test(expected = DataAccessException.class)
public void givenInvalidData_whenInserting_thenFail() {
    dsl.insertInto(authorBook)
      .set(authorBook.AUTHOR_ID, 4)
      .set(authorBook.BOOK_ID, 5)
      .execute();
}
```


### 5.2. Updating Data

现在我们来更新现有的数据：


```java

dsl.update(author)
  .set(author.LAST_NAME, "Baeldung")
  .where(author.ID.equal(3))
  .execute();
dsl.update(book)
  .set(book.TITLE, "Building your REST API with Spring")
  .where(book.ID.equal(3))
  .execute();
dsl.insertInto(authorBook)
  .set(authorBook.AUTHOR_ID, 3)
  .set(authorBook.BOOK_ID, 3)
  .execute();
```

获取必要的数据：

```java
Result<Record3<Integer, String, String>> result = dsl
  .select(author.ID, author.LAST_NAME, book.TITLE)
  .from(author)
  .join(authorBook)
  .on(author.ID.equal(authorBook.AUTHOR_ID))
  .join(book)
  .on(authorBook.BOOK_ID.equal(book.ID))
  .where(author.ID.equal(3))
  .fetch();
```
输出应为：

```$xslt
+----+---------+----------------------------------+
|  ID|LAST_NAME|TITLE                             |
+----+---------+----------------------------------+
|   3|Baeldung |Building your REST API with Spring|
+----+---------+----------------------------------+
```
以下测试将验证jOOQ是否按预期工作：

```java
assertEquals(1, result.size());
assertEquals(Integer.valueOf(3), result.getValue(0, author.ID));
assertEquals("Baeldung", result.getValue(0, author.LAST_NAME));
assertEquals("Building your REST API with Spring", result.getValue(0, book.TITLE));
```

如果发生故障，将抛出异常并且事务回滚，我们通过测试确认：

```java

@Test(expected = DataAccessException.class)
public void givenInvalidData_whenUpdating_thenFail() {
    dsl.update(authorBook)
      .set(authorBook.AUTHOR_ID, 4)
      .set(authorBook.BOOK_ID, 5)
      .execute();
}
```



### 5.3. Deleting Data

以下方法删除一些数据：


```java
dsl.delete(author)
  .where(author.ID.lt(3))
  .execute();
```

这是查看受影响的表：

```java
Result<Record3<Integer, String, String>> result = dsl
  .select(author.ID, author.FIRST_NAME, author.LAST_NAME)
  .from(author)
  .fetch();
```
查询输出：

```$xslt

+----+----------+---------+
|  ID|FIRST_NAME|LAST_NAME|
+----+----------+---------+
|   3|Bryan     |Basham   |
+----+----------+---------+
```


以下测试验证删除：


```java
assertEquals(1, result.size());
assertEquals("Bryan", result.getValue(0, author.FIRST_NAME));
assertEquals("Basham", result.getValue(0, author.LAST_NAME));
```

另一方面，如果查询无效，它将抛出异常并且事务回滚。以下测试将证明：

```java
@Test(expected = DataAccessException.class)
public void givenInvalidData_whenDeleting_thenFail() {
    dsl.delete(book)
      .where(book.ID.equal(1))
      .execute();
}
```

## 6. 总结


本教程介绍了jOOQ的基础知识，jOOQ是一个用于处理数据库的Java库。它涵盖了从数据库结构生成源代码的步骤，以及如何使用新创建的类与该数据库进行交互。


