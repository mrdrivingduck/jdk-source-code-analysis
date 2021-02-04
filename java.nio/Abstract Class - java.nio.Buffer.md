# Abstract Class - java.nio.Buffer

Created by : Mr Dk.

2021 / 02 / 04 10:52

Ningbo, Zhejiang, China

---

## Definition

定义了 NIO 中的关键组件 - 用于缓存数据的 buffer。Buffer 实际上在底层维护了一段数组内存，使用几个关键的指针维护这段内存的使用：

* `capacity` - 内存容量
* `limit` - 第一个不应当被读取或写入的位置下标 (≤ capacity)
    * 对于读取来说，大致指向最后一个可读的位置
    * 对于写入来说，大致指向最后一个可写的位置
* `position` - 下一个被读取或被写入的位置下标，一开始为 `0`

也就是说，对于读取场景：

```
|dddddddddd        |
    p      l       c
```

对于写入场景：

```
|dddddddddd        |
           p      lc
```

此外，还支持 mark / reset。综上，这些维护数据的变量满足以下条件：

```
0 ≤ mark ≤ position ≤ limit ≤ capacity
```

基本操作：

* `clear` - 重设内部指针，使 buffer 恢复初始可写状态
* `flip` - 重设内部指针，使 buffer 进入可读状态
* `rewind` - 重设 `position` 至 0，重新读取 buffer 中的数据

一些 buffer 可以是只读的，指针可变，但数组中的数据不可变，否则将抛出 `ReadOnlyBufferException` 异常。

对象不保证线程安全。另外，大部分函数都返回对象本身，以便支持链式调用。

