# Class - java.util.concurrent.atomic.AtomicIntegerArray

Created by : Mr Dk.

2020 / 06 / 08 16:16

Nanjing, Jiangsu, China

---

## Definition

这个类内部维护了一个可以被原子操作的 `int[]` 变量。

> 与 `AtomicInteger[]` 有啥区别呢？直观上来看，`AtomicIntegerArray` 本身是被同步的 **一个** 对象，而 `AtomicInteger[]` 中的每个元素都是一个被同步的对象，数组对象本身不被同步。至于性能上的对比，可能是由数组中的各个元素是否位于同一 cache line 决定的。

```java
/**
 * An {@code int} array in which elements may be updated atomically.
 * See the {@link java.util.concurrent.atomic} package
 * specification for description of the properties of atomic
 * variables.
 * @since 1.5
 * @author Doug Lea
 */
public class AtomicIntegerArray implements java.io.Serializable {

}
```

---

内部维护的状态变量：

* `Unsafe` 类的引用，用于调用其中封装的函数
* `base` 保存了 `int[]` 的内存在对象内存中的偏移
* `shift` 保存了为了访问数组中的每个元素需要移位的量 (4 字节，左移两位)

```java
private static final long serialVersionUID = 2862133569453604235L;

private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final int base = unsafe.arrayBaseOffset(int[].class);
private static final int shift;
private final int[] array;
```

---

如何计算上述的 `shift` 呢？

* 首先获得数组中每个元素的长度 `scale` (应该为 4)
* 然后根据 `4` 中的前导 0 (29 个)，算出偏移的位数应该为 2

在之后的运算中，通过数组的基地址 `base`，加上 index 左移 2 位 (乘以 4)，就能够获得每一个元素的内存地址。

```java
static {
    int scale = unsafe.arrayIndexScale(int[].class);
    if ((scale & (scale - 1)) != 0)
        throw new Error("data type scale not a power of two");
    shift = 31 - Integer.numberOfLeadingZeros(scale);
}

private long checkedByteOffset(int i) {
    if (i < 0 || i >= array.length)
        throw new IndexOutOfBoundsException("index " + i);

    return byteOffset(i);
}

private static long byteOffset(int i) {
    return ((long) i << shift) + base;
}
```

---

构造函数：

* 可以指定数组的长度，分配新的数组
* 也可以指定一个已有数组，对其进行克隆

```java
/**
 * Creates a new AtomicIntegerArray of the given length, with all
 * elements initially zero.
 *
 * @param length the length of the array
 */
public AtomicIntegerArray(int length) {
    array = new int[length];
}

/**
 * Creates a new AtomicIntegerArray with the same length as, and
 * all elements copied from, the given array.
 *
 * @param array the array to copy elements from
 * @throws NullPointerException if array is null
 */
public AtomicIntegerArray(int[] array) {
    // Visibility guaranteed by final field guarantees
    this.array = array.clone();
}

/**
 * Returns the length of the array.
 *
 * @return the length of the array
 */
public final int length() {
    return array.length;
}
```

---

Get/set 函数：

由于 `int[]` 的声明没有使用 `volatile` 关键字，本来元素将不具备线程可见性。而这里调用 `Unsafe` 类的函数全都是带有 `volatile` 的，因此可以具备线程可见性：

```java
/**
 * Gets the current value at position {@code i}.
 *
 * @param i the index
 * @return the current value
 */
public final int get(int i) {
    return getRaw(checkedByteOffset(i));
}

private int getRaw(long offset) {
    return unsafe.getIntVolatile(array, offset);
}

/**
 * Sets the element at position {@code i} to the given value.
 *
 * @param i the index
 * @param newValue the new value
 */
public final void set(int i, int newValue) {
    unsafe.putIntVolatile(array, checkedByteOffset(i), newValue);
}
```

当然，如果确定不需要具有线程可见性，使用如下的函数将会有更好的性能：

```java
/**
 * Eventually sets the element at position {@code i} to the given value.
 *
 * @param i the index
 * @param newValue the new value
 * @since 1.6
 */
public final void lazySet(int i, int newValue) {
    unsafe.putOrderedInt(array, checkedByteOffset(i), newValue);
}
```

---

原子地将一个新值代替某个旧值，并返回旧值。以下所有函数都与 `AtomicInteger` 类似，只是需要多给一个 index 参数用于指示操作数组的第几个元素。

```java
/**
 * Atomically sets the element at position {@code i} to the given
 * value and returns the old value.
 *
 * @param i the index
 * @param newValue the new value
 * @return the previous value
 */
public final int getAndSet(int i, int newValue) {
    return unsafe.getAndSetInt(array, checkedByteOffset(i), newValue);
}
```

> 该函数与 CAS 的区别：这个函数直接将一个新值替进去，把旧的值换出来，而不去管旧值是什么；而 CAS 需要先将旧值读出来，作运算，完毕后再写回去，这一过程要保持原子性。

---

CAS 操作：

```java
/**
 * Atomically sets the element at position {@code i} to the given
 * updated value if the current value {@code ==} the expected value.
 *
 * @param i the index
 * @param expect the expected value
 * @param update the new value
 * @return {@code true} if successful. False return indicates that
 * the actual value was not equal to the expected value.
 */
public final boolean compareAndSet(int i, int expect, int update) {
    return compareAndSetRaw(checkedByteOffset(i), expect, update);
}

private boolean compareAndSetRaw(long offset, int expect, int update) {
    return unsafe.compareAndSwapInt(array, offset, expect, update);
}

/**
 * Atomically sets the element at position {@code i} to the given
 * updated value if the current value {@code ==} the expected value.
 *
 * <p><a href="package-summary.html#weakCompareAndSet">May fail
 * spuriously and does not provide ordering guarantees</a>, so is
 * only rarely an appropriate alternative to {@code compareAndSet}.
 *
 * @param i the index
 * @param expect the expected value
 * @param update the new value
 * @return {@code true} if successful
 */
public final boolean weakCompareAndSet(int i, int expect, int update) {
    return compareAndSet(i, expect, update);
}
```

