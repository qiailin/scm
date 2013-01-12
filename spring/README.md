2. New Features and Enhancements in Spring Framework 3.0

If you have been using the Spring Framework for some time, you will be aware that Spring has undergone two major revisions: Spring 2.0, released in October 2006, and Spring 2.5, released in November 2007. It is now time for a third overhaul resulting in Spring Framework 3.0.

Java SE and Java EE Support

The Spring Framework is now based on Java 5, and Java 6 is fully supported.

Furthermore, Spring is compatible with J2EE 1.4 and Java EE 5, while at the same time introducing some early support for Java EE 6.
2.1 Java 5

The entire framework code has been revised to take advantage of Java 5 features like generics, varargs and other language improvements. We have done our best to still keep the code backwards compatible. We now have consistent use of generic Collections and Maps, consistent use of generic FactoryBeans, and also consistent resolution of bridge methods in the Spring AOP API. Generic ApplicationListeners automatically receive specific event types only. All callback interfaces such as TransactionCallback and HibernateCallback declare a generic result value now. Overall, the Spring core codebase is now freshly revised and optimized for Java 5.

Spring's TaskExecutor abstraction has been updated for close integration with Java 5's java.util.concurrent facilities. We provide first-class support for Callables and Futures now, as well as ExecutorService adapters, ThreadFactory integration, etc. This has been aligned with JSR-236 (Concurrency Utilities for Java EE 6) as far as possible. Furthermore, we provide support for asynchronous method invocations through the use of the new @Async annotation (or EJB 3.1's @Asynchronous annotation).
2.2 Improved documentation

The Spring reference documentation has also substantially been updated to reflect all of the changes and new features for Spring Framework 3.0. While every effort has been made to ensure that there are no errors in this documentation, some errors may nevertheless have crept in. If you do spot any typos or even more serious errors, and you can spare a few cycles during lunch, please do bring the error to the attention of the Spring team by raising an issue.
2.3 New articles and tutorials

There are many excellent articles and tutorials that show how to get started with Spring Framework 3 features. Read them at the Spring Documentation page.

The samples have been improved and updated to take advantage of the new features in Spring Framework 3. Additionally, the samples have been moved out of the source tree into a dedicated SVN repository available at:

https://anonsvn.springframework.org/svn/spring-samples/

As such, the samples are no longer distributed alongside Spring Framework 3 and need to be downloaded separately from the repository mentioned above. However, this documentation will continue to refer to some samples (in particular Petclinic) to illustrate various features.
[Note]  Note

For more information on Subversion (or in short SVN), see the project homepage at: http://subversion.apache.org/
2.4 New module organization and build system

The framework modules have been revised and are now managed separately with one source-tree per module jar:

    org.springframework.aop

    org.springframework.beans

    org.springframework.context

    org.springframework.context.support

    org.springframework.expression

    org.springframework.instrument

    org.springframework.jdbc

    org.springframework.jms

    org.springframework.orm

    org.springframework.oxm

    org.springframework.test

    org.springframework.transaction

    org.springframework.web

    org.springframework.web.portlet

    org.springframework.web.servlet

    org.springframework.web.struts

Note:

The spring.jar artifact that contained almost the entire framework is no longer provided.

We are now using a new Spring build system as known from Spring Web Flow 2.0. This gives us:

    Ivy-based "Spring Build" system

    consistent deployment procedure

    consistent dependency management

    consistent generation of OSGi manifests

2.5 Overview of new features

This is a list of new features for Spring Framework 3.0. We will cover these features in more detail later in this section.

    Spring Expression Language

    IoC enhancements/Java based bean metadata

    General-purpose type conversion system and field formatting system

    Object to XML mapping functionality (OXM) moved from Spring Web Services project

    Comprehensive REST support

    @MVC additions

    Declarative model validation

    Early support for Java EE 6

    Embedded database support

2.5.1 Core APIs updated for Java 5

BeanFactory interface returns typed bean instances as far as possible:

    T getBean(Class<T> requiredType)

    T getBean(String name, Class<T> requiredType)

    Map<String, T> getBeansOfType(Class<T> type)

Spring's TaskExecutor interface now extends java.util.concurrent.Executor:

    extended AsyncTaskExecutor supports standard Callables with Futures

New Java 5 based converter API and SPI:

    stateless ConversionService and Converters

    superseding standard JDK PropertyEditors

Typed ApplicationListener<E>
2.5.2 Spring Expression Language

Spring introduces an expression language which is similar to Unified EL in its syntax but offers significantly more features. The expression language can be used when defining XML and Annotation based bean definitions and also serves as the foundation for expression language support across the Spring portfolio. Details of this new functionality can be found in the chapter Spring Expression Language (SpEL).

The Spring Expression Language was created to provide the Spring community a single, well supported expression language that can be used across all the products in the Spring portfolio. Its language features are driven by the requirements of the projects in the Spring portfolio, including tooling requirements for code completion support within the Eclipse based SpringSource Tool Suite.

The following is an example of how the Expression Language can be used to configure some properties of a database setup

<bean class="mycompany.RewardsTestDatabase">
    <property name="databaseName"
        value="#{systemProperties.databaseName}"/>
    <property name="keyGenerator"
        value="#{strategyBean.databaseKeyGenerator}"/>
</bean>

This functionality is also available if you prefer to configure your components using annotations:

@Repository
public class RewardsTestDatabase {

    @Value("#{systemProperties.databaseName}")
    public void setDatabaseName(String dbName) { … }

    @Value("#{strategyBean.databaseKeyGenerator}")
    public void setKeyGenerator(KeyGenerator kg) { … }
}

2.5.3 The Inversion of Control (IoC) container
Java based bean metadata

Some core features from the JavaConfig project have been added to the Spring Framework now. This means that the following annotations are now directly supported:

    @Configuration

    @Bean

    @DependsOn

    @Primary

    @Lazy

    @Import

    @ImportResource

    @Value

Here is an example of a Java class providing basic configuration using the new JavaConfig features:

package org.example.config;

@Configuration
public class AppConfig {
    private @Value("#{jdbcProperties.url}") String jdbcUrl;
    private @Value("#{jdbcProperties.username}") String username;
    private @Value("#{jdbcProperties.password}") String password;

    @Bean
    public FooService fooService() {
        return new FooServiceImpl(fooRepository());
    }

    @Bean
    public FooRepository fooRepository() {
        return new HibernateFooRepository(sessionFactory());
    }

    @Bean
    public SessionFactory sessionFactory() {
        // wire up a session factory
        AnnotationSessionFactoryBean asFactoryBean =
            new AnnotationSessionFactoryBean();
        asFactoryBean.setDataSource(dataSource());
        // additional config
        return asFactoryBean.getObject();
    }

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(jdbcUrl, username, password);
    }
}

