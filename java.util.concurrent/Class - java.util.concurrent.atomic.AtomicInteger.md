# Class - java.util.concurrent.atomic.AtomicInteger

Created by : Mr Dk.

2020 / 06 / 07 11:28

Nanjing, Jiangsu, China

---

这个类主要通过 [CAS](https://en.wikipedia.org/wiki/Compare-and-swap) 操作实现了一个可以被原子操作的 Integer 计数器。CAS 操作避免使用 OS 层面的锁，使得同步问题可以在用户空间解决。该类中绝大部分功能都是由 JVM 的 `Unsafe` 类提供，通过 JVM 的 C++ 代码调用到最终的 CAS 硬件指令 (如 `CMPXCHG`)。

这个类本身的实现很简单，大部分都是对 `Unsafe` 类函数的调用。重要的是要从 CPU 的层面上去理解为什么 `Unsafe` 可以原子地完成 CAS 或加法等运算操作。

```java
/**
 * An {@code int} value that may be updated atomically.  See the
 * {@link java.util.concurrent.atomic} package specification for
 * description of the properties of atomic variables. An
 * {@code AtomicInteger} is used in applications such as atomically
 * incremented counters, and cannot be used as a replacement for an
 * {@link java.lang.Integer}. However, this class does extend
 * {@code Number} to allow uniform access by tools and utilities that
 * deal with numerically-based classes.
 *
 * @since 1.5
 * @author Doug Lea
*/
public class AtomicInteger extends Number implements java.io.Serializable {

}
```

---

该类中维护的静态变量：

* `unsafe` 是得到了 `Unsafe` 类的实例，方便调用其函数
* `valueOffset` 中记录了要维护的值 `value` 在对象内存中的偏移 (应该是用于构成指向 `value` 的指针)
* `value` 用于表示被原子地维护的具体数值，被 `volatile` 关键字修饰，因此：
    * 线程可见性
    * 禁止指令重排序

> 写 `volatile` 变量到缓存时会被强制刷新到内存；读 `volatile` 变量时缓存将失效，强制从内存中读取最新的值。

```java
private static final long serialVersionUID = 6214790243416807050L;

// setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

---

构造函数：

```java
/**
 * Creates a new AtomicInteger with the given initial value.
 *
 * @param initialValue the initial value
 */
public AtomicInteger(int initialValue) {
    value = initialValue;
}

/**
 * Creates a new AtomicInteger with initial value {@code 0}.
 */
public AtomicInteger() {
}
```

---

普通的 get/set 函数。这两个函数应该只有在单线程场景下不会出错：

```java
/**
 * Gets the current value.
 *
 * @return the current value
 */
public final int get() {
    return value;
}

/**
 * Sets to the given value.
 *
 * @param newValue the new value
 */
public final void set(int newValue) {
    value = newValue;
}
```

---

由于 `value` 的值被 `volatile` 修饰，底层由内存屏障指令来实现线程可见性。在一些确定不需要线程可见性的场景中，可以通过以下函数将 `value` 当做一个普通变量进行赋值，从而省去内存屏障：

```java
/**
 * Eventually sets to the given value.
 *
 * @param newValue the new value
 * @since 1.6
 */
public final void lazySet(int newValue) {
    unsafe.putOrderedInt(this, valueOffset, newValue);
}
```

如果想要原子地修改 `value` 值，那么就需要保证线程可见性了：

```java
/**
 * Atomically sets to the given value and returns the old value.
 *
 * @param newValue the new value
 * @return the previous value
 */
public final int getAndSet(int newValue) {
    return unsafe.getAndSetInt(this, valueOffset, newValue);
}
```

---

以下函数是 CAS 的具体实现。给定一个预期值和一个更新后的值，只有当前值与预期值相等时，才会将新值赋值给变量：

```java
/**
 * Atomically sets the value to the given updated value
 * if the current value {@code ==} the expected value.
 *
 * @param expect the expected value
 * @param update the new value
 * @return {@code true} if successful. False return indicates that
 * the actual value was not equal to the expected value.
 */
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

/**
 * Atomically sets the value to the given updated value
 * if the current value {@code ==} the expected value.
 *
 * <p><a href="package-summary.html#weakCompareAndSet">May fail
 * spuriously and does not provide ordering guarantees</a>, so is
 * only rarely an appropriate alternative to {@code compareAndSet}.
 *
 * @param expect the expected value
 * @param update the new value
 * @return {@code true} if successful
 */
public final boolean weakCompareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

---

以下是对 `value` 原子地进行增加或减少的函数，对应于 `i++` 的行为，先获取变量的值，再对变量更新。

```java
/**
 * Atomically increments by one the current value.
 *
 * @return the previous value
 */
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

/**
 * Atomically decrements by one the current value.
 *
 * @return the previous value
 */
public final int getAndDecrement() {
    return unsafe.getAndAddInt(this, valueOffset, -1);
}

/**
 * Atomically adds the given value to the current value.
 *
 * @param delta the value to add
 * @return the previous value
 */
public final int getAndAdd(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta);
}
```

而以下的函数则对应 `++i` 的行为，先对 `value` 进行修改，再获取变量的值：

```java
/**
 * Atomically increments by one the current value.
 *
 * @return the updated value
 */
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

/**
 * Atomically decrements by one the current value.
 *
 * @return the updated value
 */
public final int decrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, -1) - 1;
}

/**
 * Atomically adds the given value to the current value.
 *
 * @param delta the value to add
 * @return the updated value
 */
public final int addAndGet(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
}
```

---

上述操作只实现了 add 和 get 不同顺序的操作。如果想要支持其它的运算操作 (比如乘法) 呢？对加法操作，CPU 通常提供指令可以实现原子操作，而其它运算就不一定了。因此只能通过几条普通指令完成运算，然后再通过 CAS 操作原子地更新 `value`。

针对 get 操作和运算操作的顺序，以及运算操作的类型 (一元运算 / 二元运算)，提供了以下四个函数：

```java
/**
 * Atomically updates the current value with the results of
 * applying the given function, returning the previous value. The
 * function should be side-effect-free, since it may be re-applied
 * when attempted updates fail due to contention among threads.
 *
 * @param updateFunction a side-effect-free function
 * @return the previous value
 * @since 1.8
 */
public final int getAndUpdate(IntUnaryOperator updateFunction) {
    int prev, next;
    do {
        prev = get();
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSet(prev, next));
    return prev;
}

/**
 * Atomically updates the current value with the results of
 * applying the given function, returning the updated value. The
 * function should be side-effect-free, since it may be re-applied
 * when attempted updates fail due to contention among threads.
 *
 * @param updateFunction a side-effect-free function
 * @return the updated value
 * @since 1.8
 */
public final int updateAndGet(IntUnaryOperator updateFunction) {
    int prev, next;
    do {
        prev = get();
        next = updateFunction.applyAsInt(prev);
    } while (!compareAndSet(prev, next));
    return next;
}

/**
 * Atomically updates the current value with the results of
 * applying the given function to the current and given values,
 * returning the previous value. The function should be
 * side-effect-free, since it may be re-applied when attempted
 * updates fail due to contention among threads.  The function
 * is applied with the current value as its first argument,
 * and the given update as the second argument.
 *
 * @param x the update value
 * @param accumulatorFunction a side-effect-free function of two arguments
 * @return the previous value
 * @since 1.8
 */
public final int getAndAccumulate(int x,
                                    IntBinaryOperator accumulatorFunction) {
    int prev, next;
    do {
        prev = get();
        next = accumulatorFunction.applyAsInt(prev, x);
    } while (!compareAndSet(prev, next));
    return prev;
}

/**
 * Atomically updates the current value with the results of
 * applying the given function to the current and given values,
 * returning the updated value. The function should be
 * side-effect-free, since it may be re-applied when attempted
 * updates fail due to contention among threads.  The function
 * is applied with the current value as its first argument,
 * and the given update as the second argument.
 *
 * @param x the update value
 * @param accumulatorFunction a side-effect-free function of two arguments
 * @return the updated value
 * @since 1.8
 */
public final int accumulateAndGet(int x,
                                    IntBinaryOperator accumulatorFunction) {
    int prev, next;
    do {
        prev = get();
        next = accumulatorFunction.applyAsInt(prev, x);
    } while (!compareAndSet(prev, next));
    return next;
}
```

---

剩下的是一些比较简单的类型转换函数：

```java
/**
 * Returns the String representation of the current value.
 * @return the String representation of the current value
 */
public String toString() {
    return Integer.toString(get());
}

/**
 * Returns the value of this {@code AtomicInteger} as an {@code int}.
 */
public int intValue() {
    return get();
}

/**
 * Returns the value of this {@code AtomicInteger} as a {@code long}
 * after a widening primitive conversion.
 * @jls 5.1.2 Widening Primitive Conversions
 */
public long longValue() {
    return (long)get();
}

/**
 * Returns the value of this {@code AtomicInteger} as a {@code float}
 * after a widening primitive conversion.
 * @jls 5.1.2 Widening Primitive Conversions
 */
public float floatValue() {
    return (float)get();
}

/**
 * Returns the value of this {@code AtomicInteger} as a {@code double}
 * after a widening primitive conversion.
 * @jls 5.1.2 Widening Primitive Conversions
 */
public double doubleValue() {
    return (double)get();
}
```

---

## References

[CSDN - 理解 AtomicXXX.lazySet 方法](https://blog.csdn.net/ITer_ZC/article/details/40744485)

---