---

以下函数类似 `i++` 的功能：

```java
/**
 * Atomically increments by one the element at index {@code i}.
 *
 * @param i the index
 * @return the previous value
 */
public final int getAndIncrement(int i) {
    return getAndAdd(i, 1);
}

/**
 * Atomically decrements by one the element at index {@code i}.
 *
 * @param i the index
 * @return the previous value
 */
public final int getAndDecrement(int i) {
    return getAndAdd(i, -1);
}

/**
 * Atomically adds the given value to the element at index {@code i}.
 *
 * @param i the index
 * @param delta the value to add
 * @return the previous value
 */
public final int getAndAdd(int i, int delta) {
    return unsafe.getAndAddInt(array, checkedByteOffset(i), delta);
}
```

---

以下函数类似 `++i` 功能：

```java
/**
 * Atomically increments by one the element at index {@code i}.
 *
 * @param i the index
 * @return the updated value
 */
public final int incrementAndGet(int i) {
    return getAndAdd(i, 1) + 1;
}

/**
 * Atomically decrements by one the element at index {@code i}.
 *
 * @param i the index
 * @return the updated value
 */
public final int decrementAndGet(int i) {
    return getAndAdd(i, -1) - 1;
}

/**
 * Atomically adds the given value to the element at index {@code i}.
 *
 * @param i the index
 * @param delta the value to add
 * @return the updated value
 */
public final int addAndGet(int i, int delta) {
    return getAndAdd(i, delta) + delta;
}
```

---

以下四个函数分别可以实现一元操作符和二元操作符，并返回操作前的值或操作后的值 (类似 `i++` 和 `++i`)：

```java
/**
 * Atomically updates the element at index {@code i} with the results
 * of applying the given function, returning the previous value. The
 * function should be side-effect-free, since it may be re-applied
 * when attempted updates fail due to contention among threads.
 *
 * @param i the index
 * @param updateFunction a side-effect-free function
 * @return the previous value
 * @since 1.8
 */
public final int getAndUpdate(int i, IntUnaryOperator updateFunction) {
    long offset = checkedByteOffset(i);
    int prev, next;
    do {
        prev = getRaw(offset);
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSetRaw(offset, prev, next));
    return prev;
}

/**
 * Atomically updates the element at index {@code i} with the results
 * of applying the given function, returning the updated value. The
 * function should be side-effect-free, since it may be re-applied
 * when attempted updates fail due to contention among threads.
 *
 * @param i the index
 * @param updateFunction a side-effect-free function
 * @return the updated value
 * @since 1.8
 */
public final int updateAndGet(int i, IntUnaryOperator updateFunction) {
    long offset = checkedByteOffset(i);
    int prev, next;
    do {
        prev = getRaw(offset);
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSetRaw(offset, prev, next));
    return next;
}

/**
 * Atomically updates the element at index {@code i} with the
 * results of applying the given function to the current and
 * given values, returning the previous value. The function should
 * be side-effect-free, since it may be re-applied when attempted
 * updates fail due to contention among threads.  The function is
 * applied with the current value at index {@code i} as its first
 * argument, and the given update as the second argument.
 *
 * @param i the index
 * @param x the update value
 * @param accumulatorFunction a side-effect-free function of two arguments
 * @return the previous value
 * @since 1.8
 */
public final int getAndAccumulate(int i, int x,
                                    IntBinaryOperator accumulatorFunction) {
    long offset = checkedByteOffset(i);
    int prev, next;
    do {
        prev = getRaw(offset);
        next = accumulatorFunction.applyAsInt(prev, x);
    } while (!compareAndSetRaw(offset, prev, next));
    return prev;
}

/**
 * Atomically updates the element at index {@code i} with the
 * results of applying the given function to the current and
 * given values, returning the updated value. The function should
 * be side-effect-free, since it may be re-applied when attempted
 * updates fail due to contention among threads.  The function is
 * applied with the current value at index {@code i} as its first
 * argument, and the given update as the second argument.
 *
 * @param i the index
 * @param x the update value
 * @param accumulatorFunction a side-effect-free function of two arguments
 * @return the updated value
 * @since 1.8
 */
public final int accumulateAndGet(int i, int x,
                                    IntBinaryOperator accumulatorFunction) {
    long offset = checkedByteOffset(i);
    int prev, next;
    do {
        prev = getRaw(offset);
        next = accumulatorFunction.applyAsInt(prev, x);
    } while (!compareAndSetRaw(offset, prev, next));
    return next;
}
```

---

`toString()` 函数：

```java
/**
 * Returns the String representation of the current values of array.
 * @return the String representation of the current values of array
 */
public String toString() {
    int iMax = array.length - 1;
    if (iMax == -1)
        return "[]";

    StringBuilder b = new StringBuilder();
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(getRaw(byteOffset(i)));
        if (i == iMax)
            return b.append(']').toString();
        b.append(',').append(' ');
    }
}
```

---

## References

[Stackoverflow - AtomicIntegerArray vs AtomicInteger[]](https://stackoverflow.com/questions/692677/atomicintegerarray-vs-atomicinteger)

---

