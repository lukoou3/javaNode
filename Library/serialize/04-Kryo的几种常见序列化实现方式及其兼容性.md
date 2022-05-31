# Kryo的几种常见序列化实现方式及其兼容性

摘抄自：https://blog.csdn.net/hchaoh/article/details/106626924

## Kryo 是啥


[Kryo](https://github.com/EsotericSoftware/kryo "Kryo")是Java生态中的一种[序列化](https://so.csdn.net/so/search?q=序列化&spm=1001.2101.3001.7020 "序列化")框架。许多软件组织在使用Dubbo（Dubbox）这套RPC框架时，经常会搭配使用 Kryo 作为其序列化方案。Kryo本身自带了很多针对 Java 原始数据类型 和 JDK常见类型的 序列化实现（如，[DefaultSerializers](https://github.com/EsotericSoftware/kryo/blob/master/src/com/esotericsoftware/kryo/serializers/DefaultSerializers.java "DefaultSerializers")）。它的姐妹工程[Kryo-Serializers](https://github.com/magro/kryo-serializers "Kryo-Serializers")还提供很多额外的序列化实现。


## 序列化/[反序列化](https://so.csdn.net/so/search?q=反序列化&spm=1001.2101.3001.7020 "反序列化")的兼容性问题


序列化/反序列化 最主要的是关注对象内容。大多数场景中，我们只会序列化对象的内容，即各字段的值。而对象的类信息，它们属于元信息，是不会被序列化的。如果将这些元信息也加入到序列化内容中，会导致得到的结果数据非常大。尤其在[RPC](https://so.csdn.net/so/search?q=RPC&spm=1001.2101.3001.7020 "RPC")之类的网络传输中，这些数据会消耗大量资源。


那么，如果不传递这些元信息，数据的接收端怎么知道这些数据的类型呢？这就需要事先在接收端也准备好一份对应的类型信息（通常是一个JAR包）。（某些场景中将这类事先分发的元信息称为“字典”。有点类似战争中分发的密码本。）


这又引出了另一个问题：发送端 和 接收端 的类型信息（JAR包）版本可能不一致；这种不一致很可能导致接收端反序列化失败。尤其是在大型的复杂软件体系中，发送端 和 接收端 的升级节奏可能不一样的，很多时候需要容忍有较长时间的不同版本同时运行。（战争中密码本会定期更新；遇到密码本泄漏或密码被敌军破译的情况时，还需紧急更新。）


此文中讨论的 “兼容性” 仅表示数据本身的序列化和反序列化。业务层面的兼容性当然得由业务系统的设计者自行考量。


为了达成序列化/反序列化的的“兼容性”，**最关键的是要“跳过”那些无法被识别的字段**。如何才能实现这个“跳过”呢？显然，需要鉴别出每个字段值数据对应了哪个字段。让我们来看看 Kryo 提供的序列化实现，有什么样的兼容性能力，以及它们是如何来实现这个“鉴别”能力的。


## Kryo 常见序列化实现方式


### FieldSerializer


FieldSerializer 是 Kryo 默认的序列化实现。顾名思义，它是以直接读写字段的方式，来实现序列化和反序列化的。它会序列化所有 “非transient”字段。对于 POJO 类及大多数其它Java类，都无需额外配置。



“直接读写字段”意味着，对于 “非public”的字段，FieldSerializer 会利用 Java 的反射机制来访问字段；对于“public”字段，则是利用 ReflectASM 进行访问。所以将字段设置为 “public” 可能会获得更好的序列化性能。但是大多数项目通常不会为了这点性能，去改变约定俗成的常规POJO写法。即，字段还是用 “private” 修饰，并配上对应的 getter 和 setter 方法。


FieldSerializer 的兼容性可能无法满足某些软件系统的变更需要。如，增加或删除字段，及改变字段的类型，都会使得新系统无法兼容老版本数据。（如果重命名字段，那么各字段的字典顺序必须保持不变，才能确保兼容。）


### VersionFieldSerializer


VersionFieldSerializer 是 FieldSerializer 的一个扩展。它提供了**部分**“**向后兼容**”的能力。



向后兼容：新的系统可以兼容老版本的数据。


如果只是新增字段，那么 VersionFieldSerializer 可以向后兼容。但如果是删除字段、重命名字段 或 更改字段类型，那么也会导致兼容性问题。


它通过字段上附带的 @Since 注解来区别对待新字段。对于一个特定的字段，一旦被添加 @Since 注解，那么其注解值，也就是字段的“版本号”是不能被改变的。


它比 FieldSerializer 的性能开销多了一点点。


### TaggedFieldSerializer


TaggedFieldSerializer 也是 FieldSerializer 的一个扩展。它提供了**部分**“**向后兼容**”和“**向前兼容**”的能力。



向前兼容：老的系统可以兼容新版本的数据。


如果只是新增字段、重命名字段或删除字段，那么新系统还是能兼容老版本数据。但是更改字段类型，还是会导致兼容性问题的。


它通过字段上附带的 @Tag 注解来识别字段。@Tag 注解的作用类似于给每个字段起一个唯一的名称。所以父类与子类的Tag值也必须不同。本质上来说，它的“向前兼容”、“向后兼容”及 序列化的性能，取决于它对“未识别Tag数据”和数据分块编码的处理方式。


如果允许读取“未识别Tag的数据”（readUnknownTagData=true），那么它在序列化每个字段时，都会先将其类型信息也写入序列化数据中，以便后续反序列化时能正确处理。但是如果字段对应类被删除，那么还是会导致反序列化失败，除非启用针对每个字段做独立的 “分块编码”（chunkedEncoding=true），来跳过这些“无法识别”的字段。


如果不允许读取“未识别Tag的数据”（readUnknownTagData=false），那么新版本类中就不能删除被业务废弃的字段，否则必须启用针对每个字段做独立的 “分块编码”（chunkedEncoding=true），来跳过这些“无法识别”的字段。


上述针对每个字段做独立的“分块编码”，会使降低性能。


但是 TaggedFieldSerializer 支持用 @Deprecated 注解来标识被业务废弃的字段，从而在序列化和反序列化时忽略这些字段，以提高性能。


TaggedFieldSerializer 的“向前兼容”是通过“跳过无法识别的Tag”（skipUnknownTags=true）来设置的。


综上，与 VersionFieldSerializer 相比，TaggedFieldSerializer 有两个优势：


* 支持字段重命名；    
* 支持利用 @Deprecated 注解，忽略废弃字段，以提高性能。


#### “分块编码（Chunked Encoding）”是啥？


为了方便反序列化时知道数据的长度，以便控制读操作，需要把数据的长度也告诉反序列的那一方。所以在序列化时，我们需要先写序列化后数据的长度（字节数），再写序列化后的数据。这是典型的 “TLV（Type-Length-Value）”模式思想。


但是我们只有在完成对字段数据的序列化后，才能知道这些数据有多大。而一开始就定义一个足够大的缓冲区，来存放序列化后的内容，完成所有字段序列化后，再写数据长度和数据内容，显然是不可取的。因为我们不知道数据会有多大，这个缓存区可能会大得非常不合理。而且这么做也失去了“流式处理”的特性。“分块编码”就是用来解决这个问题的。


“分块编码”使用了一个较小的缓存区。缓存区被塞满时，就写一次缓存区数据长度，再写缓存区数据。这么一次操作的数据被称为“一块数据”。然后缓冲区会被清空，并继续执行后续操作，直到处理完所有数据。最后再写个数字“0”，表示已完成所有数据块的处理。


### CompatibleFieldSerializer


CompatibleFieldSerializer 也是 FieldSerializer 的一个扩展。它也提供了**部分**“**向后兼容**”和“**向前兼容**”的能力。


如果只是新增或删除字段，那么新系统还是能兼容老版本数据。但是更改字段类型或重命名字段，还是会导致兼容性问题。


CompatibleFieldSerializer 的用法很简单，对于绝大多数类型都无需额外注解。只需在被序列化的类上增加注解 @DefaultSerializer(CompatibleFieldSerializer.class)。这便利性和 FieldSerializer 很像。


CompatibleFieldSerializer 是根据每个字段的名称来识别一段数据对应哪个字段。这点与 TaggedFieldSerializer 非常类似。显然，我们也要避免父类和子类出现同名字段的情况请。


当一个被序列化的类第一次出现时，它会记住类的所有字段名。当执行序列化和反序列化时，它都会针对每个字段使用“分块编码”特性。这样就能实现“跳过”未知字段的特性。


与 TaggedFieldSerializer 类似，它的“向前兼容”、“向后兼容”及 序列化的性能，取决于它对“未识别字段”和数据分块编码的处理方式。


但是删除被“引用”的字段，也还是会引起兼容性问题的。


综合来说，因为 CompatibleFieldSerializer 提供的兼容性，以及几乎无额外配置的特性，很多软件系统都采用了这种模式。


（《[Dubbo 服务序列化兼容性技巧 —— CompatibleFieldSerializer](https://blog.csdn.net/hchaoh/article/details/103906673 "Dubbo 服务序列化兼容性技巧 —— CompatibleFieldSerializer")》）



千万不要忽视一项技术部署实施过程的复杂性。非常多的时候，就是这些看起来很简单的操作，引入了bug。


一个经验丰富的软件架构师或技术负责人会尽量避免人为操作。人为操作越多，风险越大。


“没有bug的程序，是没有代码的程序”。


### BeanSerializer


BeanSerializer 与 FieldSerializer 非常类似。主要区别是，BeanSerializer 用 getter 和 setter 方法来访问字段，而不是直接访问。所以 BeanSerializer 的性能会稍稍低一点，但是会更安全些。另，BeanSerializer 也**不支持**“向前兼容”和“向后兼容”。

