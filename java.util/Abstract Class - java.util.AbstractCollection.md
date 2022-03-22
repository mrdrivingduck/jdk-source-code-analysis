# Abstract Class - java.util.AbstractCollection

Created by : Mr Dk.

2019 / 11 / 04 11:07

Nanjing, Jiangsu, China

---

## Definition

```java
public abstract class AbstractCollection<E> implements Collection<E> {

}
```

实现 `Collection` 接口的基本抽象类

- 为了实现不可修改的集合，必须实现 `iterator()` 和 `size()` 函数，返回的迭代器必须实现 `hasNext()` 和 `next()`
- 为了实现可修改的集合，必须额外 override `add()` 函数，返回的迭代器必须实现 `remove()` 函数

```java
/**
 * This class provides a skeletal implementation of the <tt>Collection</tt>
 * interface, to minimize the effort required to implement this interface. <p>
 *
 * To implement an unmodifiable collection, the programmer needs only to
 * extend this class and provide implementations for the <tt>iterator</tt> and
 * <tt>size</tt> methods.  (The iterator returned by the <tt>iterator</tt>
 * method must implement <tt>hasNext</tt> and <tt>next</tt>.)<p>
 *
 * To implement a modifiable collection, the programmer must additionally
 * override this class's <tt>add</tt> method (which otherwise throws an
 * <tt>UnsupportedOperationException</tt>), and the iterator returned by the
 * <tt>iterator</tt> method must additionally implement its <tt>remove</tt>
 * method.<p>
 *
 * The programmer should generally provide a void (no argument) and
 * <tt>Collection</tt> constructor, as per the recommendation in the
 * <tt>Collection</tt> interface specification.<p>
 *
 * The documentation for each non-abstract method in this class describes its
 * implementation in detail.  Each of these methods may be overridden if
 * the collection being implemented admits a more efficient implementation.<p>
 *
 * This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see Collection
 * @since 1.2
 */
```

## Constructor

```java
/**
 * Sole constructor.  (For invocation by subclass constructors, typically
 * implicit.)
 */
protected AbstractCollection() {
}
```

## Iterator

```java
/**
 * Returns an iterator over the elements contained in this collection.
 *
 * @return an iterator over the elements contained in this collection
 */
public abstract Iterator<E> iterator();
```

## Size

```java
public abstract int size();

public boolean isEmpty() {
    return size() == 0;
}
```

## Contains

