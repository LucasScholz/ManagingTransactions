<#assign project_id="gs-managing-transactions">

# Getting Started: Managing Transactions


What you'll build
-----------------

This guide walks you through the process of wrapping database operations with transactions.


What you'll need
----------------

 - About 15 minutes
 - <@prereq_editor_jdk_buildtools/>


## <@how_to_complete_this_guide jump_ahead='Create a booking service'/>


<a name="scratch"></a>
Set up the project
------------------

<@build_system_intro/>

<@create_directory_structure_hello/>

### Create a Maven POM

    <@snippet path="pom.xml" prefix="complete"/>

<@bootstrap_starter_pom_disclaimer/>


<a name="initial"></a>
Create a booking service
------------------------
First, use the `BookingService` class to create a JDBC-based service that books people into the system by name. 

    <@snippet path="src/main/java/hello/BookingService.java" prefix="complete"/>

The code has an autowired `JdbcTemplate`, a handy template class that does all the database work.

You also have a method aimed at booking multiple people. It loops through the list of people, and for each person, inserts them into the `BOOKINGS` table. This method is tagged with `@Transactional`, meaning that any failure causes the entire operation to rollback to its previous state, and then re-throw the original exception.

You also have a `findAllBookings` method to query the database. Each row fetched from the database is converted into a `String` and then assembled into a `List`.

Build an application
-----------------------
As shown above, `JdbcTemplate` is autowired into `BookingService`, meaning you now need to define it in the `Application` code:

    <@snippet path="src/main/java/hello/Application.java" prefix="complete"/>
    
> **Note:** `SimpleDriverDataSource` is a convenience class and is _not_ intended for production. Also, in production systems, database tables are usually declared outside the application.

The method where you define `JdbcTemplate` also contains some DDL to declare the `BOOKINGS` table.

You also have wired in the `BookingService`.

The `main()` method defers to the [`SpringApplication`][] helper class, providing `Application.class` as an argument to its `run()` method. This tells Spring to read the annotation metadata from `Application` and to manage it as a component in the _[Spring application context][u-application-context]_.

Note two especially valuable parts of this application configuration:
- `@EnableTransactionManagement` activates Spring's seamless transaction features, which makes `@Transactional` function.
- [`@EnableAutoConfiguration`][] switches on reasonable default behaviors based on the content of your classpath. For example, it detects that you have **spring-tx** on the path as well as a `DataSource`, and automatically creates the other beans needed for transactions. Auto-configuration is a powerful, flexible mechanism. See the [API documentation][`@EnableAutoConfiguration`] for further details.


<@build_an_executable_jar/>

<@run_the_application/>

You should see the following output:

```sh
Creating tables
Booking Alice in a seat...
Booking Bob in a seat...
Booking Carol in a seat...
Booking Chris in a seat...
Booking Samuel in a seat...
Jul 11, 2013 10:20:14 AM org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
INFO: Loading XML bean definitions from class path resource [org/springframework/jdbc/support/sql-error-codes.xml]
Jul 11, 2013 10:20:14 AM org.springframework.jdbc.support.SQLErrorCodesFactory <init>
INFO: SQLErrorCodes loaded: [DB2, Derby, H2, HSQL, Informix, MS-SQL, MySQL, Oracle, PostgreSQL, Sybase]
PreparedStatementCallback; SQL [insert into BOOKINGS(FIRST_NAME) values (?)]; Value too long for column "FIRST_NAME VARCHAR(5) NOT NULL": "'Samuel' (6)"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [22001-171]; nested exception is org.h2.jdbc.JdbcSQLException: Value too long for column "FIRST_NAME VARCHAR(5) NOT NULL": "'Samuel' (6)"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [22001-171]
Booking Buddy in a seat...
Booking null in a seat...
PreparedStatementCallback; SQL [insert into BOOKINGS(FIRST_NAME) values (?)]; NULL not allowed for column "FIRST_NAME"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [23502-171]; nested exception is org.h2.jdbc.JdbcSQLException: NULL not allowed for column "FIRST_NAME"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [23502-171]
```

The `BOOKINGS` table has two constraints on the **first_name** column.
- Names cannot be longer than five characters.
- Names cannot be null.

The first three names inserted are **Alice**, **Bob**, and **Carol**. The application asserts that three people were added to that table. If that had not worked, the application would have exited early.

Next, another booking is done for **Chris** and **Samuel**. Samuel's name is deliberately too long, forcing an insert error. Transactional behavior stipulates that both Chris and Samuel; that is, this transaction, should be rolled back. Thus there should still be only three people in that table, which the assertion demonstrates.

Finally, **Buddy** and **null** are booked. As the output shows, null causes a rollback as well, leaving the same three people booked.

Summary
-------
Congratulations! You've just used Spring to develop a simple JDBC application wrapped with non-intrusive transactions.

[u-application-context]: /understanding/application-context
[`SpringApplication`]: http://static.springsource.org/spring-bootstrap/docs/0.5.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/bootstrap/SpringApplication.html
[`@EnableAutoConfiguration`]: http://static.springsource.org/spring-bootstrap/docs/0.5.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/bootstrap/context/annotation/SpringApplication.html

