## stream算子
java8的stream的操作类似于spark的rdd的算子，都是延时计算的，计算都是使用的pipline模式(这一点和scala的集合操作不同)。不过stream是一次性的，只能执行一次action算子，再次执行算子会抛出异常。

stream计算分串行和并行两种，创建十分简单只需调用stream和parallelStream。需要注意的是并不是使用并行流就一定比串行流效率高：数据集类型可分解性的影响(ArrayList、IntStream.range极佳，HashSet、TreeSet好，LinkedList、Stream.iterate差)；小的数据集不适用于并行流；有些算子在并行流上的性能就比顺序流差，特别是limit和findFirst等依赖于元
素顺序的操作，它们在并行流上执行的代价非常大；有些算子本身就不支持并行操作；

我看很多地方把Stream的操作符分为两种：中间操作符和终止操作符。不过我还是习惯类比spark把把Stream的操作分为转换算子和action算子。

## 转换算子
### filter(T => boolean)
```java
filter(predicate: T => boolean): Stream[T]

Stream<T> filter(Predicate<? super T> predicate);
```

### map(T => R)
```java
map(mapper: T => R): Stream[R]

<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

### flatMap(T => Stream[R])
```java
flatMap(mapper: T => Stream[R]): Stream[R]

<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

### sorted() / sorted((T, T) => int)
```java
sorted(): Stream[T] 
sorted(comparator: (T, T) => int): Stream[T] 

Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```

### skip、limit
skip(2).limit(3)跳过2个最多返回3个

limit(3).skip(2)最多返回3个跳过limit后的集合中的2个
```java
Stream<T> skip(long n);
Stream<T> limit(long maxSize);
```

### distinct()
distinct()  是通过 equals 方法来判断是否相等的
```java
Stream<T> distinct();
```
### peek(T => Unit)
peek和forEach的参数函数一样不过看方法的返回值就知道这是一个转换算子。peek的英文解释为瞥一眼、偷看。
```java
peek(action: T => Unit): Stream[T] ;

Stream<T> peek(Consumer<? super T> action);
```

看一下这个函数的官方注释，说是主要用于debug的：
This method exists mainly to support debugging, where you want to see the elements as they flow past a certain point in a pipeline:
```java
Stream.of("one", "two", "three", "four")
  .filter(e -> e.length() > 3)
  .peek(e -> System.out.println("Filtered value: " + e))
  .map(String::toUpperCase)
  .peek(e -> System.out.println("Mapped value: " + e))
  .collect(Collectors.toList());
```

## Action算子
### forEach(T => Unit)
```java
forEach(T => Unit): Unit

void forEach(Consumer<? super T> action);
```

### reduce
```java
T reduce(T, (T, T) => T)
Optional<T> reduce((T, T) => T)
U reduce(U, (U, T) => U, (U, U) => U)

reduce(identity: T, accumulator: (T, T) => T): T 
reduce(accumulator: (T, T) => T): Optional[T]
reduce(identity: U, accumulator: (U, T) => U, combiner: (U, U) => U): U 

T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
<U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
```

### allMatch/anyMatch/noneMatch(T -> boolean)
```java
allMatch(T => boolean):boolean
anyMatch(T => boolean):boolean
noneMatch(T => boolean):boolean

boolean allMatch(Predicate<? super T> predicate);
boolean anyMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);
```

### collect(collector)
collect和recuce的区别是collect的初始值是容器，累加合并函数的无返回值
```java

<R, A> R collect(Collector<? super T, A, R> collector);
<R> R collect(Supplier<R> supplier,
              BiConsumer<R, ? super T> accumulator,
              BiConsumer<R, R> combiner);
```

**Collector接口有静态的of方法用于生成Collector**

Collector接口：
```java
/**
 * 比较reduce函数(和spark的reduce差不多)：
 * identity: U 初始值
 * accumulator: (U, T) => U 累加器(注意有返回值)
 * combiner: (U, U) => U merge合并(并行才会调用)
 */


/**
 * supplier: () => A 返回容器
 * accumulator：(T, A) => Unit 累加器(注意无返回值)
 * combiner：(A, A) => A 合并器，merge合并(并行才会调用)
 * finisher: A => R 完成器，最后对容器进行转化(不是必须)
 * characteristics: Set<Characteristics> 这个Collector的特性，这个set应该是不可变的。
 * Characteristics的三个值：CONCURRENT(可以并行执行，必须同时标为UNORDERED才能并行)，UNORDERED(不收顺序影响)，IDENTITY_FINISH(跳过完成器)
 *
 */
```

