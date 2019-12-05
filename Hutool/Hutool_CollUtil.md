## 集合操作的工具类 CollUtil
### isEmpty、isNotEmpty
判断集合是否为空（包括null和没有元素的集合）。

```java
boolean isEmpty(Collection<?> collection)
boolean isNotEmpty(Collection<?> collection)
boolean isNotEmpty(Map<?,?> map)
boolean isEmpty(Map<?,?> map)
```

### list、newArrayList 创建集合
里面有很多方法，创建list、set、map的，不一一列举了。

```java
//新建一个空List。isLinked - 是否新建LinkedList
List<T> list(boolean isLinked)
List<T> list(boolean isLinked, T... values)
List<T> list(boolean isLinked, Collection<T> collection)
List<T> list(boolean isLinked, Iterable<T> iterable)
List<T> list(boolean isLinked, Iterator<T> iter)
List<T> list(boolean isLinked, Enumeration<T> enumration)
ArrayList<T> newArrayList(T... values)
ArrayList<T> toList(T... values)
LinkedList<T> newLinkedList(T... values)
```

### join 
类似于python的join和scala的mkString。以 conjunction 为分隔符将集合转换为字符串，如果集合元素为数组、Iterable或Iterator，则递归组合其为字符串
```java
String join(Iterable<T> iterable, CharSequence conjunction)
String join(Iterator<T> iterator,, CharSequence conjunction)
```

### sub方法
类似于python的切片，支持负数索引，步长。
```java
List<T> sub(List<T> list, int start, int end, int step)
List<T> sub(List<T> list, int start, int end)
List<T> sub(Collection<T> list, int start, int end, int step)
List<T> sub(Collection<T> list, int start, int end)
```

### countMap
根据集合返回一个元素计数的 Map，所谓元素计数就是假如这个集合中某个元素出现了n次，那将这个元素做为key，n做为value。

```java
Map<T,Integer> countMap(Iterable<T> collection)
```

### page、sortPageAll
page对List分页、sortPageAll对多个集合排序后分页。第二个参数是pageSize每页的条目数

```java
List<T> page(int pageNo, int pageSize, List<T> list)
List<T> sortPageAll(int pageNo, int pageSize, Comparator<T> comparator, Collection<T>... colls)
```
