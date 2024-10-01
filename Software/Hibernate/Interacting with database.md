
### Interacting with the database

To interact with the database, that is, to execute queries, or to insert, update, or delete data, we need an instance of one of the following objects:

a JPA EntityManager,

a Hibernate Session (extends entitymanager, or

a Hibernate StatelessSession

```
Session session = entityManager.unwrap(Session.class);
```


```
sessionFactory.inTransaction(session -> {
    //do the work
    ...
});
```


| **Method name and parameters** | **Effect**                                                                                                                                                                                     |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `persist(Object)`              | Make a transient object persistent and schedule a SQL insert statement for later execution.                                                                                                    |
| `remove(Object)`               | Make a persistent object transient and schedule a SQL delete statement for later execution.                                                                                                    |
| `merge(Object)`                | Copy the state of a given detached object to a corresponding managed persistent instance and return the persistent object.                                                                     |
| `detach(Object)`               | Disassociate a persistent object from a session without affecting the database.                                                                                                                |
| `clear()`                      | Empty the persistence context and detach all its entities.                                                                                                                                     |
| `flush()`                      | Detect changes made to persistent objects associated with the session and synchronize the database state with the state of the session by executing SQL insert, update, and delete statements. |



| **Kind**     | **Session method**                       | **EntityManager method**            | **Query execution method**                                   |
|--------------|------------------------------------------|-------------------------------------|--------------------------------------------------------------|
| **Selection**| `createSelectionQuery(String, Class)`    | `createQuery(String, Class)`        | `getResultList()`, `getSingleResult()`, or `getSingleResultOrNull()` |
| **Mutation** | `createMutationQuery(String)`            | `createQuery(String)`               | `executeUpdate()`                                             |
