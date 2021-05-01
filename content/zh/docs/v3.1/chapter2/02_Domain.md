---
type: docs
title: "领域模型层"
linkTitle: "领域模型层"
weight: 3
description: "本文介绍领域模型层"
---

领域模层使用JPA作为领域模型的实现，在此基础上，也可通过增加接口的方式，先定义领域模型应具备的功能，再将JPA作为该功能的一个实现方式，这里不强求在领域模型上使用接口，但需要保证领域模型为标准的POJO对象，并通过annotation的注解，使JPA能识别该POJO为领域模型。

领域模型作为整个开发数据流的底层，肩负着最终持久化到数据库的责任，同时，为了实现数据的简单、一致的原则，在由Domain层通过Repository、Service、Controller层之后，可能直接在界面层中展现，所以要求领域模型能最大限度的反映业务需求，包括父子结构、Blob的处理、使用关联还是不使用关联而直接存取引用字段等都是需要考量的范畴。

除此之外，macula开发平台对领域模型层的实现，有如下的建议：

### 主键策略

领域模型层的主键，如无特别的需求，主键策略按照数据库的情况（当前情况下，我们可认为数据库已定为Oracle），那么对于Oracle的数据库，可定义成序列的自增长方式。

* 逻辑主键要求

  macula开发平台要求领域模型以逻辑主键的方式定义主键，这样能保证JPA能正常处理数据，以及在数据发生关联时，不至于修改了业务主键后，更新大量的关联表。

* 主键类型定义为Long

  在无特殊要求的情况下，尽量将主键定义为Long型，并继承AbstractPersistable

* 主键关联与映射

  在领域模型中，需要使用关联时，使用主键关联，并将主键映射为相应的数据库字段（例如Oracle可影射为NUMBER\(15\)），注意保证主键的长度，以适应业务需求量的变化。


### 数据审计

针对业务中，大量出现的需要记录数据最后变更人和变更时间的需求，这里，在领域模型层，可通过是业务模型类直接继承AbstractAuditable的方式，实现变化日志的自动记录。

在使用AbstractAuditable是，需要注意映射表字段的关系：

* createdBy：创建人，通常只在数据新增时写入，映射字段为CREATED\_BY，该字段一般情况下不能修改，记录数据创建人。

* createdDate：创建时间，与createBy相同，映射字段为CREATED\_DATE，记录数据创建时间。

* lastModifiedBy：最后更新人，通常在数据新增和修改时变化，映射字段为LAST\_MODIFIED\_BY，记录数据最后更新人。

* lastModifiedDate：最后更新时间，与lastModifiedBy相同，映射字段为LAST\_MODIFIED\_DATE，用来记录数据最后更新时间。


_**重要**_

_以上四个字段，均不能为空。_

### 变化日志

数据变更日志采用单独的日指标进行记录，业务系统中，涉及到的数据变更日志均记录在同一数据表中，变更日志的数据表记录方式如下：

* 变化表：tableName
* 变化字段：columnName
* 变化数据ID：dataId
* 变化前的数据：oldValue
* 变化后的数据：newValue
* 变化的批次：batchNo

用户操作所产生的一次数据变化，可能导致一各实体的多个字段值变化，则记录多条变化日志，分别对应变化的字段，但对于该次变化的批次，将是一样的。

_**重要**_

_批次的值可以是用户输入的数据，也可以是程序自动生成的值，默认情况下，采用系统生成的值，它用来标识单次变化时，所变化的数据范围，便于变化日志的展示和查看等。_

对于需要记录变化日志的实体，需要在实体的类上添加@Auditable注解，然后对于需要记录变更日志的字段添加@Auditable注解。同时，EntityManagerFactory需要添加相应的监听器：

```xml
<property name="jpaProperties">
    <props>
        <prop key="hibernate.ejb.event.post-update">
            org.macula.core.hibernate.audit.AuditedEventListener
        </prop>
    </props>
</property>
```

上述设置后，当产生实体变更时系统会发出AuditChangedEvent事件，我们可以通过继承AbstractAuditChangedListener来监听AuditChangedEvent事件，并对变更数据做后续的进一步处理。

### 领域模型接口

在领域模型层，macula开发平台并没有过多的要求，只需要领域模型满足POJO规范，并实现了JPA规范要求的即可，这里列出为方便领域模型层的编写，有下列辅助类可以使用：

* AbstractPersistable：用来标识领域模型是可持久化的，并可通过泛型传入主键类型（一般情况下都是Long），这样可减少编写Id的工作量。

* AbstractAuditable：用来标识领域模型是可审计的，即增加了审计的四个字段。

* AuditChanged：用来记录变更的数据日志。


### 领域模型数据及关联定义原则

在定义领域模型之间的关系时，考虑到性能问题，可遵循如下原则：

* 减少Blob与Clob类型字段

  应尽量减少领域模型中Blob与Clob字段的定义，如果不可避免，应提供不含该Blob/Clob字段的构造，以方便在列表查询时，不用查询出该字段列，减少构建领域模型所需的内存，特别是当所载入的字段列数据并不关心时。

  如当列表显示时，可不查询出该字段，而在单个数据查看时，可查询出该字段，以方便展示给用户。

* 子表数据量较少时，可使用一对多关联

  在业务数据的定义中，会存在大量的父子关系的表，如销售单和销售单明细等，在定义的过程中，可定义成一对多的关联关系。

  而对于业务分组与分组内的数据，则不宜定义成一对多的关系，而直接定义成简单类型数据，因为这类情况下，子表中数据量较大，定义成一对多的关系也不能解决数据批量查询的问题，以及在父表加载时，大量载入子表内容，容易造成内存的浪费。


### 典型Domain类

```java
@Entity
@DynamicInsert
@DynamicUpdate
@Table(name = "MY_USER")
@Auditable
@TypeDefs({ @TypeDef(name = "binary", typeClass = RelationDbBinaryType.class),
        @TypeDef(name = "text", typeClass = RelationDbTextType.class) })
public class User extends AbstractAuditable<Long> {
    private static final long serialVersionUID = 1L;
    @javax.validation.constraints.Size(min = 1, max = 10)
    @Auditable
    @Column(name = "FIRST_NAME")
    private String firstName;
    @javax.validation.constraints.Size(max = 10)
    @Auditable
    @Column(name = "LAST_NAME")
    private String lastName;
    @Auditable
    @Column(name = "EMAIL")
    private String email;
    @Column(name = "PHOTO", columnDefinition = "LONGVARBINARY")
    @Type(type = "binary", parameters = {
            @Parameter(name = "columnName", value = "PHOTO"),
            @Parameter(name = "jdbcTemplate", value = MaculaConstants.JDBC_TEMPLATE_NAME),
            @Parameter(name = "tableName", value = "MY_USER") })
    @Audited
    private Binary photo;
    @Column(name = "PROFILE", columnDefinition = "CLOB")
    @Type(type = "text", parameters = {
            @Parameter(name = "columnName", value = "PROFILE"),
            @Parameter(name = "jdbcTemplate", value = MaculaConstants.JDBC_TEMPLATE_NAME),
            @Parameter(name = "tableName", value = "MY_USER") })
    @Audited
    private Text profile;

    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "homeTel", column = @Column(name = "HOME_TEL")),
            @AttributeOverride(name = "officeTel", column = @Column(name = "OFFICE_TEL")) })
    private EmbbedContactInfo contactInfo;

    ... 省略get set
}
```