### min/max((T, T) => int)
```java
min((T, T) => int): Optional[T]
max((T, T) => int): Optional[T]

Optional<T> min(Comparator<? super T> comparator);
Optional<T> max(Comparator<? super T> comparator);
```

### count()
```java
long count();
```

### findAny()/findFirst()
findAny() 使用 stream() 时找到的是第一个元素；使用 parallelStream() 并行时找到的是其中一个元素
```java
findFirst(): Optional[T]
findAny(): Optional[T]

Optional<T> findFirst();
Optional<T> findAny();
```

## stream测试
### entity
People:
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class People {
    private String name;
    private Sex sex;
    private int age;
}

public enum Sex {
    MALE("男"), FEMALE("女");
    private final String sex;
    Sex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return this.sex;
    }
}
```

CollectorImpl(其实这个类不需要，Collector接口有工厂方法):
```java
import java.util.Set;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

/**
 * 比较reduce函数(和spark的reduce差不多)：
 * identity: U 初始值
 * accumulator: (U, T) => U 累加器(注意有返回值)
 * combiner: (U, U) => U merge合并(并行才会调用)
 */


/**
 * supplier: () => A 返回容器
 * accumulator：(T, A) => Unit 累加器(注意无返回值)
 * combiner：(A, A) => A 合并器，merge合并(并行才会调用)
 * finisher: A => R 完成器，最后对容器进行转化(不是必须)
 * characteristics: Set<Characteristics> 这个Collector的特性，这个set应该是不可变的。
 * Characteristics的三个值：CONCURRENT(可以并行执行，必须同时标为UNORDERED才能并行)，UNORDERED(不收顺序影响)，IDENTITY_FINISH(跳过完成器)
 *
 */
public class CollectorImpl<T, A, R> implements Collector<T, A, R> {
    private final Supplier<A> supplier;
    private final BiConsumer<A, T> accumulator;
    private final BinaryOperator<A> combiner;
    private final Function<A, R> finisher;
    private final Set<Characteristics> characteristics;

    CollectorImpl(Supplier<A> supplier,
                  BiConsumer<A, T> accumulator,
                  BinaryOperator<A> combiner,
                  Function<A,R> finisher,
                  Set<Characteristics> characteristics) {
        this.supplier = supplier;
        this.accumulator = accumulator;
        this.combiner = combiner;
        this.finisher = finisher;
        this.characteristics = characteristics;
    }

    CollectorImpl(Supplier<A> supplier,
                  BiConsumer<A, T> accumulator,
                  BinaryOperator<A> combiner,
                  Set<Characteristics> characteristics) {
        this(supplier, accumulator, combiner, castingIdentity(), characteristics);
    }

    private static <I, R> Function<I, R> castingIdentity() {
        return i -> (R) i;
    }

    @Override
    public BiConsumer<A, T> accumulator() {
        return accumulator;
    }

    @Override
    public Supplier<A> supplier() {
        return supplier;
    }

    @Override
    public BinaryOperator<A> combiner() {
        return combiner;
    }

    @Override
    public Function<A, R> finisher() {
        return finisher;
    }

    @Override
    public Set<Characteristics> characteristics() {
        return characteristics;
    }
}
```

### 测试代码
#### StreamTransformationTest.java
```java
public class StreamTransformationTest {
    private List<People> list = new ArrayList<>();

    @Before
    public void init() {
        list.add(new People("莫南", Sex.MALE, 18));
        list.add(new People("燕青丝", Sex.FEMALE, 18));
        list.add(new People("苏流沙", Sex.FEMALE, 20));
        list.add(new People("沐璇音", Sex.FEMALE, 18));
        list.add(new People("莫扶苏", Sex.MALE, 300));
        list.add(new People("洛夕也", Sex.FEMALE, 19));
        list.add(new People("倾天达", Sex.FEMALE, 19));
        list.add(new People("轻轻寒", Sex.FEMALE, 320));
    }

    @Test
    public void testFilter() {
        //Stream<T> filter(T => boolean)
        List<People> peoples = list.stream().filter(p -> p.getSex() == Sex.MALE).collect(Collectors.toList());
        System.out.println(peoples);
    }

    @Test
    public void testMap() {
        //Stream<R> map(T => R)
        List<String> names = list.stream().map(p -> p.getName()).collect(Collectors.toList());
        System.out.println(names);

        String namesStr = list.stream().map(People::getName).collect(Collectors.joining("\", \"","\"","\""));
        System.out.println(namesStr);
    }

