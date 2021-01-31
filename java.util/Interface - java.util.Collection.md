# Interface - java.util.Collection

Created by : Mr Dk.

2019 / 11 / 03 18:04

Nanjing, Jiangsu, China

---

## Definition

```java
/**
 * @param <E> the type of elements in this collection
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see     Set
 * @see     List
 * @see     Map
 * @see     SortedSet
 * @see     SortedMap
 * @see     HashSet
 * @see     TreeSet
 * @see     ArrayList
 * @see     LinkedList
 * @see     Vector
 * @see     Collections
 * @see     Arrays
 * @see     AbstractCollection
 * @since 1.2
 */
public interface Collection<E> extends Iterable<E> {

}
```

是集合的最基本的接口。集合表示了一组对象，有些集合允许存在重复的对象，有些则不行；有些集合是有序的，而有些是无序的、这个接口用于定义所有集合的通用行为，针对不同类型的集合还有子接口。

所有通用集合的实现类应当至少提供两个标准构造函数：

* 一个空参数构造函数 - 用于创建空集合
* 一个同类型的单参数构造函数 - 创建一个新的具有相同元素的集合 (拷贝构造)

不同类型的集合可能有不同的限制：有些集合禁止空元素；有些集合对数据类型有限制。每个集合自己决定是否实现同步。

---

返回集合中的元素个数。如果结合中的元素多于 `Integer.MAX_VALUE`，则返回 `Integer.MAX_VALUE`。

```java
int size();
```

> 顺便记录一下 code style。最近在看 alibaba 的 Java 开发手册，interface 中的函数不加 `public`。

---

集合是否不包含元素

```java
boolean isEmpty();
```

---

返回集合中是否存在特定元素，当且仅当集合中至少有一个元素时返回 `true`。

```java
boolean contains(Object o);
```

---

返回遍历集合元素的迭代器。

```java
Iterator<E> iterator();
```

---

返回一个包含所有元素的数组。如果集合能够保证迭代器返回元素的顺序，这个函数也需要保证相同的顺序。如果集合对象内部维护了一个数组的话，该函数返回的数组一定是一个新的拷贝。

```java
Object[] toArray();
```

---

如果集合被修改，返回 `true`；如果集合不允许重复并已有元素存在，返回 `false`。

```java
boolean add(E e);
```

---

如果集合中存在一个或多个这样的元素，就删掉一个。

```java
boolean remove(Object o);
```

---

如果集合中包含所有 `c` 中指定的元素，返回 `true`。

```java
boolean containsAll(Collection<?> c);
```

---

将 `c` 中所有元素加入集合。如果特定结合在插入过程中被修改，那么操作状态未知 (暗示了自己不能 addAll 自己)。

```java
boolean addAll(Collection<? extends E> c);
```

---

删除所有也在 `c` 中出现的元素。操作结束后，只剩下 `c` 中不出现的元素 (差集)。

```java
boolean removeAll(Collection<?> c);
```

---

只保留也在 `c` 中出现的元素 (交集)。

```java
boolean retainAll(Collection<?> c);
```

---

清空集合。

```java
void clear();
```

---

移除所有满足给定条件的元素。默认的实现：用迭代器迭代集合，用迭代器的 `Iterator.remove()` 移除元素。

```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```

---

```java
boolean equals(Object o);
```

---

返回集合的 hashcode。如果覆盖了 `equals()` 则必须覆盖 `hashcode()`。`c1.equals(c2)` 意味着 `c1.hashCode()==c2.hashCode()`。

```java
int hashCode();
```

---

```java
@Override
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, 0);
}

default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}

default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}
```

这个..这个是干啥的..暂时还不懂

---

