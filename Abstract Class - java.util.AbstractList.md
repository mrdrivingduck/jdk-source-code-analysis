# Abstract Class - java.util.AbstractList

Created by : Mr Dk.

2019 / 11 / 05 10:24

Nanjing, Jiangsu, China

---

## Definition

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {

}
```

`List` 的抽象实现类

实现一个随机存取的数据容器 (比如数组)

```java
/**
 * This class provides a skeletal implementation of the {@link List}
 * interface to minimize the effort required to implement this interface
 * backed by a "random access" data store (such as an array).  For sequential
 * access data (such as a linked list), {@link AbstractSequentialList} should
 * be used in preference to this class.
 *
 * <p>To implement an unmodifiable list, the programmer needs only to extend
 * this class and provide implementations for the {@link #get(int)} and
 * {@link List#size() size()} methods.
 *
 * <p>To implement a modifiable list, the programmer must additionally
 * override the {@link #set(int, Object) set(int, E)} method (which otherwise
 * throws an {@code UnsupportedOperationException}).  If the list is
 * variable-size the programmer must additionally override the
 * {@link #add(int, Object) add(int, E)} and {@link #remove(int)} methods.
 *
 * <p>The programmer should generally provide a void (no argument) and collection
 * constructor, as per the recommendation in the {@link Collection} interface
 * specification.
 *
 * <p>Unlike the other abstract collection implementations, the programmer does
 * <i>not</i> have to provide an iterator implementation; the iterator and
 * list iterator are implemented by this class, on top of the "random access"
 * methods:
 * {@link #get(int)},
 * {@link #set(int, Object) set(int, E)},
 * {@link #add(int, Object) add(int, E)} and
 * {@link #remove(int)}.
 *
 * <p>The documentation for each non-abstract method in this class describes its
 * implementation in detail.  Each of these methods may be overridden if the
 * collection being implemented admits a more efficient implementation.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @since 1.2
 */
```

---

```java
/**
 * Sole constructor.  (For invocation by subclass constructors, typically
 * implicit.)
 */
protected AbstractList() {
}
```

---

```java
/**
 * Appends the specified element to the end of this list (optional
 * operation).
 *
 * <p>Lists that support this operation may place limitations on what
 * elements may be added to this list.  In particular, some
 * lists will refuse to add null elements, and others will impose
 * restrictions on the type of elements that may be added.  List
 * classes should clearly specify in their documentation any restrictions
 * on what elements may be added.
 *
 * <p>This implementation calls {@code add(size(), e)}.
 *
 * <p>Note that this implementation throws an
 * {@code UnsupportedOperationException} unless
 * {@link #add(int, Object) add(int, E)} is overridden.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 * @throws UnsupportedOperationException if the {@code add} operation
 *         is not supported by this list
 * @throws ClassCastException if the class of the specified element
 *         prevents it from being added to this list
 * @throws NullPointerException if the specified element is null and this
 *         list does not permit null elements
 * @throws IllegalArgumentException if some property of this element
 *         prevents it from being added to this list
 */
public boolean add(E e) {
    add(size(), e);
    return true;
}

/**
 * {@inheritDoc}
 *
 * <p>This implementation always throws an
 * {@code UnsupportedOperationException}.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 * @throws IndexOutOfBoundsException     {@inheritDoc}
 */
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}
```

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation always throws an
 * {@code UnsupportedOperationException}.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws IndexOutOfBoundsException     {@inheritDoc}
 */
public E remove(int index) {
    throw new UnsupportedOperationException();
}
```

这些都是需要覆盖的，否则不支持

