---
layout: default
title: JdbcTemplate vs. Hibernate
topic: Topic 5
---

# JdbcTemplate vs. Hibernate

Both JdbcTemplate and Hibernate are tools used to map Java classes to database tables. JdbcTemplate is an older and less efficitent technique while Hibernate is more popular and efficient. The main differencet between JdbcTemplate and Hibernate are as follows: 

1. Using JdbcTemplate, we need a RowMapper class to manually map each Java entity class to its corresponding database table. When using Hibernate, we don't need the RowMapper class anymore since Hibernate provides automatic ORM by using @Entity(name="table_name") annotation.
2. JdbcTemplate requires database-specific SQL queries which error-prone and time consuming. This also makes the changing of database become complex and disturbs the persistent logis. However, Hibernate provides HQL which actracts most SQL code and is database independent.
3. JdbcTemplate requires boilerplate code for opening/closing connections, managing transactions, handling exceptions, and mapping query results to objects repeativel with less maintainable code. Hibernate reduces boilerplate code through automatic transation management, connection pool, and entity-object mapping.
4. JdbcTemplate has no built-in cacing mechanism and we need to manually implement the caching solutions to improve data fetching performance. Hibernate provides built-in caching, with First-Level cache for a session and Second-Level cache for entire application.


# Connection and Transaction in JdbcTemplate and Hibernate

Both JDBCTemplate and Hibernate can use connection pooling, but neither of them inherently manages connection pools on their own. They rely on an external connection pool to handle the db connections. 
1. JDBCTemplate: configure a DataSource which JDBCTemplate will use to get connections. 
2. Hibernate: implements JPA and integrates with connection pool libraries. Configure connection pool for Hibernate through DataSource or in the hibernate.cfg.xml (or applications.properties in Spring)

**JDBCTemplate without @Transactional**: by default, each call to jdbcTemplate.update() is executed in its own transaction. The underlying database handles the transaction automatically.

**When to use @Transactional**: when we have multiple db operations  (in the same class with @Transactional annotation) which need to be part of a single atomic transaction. This ensures either all operations are committed together, or if an error occurs, all are rolled back. Without @Transactional, each call to jdbcTemplate.update() will open its own transaction, execute the query, and commit the result. This works fine for standalone queries but can lead to inconsistent data if we have multiple queries which depend on each other and one of them fails.

**When to avoid @Transactional**: for single, standalone operations, using @Transactional may not be necessary. 

For many concurrent request with connection pool, each request use a connection and finish the query (transaction is handled under the hood). If the requests is more than the connections, they will be waiting until a connection is available.

For Hibernate, the connection pool also has a maximum pool size and each request need a connection. If number of concurrent requests is more than the pool size, some requests need to wait until a connection becomes available. We can use @Transactional to handle transaction automatically with calling getCurrentSession() method (when using @Transactional in Spring, it is generally not recommended to use openSession() because it defeats the purpose of Spring's automatic transaction and session management):

```
@Service
public class MyService {

    @Autowired
    private SessionFactory sessionFactory;

    @Transactional
    public void saveEntity(MyEntity entity) {
        // getCurrentSession() is automatically bound to the transaction
        Session session = sessionFactory.getCurrentSession();
        
        // Perform operations
        session.save(entity);
        
        // No need to explicitly commit or rollback the transaction. Spring handles it automatically
    }
}
```

We can also use openSession() or getCurrentSession() to manually obtain and handle a transaction without using @Transactional annotation, like: 

```
Session session = null;
Transaction transaction = null;

try {
    session = sessionFactory.getCurrentSession(); // No need to open/close the session manually
    transaction = session.beginTransaction();
    
    // Perform operations
    session.save(entity);
    
    transaction.commit(); // Commit the transaction
} catch (Exception e) {
    if (transaction != null) transaction.rollback(); // Rollback in case of failure
    e.printStackTrace();
    throw e; // Optionally rethrow the exception
}
```


```
Session session = sessionFactory.openSession();
Transaction transaction = null;
try {
    transaction = session.beginTransaction();
    // Perform operations
    session.save(entity);
    transaction.commit();
} catch (Exception e) {
    if (transaction != null) transaction.rollback();
} finally {
    session.close();
}
```

Key Differences Between JDBCTemplate and Hibernate:

| Aspect   | JDBCTemplate | Hibernate |
| Connection	| Uses DataSource for connections (from pool)	| Uses Session (via openSession() or getCurrentSession()) |
| Session/Connection Management	| No explicit session handling, relies on connection pooling | Sessions must be explicitly managed (opened/closed) unless using getCurrentSession() |
Transactions |	Managed via Spring @Transactional or PlatformTransactionManager	Managed via Spring  | @Transactional or manual transaction control with transaction.commit() |
Commit/Rollback/Close	| Typically handled by Spring automatically	 | Manually managed when using openSession(), or by Spring in getCurrentSession() |