    @Test
    public void testFlatMap() {
        //Stream<R> flatMap(T => Stream<R>)
        String names = list.stream().flatMap(p -> Arrays.asList(p.getName(), p.getName() + "1").stream()).collect(Collectors.joining(", "));
        System.out.println(names);
    }

    @Test
    public void testSorted() {
        List<Integer> ages = list.stream().map(People::getAge).sorted().collect(Collectors.toList());
        System.out.println(ages);

        List<People> peopleList = list.stream().sorted((a, b) -> a.getAge() - b.getAge()).collect(Collectors.toList());
        System.out.println(peopleList);

        peopleList = list.stream().sorted(Comparator.comparingInt(People::getAge)).collect(Collectors.toList());
        System.out.println(peopleList);

        peopleList = list.stream().sorted(Comparator.comparingInt(People::getAge).reversed()).collect(Collectors.toList());
        System.out.println(peopleList);

        //Collator类是用来执行区分语言环境的String比较的，这里是选择中文环境
        Comparator<Object> china_compare = Collator.getInstance(java.util.Locale.CHINA);
        peopleList = list.stream().sorted(Comparator.comparingInt(People::getAge).thenComparing(Comparator.comparing(People::getName, china_compare))).collect(Collectors.toList());
        System.out.println(peopleList);
    }

    @Test
    public void testComparator(){
        //获取中文环境  沐字在最后？
        Comparator<Object> com = Collator.getInstance(java.util.Locale.CHINA);
        String[] newArray={"中山","汕头","广州","安庆","阳江","南京","武汉","北京","安阳","北方","洛阳","沐浴"};
        List<String> list = Arrays.asList(newArray);
        Collections.sort(list, com);
        for(String i:list){
            System.out.print(i+"  ");
        }

        System.out.println();

        newArray= new String[]{"莫南", "燕青丝", "苏流沙", "沐璇音", "莫扶苏", "洛夕也", "倾天达", "轻轻寒"};
        list = Arrays.asList(newArray);
        Collections.sort(list, com);
        for(String i:list){
            System.out.print(i+"  ");
        }
    }

    @Test
    public void testDistinct(){
        //Stream<T> distinct()  是通过 equals 方法来判断是否相等的
        list.add(new People("莫南", Sex.MALE, 18));
        List<People> peoples = list.stream().distinct().collect(Collectors.toList());
        System.out.println(peoples);
    }

    @Test
    public void testSkipAndLimit() {
        //Stream<T> skip(long n)
        //Stream<T> limit(long maxSize)
        //skip(2).limit(3)跳过2个最多返回3个
        List<People> peoples = list.stream().skip(2).limit(3).collect(Collectors.toList());
        System.out.println(peoples);

        //limit(3).skip(2)最多返回3个跳过limit后的集合中的2个
        peoples = list.stream().limit(3).skip(2).collect(Collectors.toList());
        System.out.println(peoples);
    }

    @Test
    public void testPeek() {
        //skip(2).limit(3)跳过2个最多返回3个
        List<People> peoples = list.stream().peek(p -> System.out.println(p)).collect(Collectors.toList());
        System.out.println(peoples);
    }
}
```

#### StreamActionTest.java
```java
public class StreamActionTest {
    private List<People> list = new ArrayList<>();

    @Before
    public void init() {
        list.add(new People("莫南", Sex.MALE, 18));
        list.add(new People("燕青丝", Sex.FEMALE, 18));
        list.add(new People("苏流沙", Sex.FEMALE, 20));
        list.add(new People("沐璇音", Sex.FEMALE, 18));
        list.add(new People("莫扶苏", Sex.MALE, 300));
        list.add(new People("洛夕也", Sex.FEMALE, 19));
        list.add(new People("倾天达", Sex.FEMALE, 19));
        list.add(new People("轻轻寒", Sex.FEMALE, 320));
    }

    @Test
    public void testForEach() {
        //void forEach(T => Unit)
        list.stream().filter(p -> p.getSex() == Sex.MALE).forEach(p -> System.out.println(p));
        list.stream().filter(p -> p.getSex() == Sex.MALE).forEach(System.out::println);
    }

    @Test
    public void testCollect() {
        //<R, A> R collect(Collector<? super T, A, R> collector)
        List<String> names = list.stream().map(p -> p.getName()).collect(Collectors.toList());
        System.out.println(names);

        String namesStr = list.stream().map(People::getName).collect(Collectors.joining(", "));
        System.out.println(namesStr);
    }

