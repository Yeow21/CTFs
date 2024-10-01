
This is probably the easiest for my project:

4.1. Mapping entity inheritance hierarchies
In Entity class inheritance we saw that entity classes may exist within an inheritance hierarchy. There’s three basic strategies for mapping an entity hierarchy to relational tables. Let’s put them in a table, so we can more easily compare the points of difference between them.

Table 21. Entity inheritance mapping strategies
Strategy	Mapping	Polymorphic queries	Constraints	Normalization	When to use it
SINGLE_TABLE

Map every class in the hierarchy to the same table, and uses the value of a discriminator column to determine which concrete class each row represents.

To retrieve instances of a given class, we only need to query the one table.

Attributes declared by subclasses map to columns without NOT NULL constraints. 💀

Any association may have a FOREIGN KEY constraint. 🤓

Subclass data is denormalized. 🧐

Works well when subclasses declare few or no additional attributes.

JOINED

Map every class in the hierarchy to a separate table, but each table only maps the attributes declared by the class itself.

Optionally, a discriminator column may be used.

To retrieve instances of a given class, we must JOIN the table mapped by the class with:

all tables mapped by its superclasses and

all tables mapped by its subclasses.

Any attribute may map to a column with a NOT NULL constraint. 🤓

Any association may have a FOREIGN KEY constraint. 🤓

The tables are normalized. 🤓

The best option when we care a lot about constraints and normalization.

TABLE_PER_CLASS

Map every concrete class in the hierarchy to a separate table, but denormalize all inherited attributes into the table.

To retrieve instances of a given class, we must take a UNION over the table mapped by the class and the tables mapped by its subclasses.

Associations targeting a superclass cannot have a corresponding FOREIGN KEY constraint in the database. 💀💀

Any attribute may map to a column with a NOT NULL constraint. 🤓

Superclass data is denormalized. 🧐

Not very popular.

From a certain point of view, competes with @MappedSuperclass.

The three mapping strategies are enumerated by InheritanceType. We specify an inheritance mapping strategy using the @Inheritance annotation.

For mappings with a discriminator column, we should:

specify the discriminator column name and type by annotating the root entity @DiscriminatorColumn, and

specify the values of this discriminator by annotating each entity in the hierarchy @DiscriminatorValue.

For single table inheritance we always need a discriminator:
```
@Entity
@DiscriminatorColumn(discriminatorType=CHAR, name="kind")
@DiscriminatorValue('P')
class Person { ... }

@Entity
@DiscriminatorValue('A')
class Author { ... }
```


### Mapping entities to tables

```
@Entity
@Table(name="People")
class Person { ... }
```


4.4. Mapping associations to tables
The @JoinTable annotation specifies an association table, that is, a table holding foreign keys of both associated entities. This annotation is usually used with @ManyToMany associations:

```
@Entity
class Book {
    ...

    @ManyToMany
    @JoinTable(name="BooksAuthors")
    Set<Author> authors;

    ...
}
```

But it’s even possible to use it to map a @ManyToOne or @OneToOne association to an association table.
```

@Entity
class Book {
    ...

    @ManyToOne(fetch=LAZY)
    @JoinTable(name="BookPublisher")
    Publisher publisher;

    ...
}
```



| **Constant** | **Value**   | **Description**                                                                                              |
|--------------|-------------|--------------------------------------------------------------------------------------------------------------|
| `DEFAULT`    | 255         | The default length of a `VARCHAR` or `VARBINARY` column when none is explicitly specified.                    |
| `LONG`       | 32600       | The largest column length for a `VARCHAR` or `VARBINARY` that is allowed on every database Hibernate supports.|
| `LONG16`     | 32767       | The maximum length that can be represented using 16 bits (but this length is too large for a `VARCHAR` or `VARBINARY` column for some databases). |
| `LONG32`     | 2147483647  | The maximum length that can be represented using 32 bits.                                                    |


The maximum length for a Java string

We can use these constants in the @Column annotation:
```
@Column(length=LONG)
String text;

@Column(length=LONG32)
byte[] binaryData;
```