```java
/**
 * A container for data of a specific primitive type.
 *
 * <p> A buffer is a linear, finite sequence of elements of a specific
 * primitive type.  Aside from its content, the essential properties of a
 * buffer are its capacity, limit, and position: </p>
 *
 * <blockquote>
 *
 *   <p> A buffer's <i>capacity</i> is the number of elements it contains.  The
 *   capacity of a buffer is never negative and never changes.  </p>
 *
 *   <p> A buffer's <i>limit</i> is the index of the first element that should
 *   not be read or written.  A buffer's limit is never negative and is never
 *   greater than its capacity.  </p>
 *
 *   <p> A buffer's <i>position</i> is the index of the next element to be
 *   read or written.  A buffer's position is never negative and is never
 *   greater than its limit.  </p>
 *
 * </blockquote>
 *
 * <p> There is one subclass of this class for each non-boolean primitive type.
 *
 *
 * <h2> Transferring data </h2>
 *
 * <p> Each subclass of this class defines two categories of <i>get</i> and
 * <i>put</i> operations: </p>
 *
 * <blockquote>
 *
 *   <p> <i>Relative</i> operations read or write one or more elements starting
 *   at the current position and then increment the position by the number of
 *   elements transferred.  If the requested transfer exceeds the limit then a
 *   relative <i>get</i> operation throws a {@link BufferUnderflowException}
 *   and a relative <i>put</i> operation throws a {@link
 *   BufferOverflowException}; in either case, no data is transferred.  </p>
 *
 *   <p> <i>Absolute</i> operations take an explicit element index and do not
 *   affect the position.  Absolute <i>get</i> and <i>put</i> operations throw
 *   an {@link IndexOutOfBoundsException} if the index argument exceeds the
 *   limit.  </p>
 *
 * </blockquote>
 *
 * <p> Data may also, of course, be transferred in to or out of a buffer by the
 * I/O operations of an appropriate channel, which are always relative to the
 * current position.
 *
 *
 * <h2> Marking and resetting </h2>
 *
 * <p> A buffer's <i>mark</i> is the index to which its position will be reset
 * when the {@link #reset reset} method is invoked.  The mark is not always
 * defined, but when it is defined it is never negative and is never greater
 * than the position.  If the mark is defined then it is discarded when the
 * position or the limit is adjusted to a value smaller than the mark.  If the
 * mark is not defined then invoking the {@link #reset reset} method causes an
 * {@link InvalidMarkException} to be thrown.
 *
 *
 * <h2> Invariants </h2>
 *
 * <p> The following invariant holds for the mark, position, limit, and
 * capacity values:
 *
 * <blockquote>
 *     <tt>0</tt> <tt>&lt;=</tt>
 *     <i>mark</i> <tt>&lt;=</tt>
 *     <i>position</i> <tt>&lt;=</tt>
 *     <i>limit</i> <tt>&lt;=</tt>
 *     <i>capacity</i>
 * </blockquote>
 *
 * <p> A newly-created buffer always has a position of zero and a mark that is
 * undefined.  The initial limit may be zero, or it may be some other value
 * that depends upon the type of the buffer and the manner in which it is
 * constructed.  Each element of a newly-allocated buffer is initialized
 * to zero.
 *
 *
 * <h2> Clearing, flipping, and rewinding </h2>
 *
 * <p> In addition to methods for accessing the position, limit, and capacity
 * values and for marking and resetting, this class also defines the following
 * operations upon buffers:
 *
 * <ul>
 *
 *   <li><p> {@link #clear} makes a buffer ready for a new sequence of
 *   channel-read or relative <i>put</i> operations: It sets the limit to the
 *   capacity and the position to zero.  </p></li>
 *
 *   <li><p> {@link #flip} makes a buffer ready for a new sequence of
 *   channel-write or relative <i>get</i> operations: It sets the limit to the
 *   current position and then sets the position to zero.  </p></li>
 *
 *   <li><p> {@link #rewind} makes a buffer ready for re-reading the data that
 *   it already contains: It leaves the limit unchanged and sets the position
 *   to zero.  </p></li>
 *
 * </ul>
 *
 *
 * <h2> Read-only buffers </h2>
 *
 * <p> Every buffer is readable, but not every buffer is writable.  The
 * mutation methods of each buffer class are specified as <i>optional
 * operations</i> that will throw a {@link ReadOnlyBufferException} when
 * invoked upon a read-only buffer.  A read-only buffer does not allow its
 * content to be changed, but its mark, position, and limit values are mutable.
 * Whether or not a buffer is read-only may be determined by invoking its
 * {@link #isReadOnly isReadOnly} method.
 *
 *
 * <h2> Thread safety </h2>
 *
 * <p> Buffers are not safe for use by multiple concurrent threads.  If a
 * buffer is to be used by more than one thread then access to the buffer
 * should be controlled by appropriate synchronization.
 *
 *
 * <h2> Invocation chaining </h2>
 *
 * <p> Methods in this class that do not otherwise have a value to return are
 * specified to return the buffer upon which they are invoked.  This allows
 * method invocations to be chained; for example, the sequence of statements
 *
 * <blockquote><pre>
 * b.flip();
 * b.position(23);
 * b.limit(42);</pre></blockquote>
 *
 * can be replaced by the single, more compact statement
 *
 * <blockquote><pre>
 * b.flip().position(23).limit(42);</pre></blockquote>
 *
 *
 * @author Mark Reinhold
 * @author JSR-51 Expert Group
 * @since 1.4
 */

public abstract class Buffer {

}
```

## Fields and Constructor

如上所述，内部维护数组空间的几个关键变量：

```java
// Invariants: mark <= position <= limit <= capacity
private int mark = -1;
private int position = 0;
private int limit;
private int capacity;
```

构造函数可以指定这几个变量的初始位置，但前提是要满足上述不等式条件：

```java
// Creates a new buffer with the given mark, position, limit, and capacity,
// after checking invariants.
//
Buffer(int mark, int pos, int lim, int cap) {       // package-private
    if (cap < 0)
        throw new IllegalArgumentException("Negative capacity: " + cap);
    this.capacity = cap;
    limit(lim);
    position(pos);
    if (mark >= 0) {
        if (mark > pos)
            throw new IllegalArgumentException("mark > position: ("
                                                + mark + " > " + pos + ")");
        this.mark = mark;
    }
}
```