    @Test
    public void testReduce() {
        //T reduce(T, (T, T) => T)
        Integer sum = list.stream().map(People::getAge).reduce(0, (a, b) -> a + b);
        System.out.println(sum);

        System.out.println(list.stream().map(People::getAge).reduce(0, Integer::sum));

        //Optional<T> reduce((T, T) => T)   集合为空返回empty，不会抛出异常
        Optional<Integer> sumOptional = list.stream().map(People::getAge).skip(30).reduce((a, b) -> a + b);
        System.out.println(sumOptional);

        //集合为一个元素，直接返回元素
        sumOptional = list.stream().map(People::getAge).limit(1).reduce((a, b) -> a + b);
        System.out.println(sumOptional);

        //U reduce(U, (U, T) => U, (U, U) => U)
        System.out.println(list.stream().reduce(0, (x, p) -> x + p.getAge(), (x, y) -> x + y));

        //看一下执行流程:非并行第二个合并的函数似乎没用
        list.stream().reduce(0, (x, p) -> {
            System.out.println(x +" ," + p.getName() + " => "+ (x + p.getAge()));
            return x + p.getAge();
        }, (x, y) -> {
            System.out.println(x +" ," + y + " => "+ (x + y));
            return x + y;
        });

        System.out.println("-----------------");

        //并行时第二个合并的函数有用，线程的执行顺序有点看不懂
        list.parallelStream().reduce(0, (x, p) -> {
            System.out.println(Thread.currentThread().getName()+ ": "+ x +" ," + p.getName() + " => "+ (x + p.getAge()));
            return x + p.getAge();
        }, (x, y) -> {
            System.out.println(Thread.currentThread().getName()+ ": "+ x +" ," + y + " => "+ (x + y));
            return x + y;
        });
    }

    @Test
    public void testCountMaxMin() {
        //long count()
        long count = list.stream().filter(p -> p.getAge() < 20).count();
        System.out.println(count);

        //Optional<T> max((T, T) => int)
        Optional<People> peopleOptional = list.stream().max(Comparator.comparingInt(People::getAge));
        System.out.println(peopleOptional.get());

        //Optional<T> min((T, T) => int)
        peopleOptional = list.stream().min(Comparator.comparingInt(People::getAge));
        System.out.println(peopleOptional.get());
    }

    @Test
    public void testAllMatch() {
        //boolean allMatch(T => boolean)
        boolean allMatch = list.stream().allMatch(p -> p.getAge() > 20);
        System.out.println(allMatch);

        allMatch = list.stream().allMatch(p -> p.getAge() > 15);
        System.out.println(allMatch);
    }

    @Test
    public void testAnyMatch() {
        //boolean anyMatch(T => boolean)
        boolean anyMatch = list.stream().anyMatch(p -> p.getAge() > 20);
        System.out.println(anyMatch);
    }

    @Test
    public void testNoneMatch() {
        //boolean noneMatch(T => boolean)
        boolean noneMatch = list.stream().noneMatch(p -> p.getAge() < 20);
        System.out.println(noneMatch);

        noneMatch = list.stream().noneMatch(p -> p.getAge() < 15);
        System.out.println(noneMatch);
    }

    @Test
    public void testFindFirst() {
        //Optional<T> findFirst()
        Optional<People> peopleOptional = list.stream().findFirst();
        System.out.println(peopleOptional.get());
    }

    @Test
    public void testFindAny() {
        //Optional<T> findAny() 使用 stream() 时找到的是第一个元素；使用 parallelStream() 并行时找到的是其中一个元素
        Optional<People> peopleOptional = list.stream().findAny();
        System.out.println(peopleOptional.get());
    }

}
```

#### StreamCollectTest.java
```java
public class StreamCollectTest {
    private List<People> list = new ArrayList<>();

    @Before
    public void init() {
        list.add(new People("莫南", Sex.MALE, 18));
        list.add(new People("燕青丝", Sex.FEMALE, 18));
        list.add(new People("苏流沙", Sex.FEMALE, 20));
        list.add(new People("沐璇音", Sex.FEMALE, 18));
        list.add(new People("莫扶苏", Sex.MALE, 300));
        list.add(new People("洛夕也", Sex.FEMALE, 19));
        list.add(new People("倾天达", Sex.FEMALE, 19));
        list.add(new People("轻轻寒", Sex.FEMALE, 320));
    }

