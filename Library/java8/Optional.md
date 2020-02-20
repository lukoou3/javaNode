## Optional
**Optional<T>类（java.util.Optional）是一个容器类**，代表一个值存在或不存在，原来用null表示一个值不存在，现在Optional可以更好的表达这个概念。并且可以避免空指针异常。**Optional是不可变对象，类似于scala中的Option**。

```java
public final class Optional<T> {
    private static final Optional<?> EMPTY = new Optional<>();
    private final T value;

    private Optional() {
        this.value = null;
    }

    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
}
```

### Optional的创建
```java
//空Optional的创建
public static<T> Optional<T> empty();
//不能传入null，调用了Objects.requireNonNull(value)
public static <T> Optional<T> of(T value);
//可以传入null，value == null ? empty() : of(value)
public static <T> Optional<T> ofNullable(T value)
```

### Optional类的方法
Optional类的方法很好理解，基本上看到函数的命名和函数的签名就能明白，基本和scala类似。

#### boolean isPresent()
容器内是否有值

#### T get()
如果Optional有值则将其返回，否则抛出NoSuchElementException。

#### Unit ifPresent(consumer: T => Unit)
如果Optional实例有值则为其调用consumer，否则不做处理 。隐士地其中进行了null判断。
相当于scala中的foreach

#### T orElse(other: T)
如果有值则将其返回，否则返回指定的其它值。

#### T orElseGet(other: () => T)
orElseGet与orElse方法类似，区别在于得到的默认值。orElse方法将传入的一个值作为默认值，orElseGet方法可以接受Supplier接口的实现用来生成默认值

#### [X < Throwable] T orElseThrow(exceptionSupplier: () => X)
如果有值则将其返回，否则抛出supplier接口创建的异常。

#### Optional[T] filter(predicate: T => boolean)
如果有值并且满足断言条件返回包含该值的Optional，否则返回空Optional。

#### [U] Optional[U] map(mapper: T => U)
map处理有值的情况，如果有值，则对其执行调用map参数中的函数得到返回值，否则返回空Optional。

#### [U] Optional[U] flatMap(mapper: T => Optional[U])
如果有值，为其执行mapping函数返回Optional类型返回值，否则返回空Optional。flatMap与map（Funtion）方法类似，区别在于flatMap中的mapper返回值必须是Optional。调用结束时，flatMap不会对结果用Optional封装。


