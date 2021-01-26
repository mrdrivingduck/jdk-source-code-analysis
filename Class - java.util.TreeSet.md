# Class - java.util.TreeSet

Created by : Mr Dk.

2019 / 11 / 21 23:02

Nanjing, Jiangsu, China

---

## Definition

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{

}
```

基于 `TreeMap` 实现。元素根据自然顺序或提供 `Comparator` 进行排序。内部实现保证对于增、删、查操作的复杂度都为 `log(n)`。

该实现不是线程安全的：

* 要么在外部被同步
* 要么就 - `SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));`

```java
/**
 * A {@link NavigableSet} implementation based on a {@link TreeMap}.
 * The elements are ordered using their {@linkplain Comparable natural
 * ordering}, or by a {@link Comparator} provided at set creation
 * time, depending on which constructor is used.
 *
 * <p>This implementation provides guaranteed log(n) time cost for the basic
 * operations ({@code add}, {@code remove} and {@code contains}).
 *
 * <p>Note that the ordering maintained by a set (whether or not an explicit
 * comparator is provided) must be <i>consistent with equals</i> if it is to
 * correctly implement the {@code Set} interface.  (See {@code Comparable}
 * or {@code Comparator} for a precise definition of <i>consistent with
 * equals</i>.)  This is so because the {@code Set} interface is defined in
 * terms of the {@code equals} operation, but a {@code TreeSet} instance
 * performs all element comparisons using its {@code compareTo} (or
 * {@code compare}) method, so two elements that are deemed equal by this method
 * are, from the standpoint of the set, equal.  The behavior of a set
 * <i>is</i> well-defined even if its ordering is inconsistent with equals; it
 * just fails to obey the general contract of the {@code Set} interface.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a tree set concurrently, and at least one
 * of the threads modifies the set, it <i>must</i> be synchronized
 * externally.  This is typically accomplished by synchronizing on some
 * object that naturally encapsulates the set.
 * If no such object exists, the set should be "wrapped" using the
 * {@link Collections#synchronizedSortedSet Collections.synchronizedSortedSet}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the set: <pre>
 *   SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));</pre>
 *
 * <p>The iterators returned by this class's {@code iterator} method are
 * <i>fail-fast</i>: if the set is modified at any time after the iterator is
 * created, in any way except through the iterator's own {@code remove}
 * method, the iterator will throw a {@link ConcurrentModificationException}.
 * Thus, in the face of concurrent modification, the iterator fails quickly
 * and cleanly, rather than risking arbitrary, non-deterministic behavior at
 * an undetermined time in the future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw {@code ConcurrentModificationException} on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:   <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <E> the type of elements maintained by this set
 *
 * @author  Josh Bloch
 * @see     Collection
 * @see     Set
 * @see     HashSet
 * @see     Comparable
 * @see     Comparator
 * @see     TreeMap
 * @since   1.2
 */
```

## Fields

成员变量内部实际上维护了一个 `NavigableMap<E,Object>`，而 Object 都是 "哑" 的 (没卵用的)。

```java
/**
 * The backing map.
 */
private transient NavigableMap<E,Object> m;

// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```

## Constructor

构造函数，使用了 `TreeMap` 的构造函数。其中，如果构造来源有 `Comparator`，则复制。