    @Test
    public void testToCollection() {
        List<String> names = list.stream().map(p -> p.getName()).collect(Collectors.toList());
        System.out.println(names);

        Set<String> nameSet = list.stream().map(p -> p.getName()).collect(Collectors.toSet());
        System.out.println(nameSet);

        LinkedList<String> nameList = list.stream().map(p -> p.getName()).collect(Collectors.toCollection(LinkedList::new));
        System.out.println(nameList);
    }

    @Test
    public void testToMap() {
        Map<String, Integer> map = list.stream().collect(Collectors.toMap(x -> x.getName(), x -> x.getAge()));
        System.out.println(map);

        list.add(new People("莫南", Sex.MALE, 28));
        //关于合并函数 BinaryOperator<U> mergeFunction对象：当toMap中没有用合并函数时，出现key重复时，会抛出异常
        map = list.stream().collect(Collectors.toMap(x -> x.getName(), x -> x.getAge(), (v1, v2) -> v2));
        System.out.println(map);
    }


    @Test
    public void testJoining() {
        //Collectors.joining这能对Stream[CharSequence]操作
        String names = list.stream().map(p -> p.getName()).collect(Collectors.joining(" :: "));
        System.out.println(names);

        //用mapping对joining包装一下
        names = list.stream().collect(Collectors.mapping(People::getName, Collectors.joining(" :: ")));
        System.out.println(names);
    }

    @Test
    public void testGroupingBy() {
        //groupingBy(T => K)
        Map<Sex, List<People>> sexListMap = list.stream().collect(Collectors.groupingBy(People::getSex));
        System.out.println(sexListMap);

        //groupingBy(T => K, Collector[T, A, D])
        //Collector对分组的集合进行操作
        Map<Sex, Set<People>> setMap = list.stream().collect(Collectors.groupingBy(People::getSex, Collectors.toSet()));
        System.out.println(setMap);

        //groupingBy(T => K, () -> Map[K,D],Collector[T, A, D])
        LinkedHashMap<Sex, Set<People>> setLinkedHashMap = list.stream().collect(Collectors.groupingBy(People::getSex, () -> new LinkedHashMap<>(), Collectors.toSet()));
        System.out.println(setLinkedHashMap);

        LinkedHashMap<Sex, String> linkedHashMap = list.stream().collect(Collectors.groupingBy(People::getSex, () -> new LinkedHashMap<>(),
                Collectors.mapping(People::getName, Collectors.joining(", "))));
        System.out.println(linkedHashMap);
    }

    @Test
    public void testPartitioningBy() {
        //partitioningBy(T => boolean)
        Map<Boolean, List<People>> map = list.stream().collect(Collectors.partitioningBy(p -> p.getSex() == Sex.MALE));
        System.out.println(map);

        //partitioningBy(T => boolean, Collector[T, A, D])
        //Collector对分组的集合进行操作
        Map<Boolean, String> stringMap = list.stream().collect(Collectors.partitioningBy(p -> p.getSex() == Sex.MALE,
                Collectors.mapping(People::getName, Collectors.joining(", "))));
        System.out.println(stringMap);
    }

    @Test
    public void testCollectorImpl() {
        Double avg = list.stream().collect(Collectors.averagingInt(People::getAge));
        System.out.println(avg);

        OptionalDouble average = list.stream().mapToInt(People::getAge).average();
        System.out.println(average.getAsDouble());

        int[] avgs = list.stream().reduce(new int[]{0, 0}
                , (a, p) -> new int[]{a[0] + p.getAge(), a[1] + 1}
                , (a, b) -> new int[]{a[0] + b[0], a[1] + b[1]});
        System.out.println((double) avgs[0] / avgs[1]);

        Integer sum = list.stream().collect(Collectors.reducing(0, People::getAge, Integer::sum));
        System.out.println(sum);

        /**
         * 自定义的Collector第一个函数返回的是一个容器，reduce不是
         * 自定义的Collector第二个函数无返回值，reduce有，从这里就可以看出来Collector使用的是一个容器
         * 自定义的Collector比reduce多了一步对最终结果的格式化
         */
        avg = list.stream().collect(new CollectorImpl<>(
                () -> new long[2],
                (a, t) -> {
                    a[0] += t.getAge();
                    a[1]++;
                },
                (a, b) -> {
                    a[0] += b[0];
                    a[1] += b[1];
                    return a;
                },
                a -> (a[1] == 0) ? 0.0d : (double) a[0] / a[1], Collections.emptySet()));
        System.out.println(avg);
    }


}
```

