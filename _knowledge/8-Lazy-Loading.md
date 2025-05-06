---
layout: default
title: Lazy Loading
topic: Topic 8
---

## 1. What is Lazy Fetching?
Lazy Fetching: also known as Lazy Loading, is a fetching strategy in Hibernate (and other ORM frameworks) where the associated or related data is not loaded from the database until it is explicitly accessed. This approach helps optimize performance by avoiding unnecessary data retrieval. 

**Key Characteristics:**
1. The initial query only retrieves the main entity, and related entities are fetched later when accessed. 
2. It reduces memory consumption and improves performance when the associated entities are not always needed.
3. Lazy loading is the default fetching strategy for many associations in Hibernate, such as @ManyToOne or @OneToMany.

## 2. What is LazyInitializationException
LazyInitializationException is an exception which occurs in Hibernate when we try to access a lazily loaded entity or collection after the Hibernate Session that was responsible to loading the entity has already beed closed. 

## 3. How to deal with LazyInitializationException?

There are several approaches to handle the LazyInitializationException:

**1. Open Session in View Patten (OSIV)**
    
1. OSIV pattern keeps the Hibernate session open for the entire duration of a web request so that the lazy loading ban happen anytime during the request. Spring Boot can enable this pattern by default with spring.jpa.open-in-view-true in application.properties.

2. Pros and Cons: OSIV allows lazy loading throughout the view rendering process. However, it keep the session open longer, potentially leading to performance issues like too many database connections or memory leaks in long-running requests.

**2. Eager Fetching**

1. User eager fetching by specifying FetchType.EAGER in out entity mappings. This forces Hibernate to load all related entities immediately with the main entity.

2. Pros and Cons: Eager Fetching avoids LazyInitializationException since the related entities are fetched upfront. However, it can lead to performance issues like the N+1 Select Problem, where multiple queries are fires for related entities even when not needed.

**3. Dynamic Fetching**

1. Dynamic Fetching allows for adjusting the behavior on a per-query basis, overriding the default fetch strategy dynamically at runtime. We can implement dynamic fetching using HQL with JOIN FETCH clause to load related entities eagerly, even if they are mapped as lazy, like: 

	```
	// Default fetch type is lazy for orders in the User entity
	Query query = session.createQuery("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :userId");
	query.setParameter("userId", 1L);
	User user = query.getSingleResult();
	```

2. Pros and Cons: Dynamic Fetching allows fine-grained control over fetching behavior without changing entity mappings and helps optimize performance by reducing the number of queries. However, Dynamic Fetching requires explicit query customization for each scenario, adding complexity to the code. Overuse of dynamic fetching can lead to unexpected performance issues if not carefully managed. 

**4. Iterating Through the Associated Data Before the Transaction is Closed**

1. Another way to deal with LazyInitializationException is to iterate through or access all lazily loaded associations while the transaction is still open, which ensures that the related entities are loaded before the Session or transaction is closed.

2. Pros and Cons: This approach is simple and ensures all necessary data is loaded before the session is closed. However, this approach requires developers to be aware of which data is lazily loaded and to manually access or iterate through it, which can introduce potential code clutter.