```java
/**
 * Constructs a set backed by the specified navigable map.
 */
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}

/**
 * Constructs a new, empty tree set, sorted according to the
 * natural ordering of its elements.  All elements inserted into
 * the set must implement the {@link Comparable} interface.
 * Furthermore, all such elements must be <i>mutually
 * comparable</i>: {@code e1.compareTo(e2)} must not throw a
 * {@code ClassCastException} for any elements {@code e1} and
 * {@code e2} in the set.  If the user attempts to add an element
 * to the set that violates this constraint (for example, the user
 * attempts to add a string element to a set whose elements are
 * integers), the {@code add} call will throw a
 * {@code ClassCastException}.
 */
public TreeSet() {
    this(new TreeMap<E,Object>());
}

/**
 * Constructs a new, empty tree set, sorted according to the specified
 * comparator.  All elements inserted into the set must be <i>mutually
 * comparable</i> by the specified comparator: {@code comparator.compare(e1,
 * e2)} must not throw a {@code ClassCastException} for any elements
 * {@code e1} and {@code e2} in the set.  If the user attempts to add
 * an element to the set that violates this constraint, the
 * {@code add} call will throw a {@code ClassCastException}.
 *
 * @param comparator the comparator that will be used to order this set.
 *        If {@code null}, the {@linkplain Comparable natural
 *        ordering} of the elements will be used.
 */
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

/**
 * Constructs a new tree set containing the elements in the specified
 * collection, sorted according to the <i>natural ordering</i> of its
 * elements.  All elements inserted into the set must implement the
 * {@link Comparable} interface.  Furthermore, all such elements must be
 * <i>mutually comparable</i>: {@code e1.compareTo(e2)} must not throw a
 * {@code ClassCastException} for any elements {@code e1} and
 * {@code e2} in the set.
 *
 * @param c collection whose elements will comprise the new set
 * @throws ClassCastException if the elements in {@code c} are
 *         not {@link Comparable}, or are not mutually comparable
 * @throws NullPointerException if the specified collection is null
 */
public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

/**
 * Constructs a new tree set containing the same elements and
 * using the same ordering as the specified sorted set.
 *
 * @param s sorted set whose elements will comprise the new set
 * @throws NullPointerException if the specified sorted set is null
 */
public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```

## Iterator

迭代器及反向视角，借用了 `TreeMap` keySet 的迭代器。

```java
/**
 * Returns an iterator over the elements in this set in ascending order.
 *
 * @return an iterator over the elements in this set in ascending order
 */
public Iterator<E> iterator() {
    return m.navigableKeySet().iterator();
}

/**
 * Returns an iterator over the elements in this set in descending order.
 *
 * @return an iterator over the elements in this set in descending order
 * @since 1.6
 */
public Iterator<E> descendingIterator() {
    return m.descendingKeySet().iterator();
}

/**
 * @since 1.6
 */
public NavigableSet<E> descendingSet() {
    return new TreeSet<>(m.descendingMap());
}
```

## Collection Status

```java
/**
 * Returns the number of elements in this set (its cardinality).
 *
 * @return the number of elements in this set (its cardinality)
 */
public int size() {
    return m.size();
}

/**
 * Returns {@code true} if this set contains no elements.
 *
 * @return {@code true} if this set contains no elements
 */
public boolean isEmpty() {
    return m.isEmpty();
}
```

## Contains

利用 `TreeMap` 的函数查询 key 是否存在。

```java
/**
 * Returns {@code true} if this set contains the specified element.
 * More formally, returns {@code true} if and only if this set
 * contains an element {@code e} such that
 * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>.
 *
 * @param o object to be checked for containment in this set
 * @return {@code true} if this set contains the specified element
 * @throws ClassCastException if the specified object cannot be compared
 *         with the elements currently in the set
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 */
public boolean contains(Object o) {
    return m.containsKey(o);
}
```

## Add

将元素插入集合：

* key 是元素本身
* value 是没有用处的 "哑" 元素

```java
/**
 * Adds the specified element to this set if it is not already present.
 * More formally, adds the specified element {@code e} to this set if
 * the set contains no element {@code e2} such that
 * <tt>(e==null&nbsp;?&nbsp;e2==null&nbsp;:&nbsp;e.equals(e2))</tt>.
 * If this set already contains the element, the call leaves the set
 * unchanged and returns {@code false}.
 *
 * @param e element to be added to this set
 * @return {@code true} if this set did not already contain the specified
 *         element
 * @throws ClassCastException if the specified object cannot be compared
 *         with the elements currently in this set
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 */
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```

## Remove

从集合中移除元素。借用 `TreeMap` 的 `remove()`，以及清空集合。

```java
/**
 * Removes the specified element from this set if it is present.
 * More formally, removes an element {@code e} such that
 * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>,
 * if this set contains such an element.  Returns {@code true} if
 * this set contained the element (or equivalently, if this set
 * changed as a result of the call).  (This set will not contain the
 * element once the call returns.)
 *
 * @param o object to be removed from this set, if present
 * @return {@code true} if this set contained the specified element
 * @throws ClassCastException if the specified object cannot be compared
 *         with the elements currently in this set
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 */
public boolean remove(Object o) {
    return m.remove(o)==PRESENT;
}

/**
 * Removes all of the elements from this set.
 * The set will be empty after this call returns.
 */
public void clear() {
    m.clear();
}
```

