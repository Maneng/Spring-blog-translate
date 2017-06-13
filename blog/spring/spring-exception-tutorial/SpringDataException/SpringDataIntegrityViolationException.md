# SpringDataIntegrityViolationException

## **1.概述**

在这片文章中，我们会讨论关于Spring的异常：
> Spring org.springframework.beans.factory.DataIntegrityViolationException   

这是处理较低级持久异常时通常由Spring异常转换机制抛出的通用数据异常。

本文将讨论这种异常的最常见原因以及每个解决方案的解决方案


## **2.DataIntegrityViolationException and Spring Exception Translation**

Spring异常转换机制可以透明地应用于用**@Repository**注释的所有bean - 通过在Context中定义一个异常转换bean后处理器bean

通过在Context中定义一个异常转换bean后处理器bean：

``` javascript
<bean id="persistenceExceptionTranslationPostProcessor"
   class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />
```

Or in Java:


``` java
@Configuration
public class PersistenceHibernateConfig{
   @Bean
   public PersistenceExceptionTranslationPostProcessor exceptionTranslation(){
      return new PersistenceExceptionTranslationPostProcessor();
   }
}
```
默认情况下，在Spring中可用的较旧的持久性模板--HibernateTemplate，JpaTemplate等中也启用异常转换机制。





## **3.Where is DataIntegrityViolationException thrown**


### 3.1. DataIntegrityViolationException with Hibernate

当Spring配置为Hibernate时，异常会抛出**Spring-SessionFactoryUtils**提供的异常转换层 - **convertHibernateAccessException**。

有三种可能的Hibernate异常可能会导致抛出**DataIntegrityViolationException**：

- org.hibernate.exception.ConstraintViolationException
- org.hibernate.PropertyValueException
- org.hibernate.exception.DataException

### 3.2. DataIntegrityViolationException With JPA

当Spring配置为JPA作为其持久性提供程序时，**DataIntegrityViolationException**将抛出异常转换层中的Hibernate类，即**EntityManagerFactoryUtils** - **convertJpaAccessExceptionIfPossible**。

有一个JPA异常可能会触发要抛出的**DataIntegrityViolationException** - **javax.persistence.EntityExistsException**。




## **4. Cause: org.hibernate.exception.ConstraintViolationException**

这是迄今为止抛出**DataIntegrityViolationException**的最常见原因-**-Hibernate ConstraintViolationException**表示操作违反了数据库完整性约束。

考虑以下示例 - 对于通过Parent和Child实体之间的显式外键列进行一对一映射，以下操作将失败：

``` java
@Test(expected = DataIntegrityViolationException.class)
public void whenChildIsDeletedWhileParentStillHasForeignKeyToIt_thenDataException() {
   Child childEntity = new Child();
   childService.create(childEntity);
 
   Parent parentEntity = new Parent(childEntity);
   service.create(parentEntity);
 
   childService.delete(childEntity);
}
```
父实体具有子实体的外键 - 因此删除子代将破坏父对象上的外键约束，从而导致**ConstraintViolationException**由Spring在DataIntegrityViolationException中包装：

``` sql
org.springframework.dao.DataIntegrityViolationException: 
could not execute statement; SQL [n/a]; constraint [null]; 
nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement
    at o.s.orm.h.SessionFactoryUtils.convertHibernateAccessException(SessionFactoryUtils.java:138)
Caused by: org.hibernate.exception.ConstraintViolationException: could not execute statement
```


要解决这个问题，应该首先删除父项：

``` java
@Test
public void whenChildIsDeletedAfterTheParent_thenNoExceptions() {
   Child childEntity = new Child();
   childService.create(childEntity);
 
   Parent parentEntity = new Parent(childEntity);
   service.create(parentEntity);
 
   service.delete(parentEntity);
   childService.delete(childEntity);
}
```


## **5.Cause: org.hibernate.PropertyValueException**

这是**DataIntegrityViolationException**的更常见的一个原因- 在Hibernate中，这将导致一个持续存在问题的实体。实体有一个null属性，它用非空约束来定义。
或者实体的关联可能引用了未保存的瞬时实例。 （unsaved, transient instance.）

例如，以下实体具有非空名称属性 - 