遍历集合中的每个元素，查看是否存在相同的元素。注意比较用的是 `equals()`，默认比较内存地址 (是否是同一个对象)——如果想要比较内容，需要 override `equals()`：

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over the elements in the collection,
 * checking each element in turn for equality with the specified element.
 *
 * @throws ClassCastException   {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public boolean contains(Object o) {
    Iterator<E> it = iterator();
    if (o==null) {
        while (it.hasNext())
            if (it.next()==null)
                return true;
    } else {
        while (it.hasNext())
            if (o.equals(it.next()))
                return true;
    }
    return false;
}
```

## To Array

返回与迭代器遍历顺序相同的元素。

首先根据集合的大小，分配好对应大小的空间；同时做好准备：实际的元素可能少或更多。

- 如果数量与实际相同，则返回 `r`
- 如果实际元素数量少了，就只返回这么多元素
- 如果实际元素数量多了，就还要在 `finishToArray()` 中把迭代器遍历完，返回一个更大的数组

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation returns an array containing all the elements
 * returned by this collection's iterator, in the same order, stored in
 * consecutive elements of the array, starting with index {@code 0}.
 * The length of the returned array is equal to the number of elements
 * returned by the iterator, even if the size of this collection changes
 * during iteration, as might happen if the collection permits
 * concurrent modification during iteration.  The {@code size} method is
 * called only as an optimization hint; the correct result is returned
 * even if the iterator returns a different number of elements.
 *
 * <p>This method is equivalent to:
 *
 *  <pre> {@code
 * List<E> list = new ArrayList<E>(size());
 * for (E e : this)
 *     list.add(e);
 * return list.toArray();
 * }</pre>
 */
public Object[] toArray() {
    // Estimate size of array; be prepared to see more or fewer elements
    Object[] r = new Object[size()];
    Iterator<E> it = iterator();
    for (int i = 0; i < r.length; i++) {
        if (! it.hasNext()) // fewer elements than expected
            return Arrays.copyOf(r, i);
        r[i] = it.next();
    }
    return it.hasNext() ? finishToArray(r, it) : r;
}
```

`cap + (cap >> 1) + 1` 是将数组扩容到当前长度的 1.5 倍？

数组的扩容大小不能超过上限，否则会溢出。重新分配完毕后，开始将迭代器未迭代到的元素加入数组。迭代完毕后，如果数组后还有多余空间，则放弃多余空间后返回。

```java
/**
 * The maximum size of array to allocate.
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * Reallocates the array being used within toArray when the iterator
 * returned more elements than expected, and finishes filling it from
 * the iterator.
 *
 * @param r the array, replete with previously stored elements
 * @param it the in-progress iterator over this collection
 * @return array containing the elements in the given array, plus any
 *         further elements returned by the iterator, trimmed to size
 */
@SuppressWarnings("unchecked")
private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
    int i = r.length;
    while (it.hasNext()) {
        int cap = r.length;
        if (i == cap) {
            int newCap = cap + (cap >> 1) + 1;
            // overflow-conscious code
            if (newCap - MAX_ARRAY_SIZE > 0)
                newCap = hugeCapacity(cap + 1);
            r = Arrays.copyOf(r, newCap);
        }
        r[i++] = (T)it.next();
    }
    // trim if overallocated
    return (i == r.length) ? r : Arrays.copyOf(r, i);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError
            ("Required array size too large");
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

## Add / Remove

可修改集合必须实现该函数，否则异常。

```java
public boolean add(E e) {
    throw new UnsupportedOperationException();
}
```

对集合进行迭代，如果找到对应元素，调用迭代器的 `remove()` 函数删掉元素。

> 所以实际删除元素的操作是由迭代器来做的。难怪用迭代器迭代时，不能对集合进行修改，不然相当于两个迭代器在并发对集合进行操作，可能有安全问题。

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over the collection looking for the
 * specified element.  If it finds the element, it removes the element
 * from the collection using the iterator's remove method.
 *
 * <p>Note that this implementation throws an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by this
 * collection's iterator method does not implement the <tt>remove</tt>
 * method and this collection contains the specified object.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 */
public boolean remove(Object o) {
    Iterator<E> it = iterator();
    if (o==null) {
        while (it.hasNext()) {
            if (it.next()==null) {
                it.remove();
                return true;
            }
        }
    } else {
        while (it.hasNext()) {
            if (o.equals(it.next())) {
                it.remove();
                return true;
            }
        }
    }
    return false;
}
```

## 集合操作

迭代输入集合中的每一个元素。如果有本集合不存在的元素，则立刻退出。

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over the specified collection,
 * checking each element returned by the iterator in turn to see
 * if it's contained in this collection.  If all elements are so
 * contained <tt>true</tt> is returned, otherwise <tt>false</tt>.
 *
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @see #contains(Object)
 */
public boolean containsAll(Collection<?> c) {
    for (Object e : c)
        if (!contains(e))
            return false;
    return true;
}
```

迭代输入集合中的每个元素，并接入到本集合中。

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over the specified collection, and adds
 * each object returned by the iterator to this collection, in turn.
 *
 * <p>Note that this implementation will throw an
 * <tt>UnsupportedOperationException</tt> unless <tt>add</tt> is
 * overridden (assuming the specified collection is non-empty).
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 * @throws IllegalStateException         {@inheritDoc}
 *
 * @see #add(Object)
 */
public boolean addAll(Collection<? extends E> c) {
    boolean modified = false;
    for (E e : c)
        if (add(e))
            modified = true;
    return modified;
}
```

迭代本集合中的每个元素。如果出现在了输入集合中，就用迭代器的 `remove()` 函数在本集合中删掉该元素。

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over this collection, checking each
 * element returned by the iterator in turn to see if it's contained
 * in the specified collection.  If it's so contained, it's removed from
 * this collection with the iterator's <tt>remove</tt> method.
 *
 * <p>Note that this implementation will throw an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by the
 * <tt>iterator</tt> method does not implement the <tt>remove</tt> method
 * and this collection contains one or more elements in common with the
 * specified collection.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 *
 * @see #remove(Object)
 * @see #contains(Object)
 */
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    boolean modified = false;
    Iterator<?> it = iterator();
    while (it.hasNext()) {
        if (c.contains(it.next())) {
            it.remove();
            modified = true;
        }
    }
    return modified;
}
```

用迭代器迭代本集合。删除所有不出现在输入集合 `c` 中的元素。

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over this collection, checking each
 * element returned by the iterator in turn to see if it's contained
 * in the specified collection.  If it's not so contained, it's removed
 * from this collection with the iterator's <tt>remove</tt> method.
 *
 * <p>Note that this implementation will throw an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by the
 * <tt>iterator</tt> method does not implement the <tt>remove</tt> method
 * and this collection contains one or more elements not present in the
 * specified collection.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 *
 * @see #remove(Object)
 * @see #contains(Object)
 */
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    boolean modified = false;
    Iterator<E> it = iterator();
    while (it.hasNext()) {
        if (!c.contains(it.next())) {
            it.remove();
            modified = true;
        }
    }
    return modified;
}
```

用迭代器迭代每一个元素并删除。具体实现类应该会覆盖该函数，以得到更好的性能。

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over this collection, removing each
 * element using the <tt>Iterator.remove</tt> operation.  Most
 * implementations will probably choose to override this method for
 * efficiency.
 *
 * <p>Note that this implementation will throw an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by this
 * collection's <tt>iterator</tt> method does not implement the
 * <tt>remove</tt> method and this collection is non-empty.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 */
public void clear() {
    Iterator<E> it = iterator();
    while (it.hasNext()) {
        it.next();
        it.remove();
    }
}
```

## To String

用 `[]` 和 `,` 将集合用字符串表示。

> 那个 `(this Collection)` 是啥意思？？？

```java
/**
 * Returns a string representation of this collection.  The string
 * representation consists of a list of the collection's elements in the
 * order they are returned by its iterator, enclosed in square brackets
 * (<tt>"[]"</tt>).  Adjacent elements are separated by the characters
 * <tt>", "</tt> (comma and space).  Elements are converted to strings as
 * by {@link String#valueOf(Object)}.
 *
 * @return a string representation of this collection
 */
public String toString() {
    Iterator<E> it = iterator();
    if (! it.hasNext())
        return "[]";

    StringBuilder sb = new StringBuilder();
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (! it.hasNext())
            return sb.append(']').toString();
        sb.append(',').append(' ');
    }
}
```
