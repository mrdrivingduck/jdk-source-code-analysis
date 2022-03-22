# Interface - java.util.Iterator

Created by : Mr Dk.

2019 / 11 / 05 22:58

Nanjing, Jiangsu, China

---

## Definition

```java
public interface Iterator<E> {

}
```

集合迭代器的抽象接口。`Iterator` 和 `Enumeration` 的区别：迭代器允许调用者在迭代期间删除元素。

```java
/**
 * An iterator over a collection.  {@code Iterator} takes the place of
 * {@link Enumeration} in the Java Collections Framework.  Iterators
 * differ from enumerations in two ways:
 *
 * <ul>
 *      <li> Iterators allow the caller to remove elements from the
 *           underlying collection during the iteration with well-defined
 *           semantics.
 *      <li> Method names have been improved.
 * </ul>
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <E> the type of elements returned by this iterator
 *
 * @author  Josh Bloch
 * @see Collection
 * @see ListIterator
 * @see Iterable
 * @since 1.2
 */
```

## Has Next

返回迭代器是否还有更多元素。

```java
/**
 * Returns {@code true} if the iteration has more elements.
 * (In other words, returns {@code true} if {@link #next} would
 * return an element rather than throwing an exception.)
 *
 * @return {@code true} if the iteration has more elements
 */
boolean hasNext();
```

## Next

返回迭代的下一个元素。

```java
/**
 * Returns the next element in the iteration.
 *
 * @return the next element in the iteration
 * @throws NoSuchElementException if the iteration has no more elements
 */
E next();
```

## Remove

删除底层集合中最后一个返回的元素。对于一个元素只能被调用一次，当底层集合被迭代过程中发生了修改，则行为不确定。

```java
/**
 * Removes from the underlying collection the last element returned
 * by this iterator (optional operation).  This method can be called
 * only once per call to {@link #next}.  The behavior of an iterator
 * is unspecified if the underlying collection is modified while the
 * iteration is in progress in any way other than by calling this
 * method.
 *
 * @implSpec
 * The default implementation throws an instance of
 * {@link UnsupportedOperationException} and performs no other action.
 *
 * @throws UnsupportedOperationException if the {@code remove}
 *         operation is not supported by this iterator
 *
 * @throws IllegalStateException if the {@code next} method has not
 *         yet been called, or the {@code remove} method has already
 *         been called after the last call to the {@code next}
 *         method
 */
default void remove() {
    throw new UnsupportedOperationException("remove");
}
```

## For Each

给定一个操作，对每一个元素应用一次操作直到完成，或出现异常。

```java
/**
 * Performs the given action for each remaining element until all elements
 * have been processed or the action throws an exception.  Actions are
 * performed in the order of iteration, if that order is specified.
 * Exceptions thrown by the action are relayed to the caller.
 *
 * @implSpec
 * <p>The default implementation behaves as if:
 * <pre>{@code
 *     while (hasNext())
 *         action.accept(next());
 * }</pre>
 *
 * @param action The action to be performed for each element
 * @throws NullPointerException if the specified action is null
 * @since 1.8
 */
default void forEachRemaining(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    while (hasNext())
        action.accept(next());
}
```