``` java
@Entity
public class Foo {
   ...
 
   @Column(nullable = false)
   private String name;
 
   ...
}
```

如果以下测试尝试使用空值持久化实体名称：

``` java
@Test(expected = DataIntegrityViolationException.class)
public void whenInvalidEntityIsCreated_thenDataException() {
   fooService.create(new Foo());
}
```
违反数据库集成约束，因此抛出**DataIntegrityViolationException**：

``` smali
org.springframework.dao.DataIntegrityViolationException: 
not-null property references a null or transient value: 
org.baeldung.spring.persistence.model.Foo.name; 
nested exception is org.hibernate.PropertyValueException: 
not-null property references a null or transient value: 
org.baeldung.spring.persistence.model.Foo.name
    at o.s.orm.h.SessionFactoryUtils.convertHibernateAccessException(SessionFactoryUtils.java:160)
...
Caused by: org.hibernate.PropertyValueException: 
not-null property references a null or transient value: 
org.baeldung.spring.persistence.model.Foo.name
    at o.h.e.i.Nullability.checkNullability(Nullability.java:103)
```


## **6.Cause: org.hibernate.exception.DataException**

**Hibernate DataException**表示无效的SQL语句 - 在该特定上下文中，该语句或数据出错。例如，使用之前的Foo实体，以下将触发此异常：


``` java
@Test(expected = DataIntegrityViolationException.class)
public final void whenEntityWithLongNameIsCreated_thenDataException() {
   service.create(new Foo(randomAlphabetic(2048)));
}
```
持久化数据类型为long的对象的实际异常是：

``` sql
org.springframework.dao.DataIntegrityViolationException: 
could not execute statement; SQL [n/a]; 
nested exception is org.hibernate.exception.DataException: could not execute statement
   at o.s.o.h.SessionFactoryUtils.convertHibernateAccessException(SessionFactoryUtils.java:143)
...
Caused by: org.hibernate.exception.DataException: could not execute statement
    at o.h.e.i.SQLExceptionTypeDelegate.convert(SQLExceptionTypeDelegate.java:71)
```
在这个特定的例子中，解决方案是指定名称的最大长度：

``` stylus
@Column(nullable = false, length = 4096)
```


## **7.Cause: javax.persistence.EntityExistsException**

与Hibernate相似的是，**EntityExistsException**  JPA异常也会被Spring异常转换包装**成DataIntegrityViolationException**。唯一的区别是，JPA本身已经是高级别，这使得JPA异常成为数据完整性违规的唯一潜在原因。


## **8.Potentially DataIntegrityViolationException**

在某些可能需要**DataIntegrityViolationException**的情况下，可能会抛出另一个异常 - 如果类路径中存在一个JSR-303验证器（如**hibernate-validator** 4或5），则会出现这种情况。
在这种情况下，如果以下实体用名称的空值持久化，则不再会由持久层触发的数据完整性违例失败：


``` java
@Entity
public class Foo {
    ...
    @Column(nullable = false)
    @NotNull
    private String name;
 
    ...
}
```

这是因为执行不会到达持久层 - 它会在使用**javax.validation.ConstraintViolationException**之前失败：

**javax.validation.ConstraintViolationException: 
Validation failed for classes [org.baeldung.spring.persistence.model.Foo] 
during persist time for groups [javax.validation.groups.Default, ]
List of constraint violations:[ ConstraintViolationImpl{
    interpolatedMessage='may not be null', propertyPath=name, 
    rootBeanClass=class org.baeldung.spring.persistence.model.Foo, 
    messageTemplate='{javax.validation.constraints.NotNull.message}'}
]
    at o.h.c.b.BeanValidationEventListener.validate(BeanValidationEventListener.java:159)
    at o.h.c.b.BeanValidationEventListener.onPreInsert(BeanValidationEventListener.java:94)**




## **5.总结 …**

在本文末尾，我们应该有一个清晰的思路来浏览可能导致Spring中**DataIntegrityViolationException**异常的原因和问题，以及如何解决所有这些问题。

在[Github](https://github.com/eugenp/tutorials/tree/master/spring-all)项目中可以找到一些这些例外示例的实现 
这是一个基于Eclipse的项目，所以应该很容易导入和运行。