```java
/**
 * Sets this buffer's position.  If the mark is defined and larger than the
 * new position then it is discarded.
 *
 * @param  newPosition
 *         The new position value; must be non-negative
 *         and no larger than the current limit
 *
 * @return  This buffer
 *
 * @throws  IllegalArgumentException
 *          If the preconditions on <tt>newPosition</tt> do not hold
 */
public final Buffer position(int newPosition) {
    if ((newPosition > limit) || (newPosition < 0))
        throw new IllegalArgumentException();
    position = newPosition;
    if (mark > position) mark = -1;
    return this;
}

/**
 * Sets this buffer's limit.  If the position is larger than the new limit
 * then it is set to the new limit.  If the mark is defined and larger than
 * the new limit then it is discarded.
 *
 * @param  newLimit
 *         The new limit value; must be non-negative
 *         and no larger than this buffer's capacity
 *
 * @return  This buffer
 *
 * @throws  IllegalArgumentException
 *          If the preconditions on <tt>newLimit</tt> do not hold
 */
public final Buffer limit(int newLimit) {
    if ((newLimit > capacity) || (newLimit < 0))
        throw new IllegalArgumentException();
    limit = newLimit;
    if (position > limit) position = limit;
    if (mark > limit) mark = -1;
    return this;
}
```

## State

获取维护数据的状态变量：

```java
/**
 * Returns this buffer's capacity.
 *
 * @return  The capacity of this buffer
 */
public final int capacity() {
    return capacity;
}

/**
 * Returns this buffer's position.
 *
 * @return  The position of this buffer
 */
public final int position() {
    return position;
}

/**
 * Returns this buffer's limit.
 *
 * @return  The limit of this buffer
 */
public final int limit() {
    return limit;
}
```

## Mark / Reset

`mark()` 负责记录当前 `position` 的位置，`reset()` 负责将当前 `position` 的位置回退到 `mark` 的位置。

```java
/**
 * Sets this buffer's mark at its position.
 *
 * @return  This buffer
 */
public final Buffer mark() {
    mark = position;
    return this;
}

/**
 * Resets this buffer's position to the previously-marked position.
 *
 * <p> Invoking this method neither changes nor discards the mark's
 * value. </p>
 *
 * @return  This buffer
 *
 * @throws  InvalidMarkException
 *          If the mark has not been set
 */
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```

## Clear

恢复状态变量到初始可写状态：

* `position` 恢复到 `0` (从头开始写)
* `limit` 恢复到 `capacity` (整个空间可写)
* `mark` 恢复到未定义状态

```java
/**
 * Clears this buffer.  The position is set to zero, the limit is set to
 * the capacity, and the mark is discarded.
 *
 * <p> Invoke this method before using a sequence of channel-read or
 * <i>put</i> operations to fill this buffer.  For example:
 *
 * <blockquote><pre>
 * buf.clear();     // Prepare buffer for reading
 * in.read(buf);    // Read data</pre></blockquote>
 *
 * <p> This method does not actually erase the data in the buffer, but it
 * is named as if it did because it will most often be used in situations
 * in which that might as well be the case. </p>
 *
 * @return  This buffer
 */
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```

## Flip

将 buffer 从可写状态转变为可读状态：

* `position` 被重置到 `0`
* `limit` 被设置到原 `position` 的位置
* `mark` 被丢弃