To get this to work you need to add the following component scanning entry in your minimal application context XML file.

<context:component-scan base-package="org.example.config"/>
<util:properties id="jdbcProperties" location="classpath:org/example/config/jdbc.properties"/>
        

Or you can bootstrap a @Configuration class directly using AnnotationConfigApplicationContext:

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    FooService fooService = ctx.getBean(FooService.class);
    fooService.doStuff();
}

See Section 5.12.2, “Instantiating the Spring container using AnnotationConfigApplicationContext” for full information on AnnotationConfigApplicationContext.
Defining bean metadata within components

@Bean annotated methods are also supported inside Spring components. They contribute a factory bean definition to the container. See Defining bean metadata within components for more information
2.5.4 General purpose type conversion system and field formatting system

A general purpose type conversion system has been introduced. The system is currently used by SpEL for type conversion, and may also be used by a Spring Container and DataBinder when binding bean property values.

In addition, a formatter SPI has been introduced for formatting field values. This SPI provides a simpler and more robust alternative to JavaBean PropertyEditors for use in client environments such as Spring MVC.
2.5.5 The Data Tier

Object to XML mapping functionality (OXM) from the Spring Web Services project has been moved to the core Spring Framework now. The functionality is found in the org.springframework.oxm package. More information on the use of the OXM module can be found in the Marshalling XML using O/X Mappers chapter.
2.5.6 The Web Tier

The most exciting new feature for the Web Tier is the support for building RESTful web services and web applications. There are also some new annotations that can be used in any web application.
Comprehensive REST support

Server-side support for building RESTful applications has been provided as an extension of the existing annotation driven MVC web framework. Client-side support is provided by the RestTemplate class in the spirit of other template classes such as JdbcTemplate and JmsTemplate. Both server and client side REST functionality make use of HttpConverters to facilitate the conversion between objects and their representation in HTTP requests and responses.

The MarshallingHttpMessageConverter uses the Object to XML mapping functionality mentioned earlier.

Refer to the sections on MVC and the RestTemplate for more information.
@MVC additions

A mvc namespace has been introduced that greatly simplifies Spring MVC configuration.

Additional annotations such as @CookieValue and @RequestHeaders have been added. See Mapping cookie values with the @CookieValue annotation and Mapping request header attributes with the @RequestHeader annotation for more information.
2.5.7 Declarative model validation

Several validation enhancements, including JSR 303 support that uses Hibernate Validator as the default provider.
2.5.8 Early support for Java EE 6

We provide support for asynchronous method invocations through the use of the new @Async annotation (or EJB 3.1's @Asynchronous annotation).

JSR 303, JSF 2.0, JPA 2.0, etc
2.5.9 Support for embedded databases

Convenient support for embedded Java database engines, including HSQL, H2, and Derby, is now provided.