---

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation first gets a list iterator (with
 * {@code listIterator()}).  Then, it iterates over the list until the
 * specified element is found or the end of the list is reached.
 *
 * @throws ClassCastException   {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public int indexOf(Object o) {
    ListIterator<E> it = listIterator();
    if (o==null) {
        while (it.hasNext())
            if (it.next()==null)
                return it.previousIndex();
    } else {
        while (it.hasNext())
            if (o.equals(it.next()))
                return it.previousIndex();
    }
    return -1;
}

/**
 * {@inheritDoc}
 *
 * <p>This implementation first gets a list iterator that points to the end
 * of the list (with {@code listIterator(size())}).  Then, it iterates
 * backwards over the list until the specified element is found, or the
 * beginning of the list is reached.
 *
 * @throws ClassCastException   {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public int lastIndexOf(Object o) {
    ListIterator<E> it = listIterator(size());
    if (o==null) {
        while (it.hasPrevious())
            if (it.previous()==null)
                return it.nextIndex();
    } else {
        while (it.hasPrevious())
            if (o.equals(it.previous()))
                return it.nextIndex();
    }
    return -1;
}
```

这个 `ListIterator` 是实现了迭代器接口的两个内部类：

* `Itr` 类实现了 `Iterator` 接口
* `ListItr` 类继承了 `Itr` 类并实现了 `ListIterator` 接口

```java
private class Itr implements Iterator<E> {
    /**
        * Index of element to be returned by subsequent call to next.
        */
    int cursor = 0;

    /**
        * Index of element returned by most recent call to next or
        * previous.  Reset to -1 if this element is deleted by a call
        * to remove.
        */
    int lastRet = -1;

    /**
        * The modCount value that the iterator believes that the backing
        * List should have.  If this expectation is violated, the iterator
        * has detected concurrent modification.
        */
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size();
    }

    public E next() {
        checkForComodification();
        try {
            int i = cursor;
            E next = get(i);
            lastRet = i;
            cursor = i + 1;
            return next;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            AbstractList.this.remove(lastRet);
            if (lastRet < cursor)
                cursor--;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException e) {
            throw new ConcurrentModificationException();
        }
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

```java
private class ListItr extends Itr implements ListIterator<E> {

    ListItr(int index) {
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;
    }

    public E previous() {
        checkForComodification();
        try {
            int i = cursor - 1;
            E previous = get(i);
            lastRet = cursor = i;
            return previous;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            AbstractList.this.set(lastRet, e);
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            AbstractList.this.add(i, e);
            lastRet = -1;
            cursor = i + 1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```

由于 List 结构是随机存取的

所以这个迭代器实际上维护的是元素的 index

在构造时，就将 `index` 赋值给了内部维护的 `cursor`

那么，迭代器的 `hasNext()` 和 `hasPrevious()` 函数就可以通过判断 `cursor` 的位置来实现

---

通过不同的方法可以获取迭代器

```java
/**
 * Returns an iterator over the elements in this list in proper sequence.
 *
 * <p>This implementation returns a straightforward implementation of the
 * iterator interface, relying on the backing list's {@code size()},
 * {@code get(int)}, and {@code remove(int)} methods.
 *
 * <p>Note that the iterator returned by this method will throw an
 * {@link UnsupportedOperationException} in response to its
 * {@code remove} method unless the list's {@code remove(int)} method is
 * overridden.
 *
 * <p>This implementation can be made to throw runtime exceptions in the
 * face of concurrent modification, as described in the specification
 * for the (protected) {@link #modCount} field.
 *
 * @return an iterator over the elements in this list in proper sequence
 */
public Iterator<E> iterator() {
    return new Itr();
}
```

通过上述方法获取了一个通用迭代器

也可以获取 List 特有的迭代器：

* 由于是随机访问，那么初始化时可以指向任意元素

```java
/**
 * {@inheritDoc}
 *
 * <p>This implementation returns {@code listIterator(0)}.
 *
 * @see #listIterator(int)
 */
public ListIterator<E> listIterator() {
    return listIterator(0);
}

/**
 * {@inheritDoc}
 *
 * <p>This implementation returns a straightforward implementation of the
 * {@code ListIterator} interface that extends the implementation of the
 * {@code Iterator} interface returned by the {@code iterator()} method.
 * The {@code ListIterator} implementation relies on the backing list's
 * {@code get(int)}, {@code set(int, E)}, {@code add(int, E)}
 * and {@code remove(int)} methods.
 *
 * <p>Note that the list iterator returned by this implementation will
 * throw an {@link UnsupportedOperationException} in response to its
 * {@code remove}, {@code set} and {@code add} methods unless the
 * list's {@code remove(int)}, {@code set(int, E)}, and
 * {@code add(int, E)} methods are overridden.
 *
 * <p>This implementation can be made to throw runtime exceptions in the
 * face of concurrent modification, as described in the specification for
 * the (protected) {@link #modCount} field.
 *
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public ListIterator<E> listIterator(final int index) {
    rangeCheckForAdd(index);

    return new ListItr(index);
}
```

---

```java
/**
 * The number of times this list has been <i>structurally modified</i>.
 * Structural modifications are those that change the size of the
 * list, or otherwise perturb it in such a fashion that iterations in
 * progress may yield incorrect results.
 *
 * <p>This field is used by the iterator and list iterator implementation
 * returned by the {@code iterator} and {@code listIterator} methods.
 * If the value of this field changes unexpectedly, the iterator (or list
 * iterator) will throw a {@code ConcurrentModificationException} in
 * response to the {@code next}, {@code remove}, {@code previous},
 * {@code set} or {@code add} operations.  This provides
 * <i>fail-fast</i> behavior, rather than non-deterministic behavior in
 * the face of concurrent modification during iteration.
 *
 * <p><b>Use of this field by subclasses is optional.</b> If a subclass
 * wishes to provide fail-fast iterators (and list iterators), then it
 * merely has to increment this field in its {@code add(int, E)} and
 * {@code remove(int)} methods (and any other methods that it overrides
 * that result in structural modifications to the list).  A single call to
 * {@code add(int, E)} or {@code remove(int)} must add no more than
 * one to this field, or the iterators (and list iterators) will throw
 * bogus {@code ConcurrentModificationExceptions}.  If an implementation
 * does not wish to provide fail-fast iterators, this field may be
 * ignored.
 */
protected transient int modCount = 0;
```

这个是我认为最有意思的一个地方

这个值用于记录 List __结构性修改__ 的次数

* 结构性修改指导致 List 的 size 发生变化的操作

如果在迭代器迭代 List 的过程中，List 发生了结构性修改

那么迭代过程将会被打乱

如果这个值在意料之外地被修改

那么在迭代器调 `next()`, `remove()`, `previous()`, `set()`, `add()` 时

抛出 `ConcurrentModificationExceptions` 异常

* 这种实现是一种 fail-fast 行为
* 快速抛出错误，防止并发修改后不确定性情况的发生

那么，从具体上，这个值是怎么用的呢？

从 List 中获取迭代器时，这个值被初始化给了迭代器：

```java
/**
 * The modCount value that the iterator believes that the backing
 * List should have.  If this expectation is violated, the iterator
 * has detected concurrent modification.
 */
int expectedModCount = modCount;
```

在使用这个迭代器进行 `add()` 或 `remove()` 时

迭代器会自行维护这个 `expectedModeCount` 与 List 的 `modCount` 一致：

```java
public void add(E e) {
    checkForComodification();

    try {
        int i = cursor;
        AbstractList.this.add(i, e);
        lastRet = -1;
        cursor = i + 1;
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```

```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        AbstractList.this.remove(lastRet);
        if (lastRet < cursor)
            cursor--;
        lastRet = -1;
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException e) {
        throw new ConcurrentModificationException();
    }
}
```

同时，在操作前，迭代器都会调 `checkForComodification()` 函数来判断 `expectedModCount` 是否与 `modCount` 一致：

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

从而保证，在遍历期间

只有本迭代器才能对 List 有结构性修改的行为

其中，`expectedModCount` 由迭代器维护

`modCount` 由 List 维护

迭代器时刻负责维护 `expectedModCount` 与 `modCount` 一致

而 List 在其对应的 `add()`, `remove()` 函数中对 `modCount` 进行维护

由于本类是个抽象类，所以没有相关代码

由具体的实现类完成

---

```java
/**
 * Index of element returned by most recent call to next or
 * previous.  Reset to -1 if this element is deleted by a call
 * to remove.
 */
int lastRet = -1;
```

这个值用于记录迭代器移动后的当前元素的 index

也就是说，在调迭代器的 `next()` 或 `previous()` 后

`lastRet` 会记录当前元素的 index

在调迭代器的 `remove()` 或 `set()` 后

迭代器直接对 `lastRet` 对应的元素进行操作

如果是 `remove()` 操作，则操作完毕后将该值复位为 -1

因此重复调用 `remove()` 肯定是会出问题滴

---