```java
/**
 * Flips this buffer.  The limit is set to the current position and then
 * the position is set to zero.  If the mark is defined then it is
 * discarded.
 *
 * <p> After a sequence of channel-read or <i>put</i> operations, invoke
 * this method to prepare for a sequence of channel-write or relative
 * <i>get</i> operations.  For example:
 *
 * <blockquote><pre>
 * buf.put(magic);    // Prepend header
 * in.read(buf);      // Read data into rest of buffer
 * buf.flip();        // Flip buffer
 * out.write(buf);    // Write header + data to channel</pre></blockquote>
 *
 * <p> This method is often used in conjunction with the {@link
 * java.nio.ByteBuffer#compact compact} method when transferring data from
 * one place to another.  </p>
 *
 * @return  This buffer
 */
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

## Rewind

将 `position` 置为 `0`，并丢弃 `mark`。

```java
/**
 * Rewinds this buffer.  The position is set to zero and the mark is
 * discarded.
 *
 * <p> Invoke this method before a sequence of channel-write or <i>get</i>
 * operations, assuming that the limit has already been set
 * appropriately.  For example:
 *
 * <blockquote><pre>
 * out.write(buf);    // Write remaining data
 * buf.rewind();      // Rewind buffer
 * buf.get(array);    // Copy data into array</pre></blockquote>
 *
 * @return  This buffer
 */
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

## Remaining

查看数组中的有效字节数 (`position` 与 `limit` 之间)。

```java
/**
 * Returns the number of elements between the current position and the
 * limit.
 *
 * @return  The number of elements remaining in this buffer
 */
public final int remaining() {
    return limit - position;
}

/**
 * Tells whether there are any elements between the current position and
 * the limit.
 *
 * @return  <tt>true</tt> if, and only if, there is at least one element
 *          remaining in this buffer
 */
public final boolean hasRemaining() {
    return position < limit;
}
```

## Others

底层数组是否只读、可访问：

```java
/**
 * Tells whether or not this buffer is read-only.
 *
 * @return  <tt>true</tt> if, and only if, this buffer is read-only
 */
public abstract boolean isReadOnly();

/**
 * Tells whether or not this buffer is backed by an accessible
 * array.
 *
 * <p> If this method returns <tt>true</tt> then the {@link #array() array}
 * and {@link #arrayOffset() arrayOffset} methods may safely be invoked.
 * </p>
 *
 * @return  <tt>true</tt> if, and only if, this buffer
 *          is backed by an array and is not read-only
 *
 * @since 1.6
 */
public abstract boolean hasArray();

/**
 * Returns the array that backs this
 * buffer&nbsp;&nbsp;<i>(optional operation)</i>.
 *
 * <p> This method is intended to allow array-backed buffers to be
 * passed to native code more efficiently. Concrete subclasses
 * provide more strongly-typed return values for this method.
 *
 * <p> Modifications to this buffer's content will cause the returned
 * array's content to be modified, and vice versa.
 *
 * <p> Invoke the {@link #hasArray hasArray} method before invoking this
 * method in order to ensure that this buffer has an accessible backing
 * array.  </p>
 *
 * @return  The array that backs this buffer
 *
 * @throws  ReadOnlyBufferException
 *          If this buffer is backed by an array but is read-only
 *
 * @throws  UnsupportedOperationException
 *          If this buffer is not backed by an accessible array
 *
 * @since 1.6
 */
public abstract Object array();

/**
 * Returns the offset within this buffer's backing array of the first
 * element of the buffer&nbsp;&nbsp;<i>(optional operation)</i>.
 *
 * <p> If this buffer is backed by an array then buffer position <i>p</i>
 * corresponds to array index <i>p</i>&nbsp;+&nbsp;<tt>arrayOffset()</tt>.
 *
 * <p> Invoke the {@link #hasArray hasArray} method before invoking this
 * method in order to ensure that this buffer has an accessible backing
 * array.  </p>
 *
 * @return  The offset within this buffer's array
 *          of the first element of the buffer
 *
 * @throws  ReadOnlyBufferException
 *          If this buffer is backed by an array but is read-only
 *
 * @throws  UnsupportedOperationException
 *          If this buffer is not backed by an accessible array
 *
 * @since 1.6
 */
public abstract int arrayOffset();

/**
 * Tells whether or not this buffer is
 * <a href="ByteBuffer.html#direct"><i>direct</i></a>.
 *
 * @return  <tt>true</tt> if, and only if, this buffer is direct
 *
 * @since 1.6
 */
public abstract boolean isDirect();
```

---

