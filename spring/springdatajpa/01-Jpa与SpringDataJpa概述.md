## Jpa与SpringDataJpa概述
jpa是sun公司提出的规范。Hibernate 在 3.2 以后根据 JPA 规范提供了一套操作持久层的 API。SpringDataJpa对jpa做了一层封装，具体的实现可以选择不同的厂商，一般默认使用Hibernate来实现。

回顾jdbc操作以及引入orm
![01-回顾jdbc操作以及引入orm](/assets/01-回顾jdbc操作以及引入orm.png)

jpa:
![02-jpa](/assets/02-jpa.png)

### 什么是JPA

#### 什么是ORM
ORM（Object-Relational Mapping） 表示对象关系映射。在面向对象的软件开发中，通过ORM，就可以把对象映射到关系型数据库中。只要有一套程序能够做到建立对象与数据库的关联，操作对象就可以直接操作数据库数据，就可以说这套程序实现了ORM对象关系映射。简单的说：ORM就是建立实体类和数据库表之间的关系，从而达到操作实体类就相当于操作数据库表的目的。ORM是一种解决问题的思想。

#### JPA概念
JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

JPA由EJB 3.0软件专家组开发，作为JSR-220实现的一部分。但它又不限于EJB 3.0，你可以在Web应用、甚至桌面应用中使用。JPA的宗旨是为POJO提供持久化标准规范。总的来说，JPA包括以下3方面的技术： 
 
* ORM映射元数据，JPA支持XML和JDK 5.0注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中；    
* JPA 的API，用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，开发者从繁琐的JDBC和SQL代码中解脱出来。    
* 查询语言，这是持久化操作中很重要的一个方面，通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。  

#### jpa的优势
	标准化
   JPA 是 JCP 组织发布的 Java EE 标准之一，因此任何声称符合 JPA 标准的框架都遵循同样的架构，提供相同的访问API，这保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行。

	容器级特性的支持
   JPA框架中支持大数据集、事务、并发等容器级事务，这使得 JPA 超越了简单持久化框架的局限，在企业应用发挥更大的作用。

	简单方便
   JPA的主要目标之一就是提供更加简单的编程模型：在JPA框架下创建实体和创建Java 类一样简单，没有任何的约束和限制，只需要使用 javax.persistence.Entity进行注释，JPA的框架和接口也都非常简单，没有太多特别的规则和设计模式的要求，开发者可以很容易的掌握。JPA基于非侵入式原则设计，因此可以很容易的和其它框架或者容器集成

	查询能力
   JPA的查询语言是面向对象而非面向数据库的，它以面向对象的自然语法构造查询语句，可以看成是Hibernate HQL的等价物。JPA定义了独特的JPQL（Java Persistence Query Language），JPQL是EJB QL的一种扩展，它是针对实体的一种查询语言，操作对象是实体，而不是关系数据库的表，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能够提供的高级查询特性，甚至还能够支持子查询。

	高级特性
   JPA 中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系，这样的支持能够让开发者最大限度的使用面向对象的模型设计企业应用，而不需要自行处理这些特性在关系数据库的持久化。

#### Jpa的供应商
JPA 的目标之一是制定一个可以由很多供应商实现的API，并且开发人员可以编码来实现该API，而不是使用私有供应商特有的API。因此开发人员只需使用供应商特有的API来获得JPA规范没有解决但应用程序中需要的功能。尽可能地使用JPA API，但是当需要供应商公开但是规范中没有提供的功能时，则使用供应商特有的API。

**Hibernate**

JPA是需要Provider来实现其功能的，Hibernate就是JPA Provider中很强的一个，应该说无人能出其右。从功能上来说，JPA就是Hibernate功能的一个子集。**Hibernate 从3.2开始，就开始兼容JPA**。Hibernate3.2获得了Sun TCK的JPA(Java Persistence API) 兼容认证。
只要熟悉Hibernate或者其他ORM框架，在使用JPA时会发现其实非常容易上手。

**OpenJPA**

OpenJPA 是 Apache 组织提供的开源项目，它实现了 EJB 3.0 中的 JPA 标准，为开发者提供功能强大、使用简单的持久化数据管理框架。OpenJPA 封装了和关系型数据库交互的操作，让开发者把注意力集中在编写业务逻辑上。OpenJPA 可以作为独立的持久层框架发挥作用，也可以轻松的与其它 Java EE 应用框架或者符合 EJB 3.0 标准的容器集成。

**TopLink**

TopLink，是位居第一的Java对象关系可持续性体系结构，原署WebGain公司的产品，后被Oracle收购，并重新包装为Oracle AS TopLink。TOPLink为在关系数据库表中存储 Java 对象和企业 Java 组件 (EJB) 提供高度灵活和高效的机制。TopLink 为开发人员提供极佳的性能和选择，可以与任何数据库、任何应用服务器、任何开发工具集和过程以及任何 J2EE 体系结构协同工作。

### 什么是SpringDataJpa
Spring Data JPA 框架，主要针对的就是 Spring 唯一没有简化到的业务逻辑代码，至此，开发者连仅剩的实现持久层业务逻辑的工作都省了，唯一要做的，就只是声明持久层的接口，其他都交给 Spring Data JPA 来帮你完成！

Spring data jpa是在JPA规范下提供了Repository层的实现，但是使用哪一种ORM需要你来决定（默认使用Hibernate JPA的实现）。虽然ORM框架都实现了JPA规范，但是在不同的ORM框架之间切换仍然需要编写不同的代码，而通过使用Spring data jpa能够方便大家在不同的ORM框架之间进行切换而不要更改代码。并且spring data jpa 对Repository层封装的很好，也省去了不少的麻烦。

spring data jpa、jpa以及ORM框架之间的关系:
![](assets/markdown-img-paste-2020013018423316.png)