## AddAll

似乎需要判断一下 `Comparator` 是否相同

* 如果相同，就使用 `TreeMap` 的 `buildFromSorted()` 将元素插入结合
* 否则就使用默认的迭代每个元素的方式插入结合

```java
/**
 * Adds all of the elements in the specified collection to this set.
 *
 * @param c collection containing elements to be added to this set
 * @return {@code true} if this set changed as a result of the call
 * @throws ClassCastException if the elements provided cannot be compared
 *         with the elements currently in the set
 * @throws NullPointerException if the specified collection is null or
 *         if any element is null and this set uses natural ordering, or
 *         its comparator does not permit null elements
 */
public  boolean addAll(Collection<? extends E> c) {
    // Use linear-time version if applicable
    if (m.size()==0 && c.size() > 0 &&
        c instanceof SortedSet &&
        m instanceof TreeMap) {
        SortedSet<? extends E> set = (SortedSet<? extends E>) c;
        TreeMap<E,Object> map = (TreeMap<E, Object>) m;
        Comparator<?> cc = set.comparator();
        Comparator<? super E> mc = map.comparator();
        if (cc==mc || (cc != null && cc.equals(mc))) {
            map.addAllForTreeSet(set, PRESENT);
            return true;
        }
    }
    return super.addAll(c);
}
```

## SubSet

取子集的函数，借用了 `TreeMap` 的方法实现。

```java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if {@code fromElement} or {@code toElement}
 *         is null and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @throws IllegalArgumentException {@inheritDoc}
 * @since 1.6
 */
public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                                E toElement,   boolean toInclusive) {
    return new TreeSet<>(m.subMap(fromElement, fromInclusive,
                                    toElement,   toInclusive));
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if {@code toElement} is null and
 *         this set uses natural ordering, or its comparator does
 *         not permit null elements
 * @throws IllegalArgumentException {@inheritDoc}
 * @since 1.6
 */
public NavigableSet<E> headSet(E toElement, boolean inclusive) {
    return new TreeSet<>(m.headMap(toElement, inclusive));
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if {@code fromElement} is null and
 *         this set uses natural ordering, or its comparator does
 *         not permit null elements
 * @throws IllegalArgumentException {@inheritDoc}
 * @since 1.6
 */
public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
    return new TreeSet<>(m.tailMap(fromElement, inclusive));
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if {@code fromElement} or
 *         {@code toElement} is null and this set uses natural ordering,
 *         or its comparator does not permit null elements
 * @throws IllegalArgumentException {@inheritDoc}
 */
public SortedSet<E> subSet(E fromElement, E toElement) {
    return subSet(fromElement, true, toElement, false);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if {@code toElement} is null
 *         and this set uses natural ordering, or its comparator does
 *         not permit null elements
 * @throws IllegalArgumentException {@inheritDoc}
 */
public SortedSet<E> headSet(E toElement) {
    return headSet(toElement, false);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if {@code fromElement} is null
 *         and this set uses natural ordering, or its comparator does
 *         not permit null elements
 * @throws IllegalArgumentException {@inheritDoc}
 */
public SortedSet<E> tailSet(E fromElement) {
    return tailSet(fromElement, true);
}
```

## Range Operation

借用 `TreeMap` 的函数，取不同条件范围下的 key。

```java
/**
 * @throws NoSuchElementException {@inheritDoc}
 */
public E first() {
    return m.firstKey();
}

/**
 * @throws NoSuchElementException {@inheritDoc}
 */
public E last() {
    return m.lastKey();
}

// NavigableSet API methods

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E lower(E e) {
    return m.lowerKey(e);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E floor(E e) {
    return m.floorKey(e);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E ceiling(E e) {
    return m.ceilingKey(e);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E higher(E e) {
    return m.higherKey(e);
}
```

## Poll

返回并删除第一个或最后一个元素。

```java
/**
 * @since 1.6
 */
public E pollFirst() {
    Map.Entry<E,?> e = m.pollFirstEntry();
    return (e == null) ? null : e.getKey();
}

/**
 * @since 1.6
 */
public E pollLast() {
    Map.Entry<E,?> e = m.pollLastEntry();
    return (e == null) ? null : e.getKey();
}
```

## Summary

佛了，全是借用对应的 Map 的函数，所以根本没什么看头。😑

---

