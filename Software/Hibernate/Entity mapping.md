https://docs.jboss.org/hibernate/orm/6.6/introduction/html_single/Hibernate_Introduction.html#organizing-persistence
### Table 16. Declaring Entities and Embeddable Types

| Annotation        | Purpose                                            | JPA-standard |
|-------------------|----------------------------------------------------|--------------|
| @Entity           | Declare an entity class                            | ✔            |
| @MappedSuperclass | Declare a non-entity class with mapped attributes inherited by an entity | ✔            |
| @Embeddable       | Declare an embeddable type                         | ✔            |
| @IdClass          | Declare the identifier class for an entity with multiple @Id attributes | ✔            |

### Table 17. Declaring Basic and Embedded Attributes

| Annotation         | Purpose                                                               | JPA-standard |     |
| ------------------ | --------------------------------------------------------------------- | ------------ | --- |
| @Id                | Declare a basic-typed identifier attribute                            | ✔            |     |
| @Version           | Declare a version attribute                                           | ✔            |     |
| @Basic             | Declare a basic attribute                                             | ✔            |     |
| @EmbeddedId        | Declare an embeddable-typed identifier attribute                      | ✔            |     |
| @Embedded          | Declare an embeddable-typed attribute                                 | Inferred     | ✔   |
| @Enumerated        | Declare an enum-typed attribute and specify how it is encoded         | Inferred     | ✔   |
| @Array             | Declare that an attribute maps to a SQL ARRAY, and specify the length | Inferred     | ✖   |
| @ElementCollection | Declare that a collection is mapped to a dedicated table              | ✔            |     |

### Table 18. Converters and Compositional Basic Types

| Annotation           | Purpose                                                        | JPA-standard |
|----------------------|----------------------------------------------------------------|--------------|
| @Converter           | Register an AttributeConverter                                 | ✔            |
| @Convert             | Apply a converter to an attribute                              | ✔            |
| @JavaType            | Explicitly specify an implementation of JavaType for a basic attribute | ✖            |
| @JdbcType            | Explicitly specify an implementation of JdbcType for a basic attribute | ✖            |
| @JdbcTypeCode        | Explicitly specify a JDBC type code used to determine the JdbcType for a basic attribute | ✖            |
| @JavaTypeRegistration | Register a JavaType for a given Java type                      | ✖            |
| @JdbcTypeRegistration | Register a JdbcType for a given JDBC type code                 | ✖            |

### Table 19. System-Generated Identifiers

| Annotation         | Purpose                                                   | JPA-standard |
|--------------------|-----------------------------------------------------------|--------------|
| @GeneratedValue    | Specify that an identifier is system-generated             | ✔            |
| @SequenceGenerator | Define an id generated backed by on a database sequence    | ✔            |
| @TableGenerator    | Define an id generated backed by a database table          | ✔            |
| @IdGeneratorType   | Declare an annotation that associates a custom Generator with each @Id attribute it annotates | ✖            |
| @ValueGenerationType | Declare an annotation that associates a custom Generator with each @Basic attribute it annotates | ✖            |

### Table 20. Declaring Entity Associations

| Annotation | Purpose | JPA-standard |
|------------|---------|--------------|
| @ManyToOne | Declare the single-valued side of a many-to-one association (the owning side) | ✔ |
| @OneToMany | Declare the many-valued side of a many-to-one association (the unowned side) | ✔ |
| @ManyToMany | Declare either side of a many-to-many association | ✔ |
| @OneToOne | Declare either side of a one-to-one association | ✔ |
| @MapsId | Declare that the owning side of a @OneToOne association maps the primary key column | ✔ |
