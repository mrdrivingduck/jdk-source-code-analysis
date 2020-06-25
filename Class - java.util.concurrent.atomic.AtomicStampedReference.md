# Class - java.util.concurrent.atomic.AtomicStampedReference

Created by : Mr Dk.

2020 / 06 / 25 11:39

Nanjing, Jiangsu, China

---

## Definition

这个类将一个 `int` 类型的 stamp 与一个对象引用一起维护，两者将被共同地原子地更新。Stamp 的作用相当于一个版本号，每当对象引用被修改时，同时更新这个 stamp。有了 stamp，就可以解决 ABA 问题。

```java
/**
 * An {@code AtomicStampedReference} maintains an object reference
 * along with an integer "stamp", that can be updated atomically.
 *
 * <p>Implementation note: This implementation maintains stamped
 * references by creating internal objects representing "boxed"
 * [reference, integer] pairs.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <V> The type of object referred to by this reference
 */
public class AtomicStampedReference<V> {

}
```

---

## Internal State

内部维护的状态是一个 `volatile` 修饰的 pair，而 pair 中包含了对象引用与 stamp：

```java
private static class Pair<T> {
    final T reference;
    final int stamp;
    private Pair(T reference, int stamp) {
        this.reference = reference;
        this.stamp = stamp;
    }
    static <T> Pair<T> of(T reference, int stamp) {
        return new Pair<T>(reference, stamp);
    }
}

private volatile Pair<V> pair;
```

---

## Constructor

在构造对象时，传入对象引用和初始版本号：

```java
/**
 * Creates a new {@code AtomicStampedReference} with the given
 * initial values.
 *
 * @param initialRef the initial reference
 * @param initialStamp the initial stamp
 */
public AtomicStampedReference(V initialRef, int initialStamp) {
    pair = Pair.of(initialRef, initialStamp);
}
```

---

## Getter

获取内部状态。可以分别获取对象引用或 stamp，也可以直接一次性获取：

```java
/**
 * Returns the current value of the reference.
 *
 * @return the current value of the reference
 */
public V getReference() {
    return pair.reference;
}

/**
 * Returns the current value of the stamp.
 *
 * @return the current value of the stamp
 */
public int getStamp() {
    return pair.stamp;
}

/**
 * Returns the current values of both the reference and the stamp.
 * Typical usage is {@code int[1] holder; ref = v.get(holder); }.
 *
 * @param stampHolder an array of size of at least one.  On return,
 * {@code stampholder[0]} will hold the value of the stamp.
 * @return the current value of the reference
 */
public V get(int[] stampHolder) {
    Pair<V> pair = this.pair;
    stampHolder[0] = pair.stamp;
    return pair.reference;
}
```

---

## Setter

无条件设置内部状态：

```java
/**
 * Unconditionally sets the value of both the reference and stamp.
 *
 * @param newReference the new value for the reference
 * @param newStamp the new value for the stamp
 */
public void set(V newReference, int newStamp) {
    Pair<V> current = pair;
    if (newReference != current.reference || newStamp != current.stamp)
        this.pair = Pair.of(newReference, newStamp);
}
```

---

## CAS Implementation

首先通过 `Unsafe` 类获得 pair 数据的地址偏移，然后对该数据整体进行 CAS：

```java
// Unsafe mechanics

private static final sun.misc.Unsafe UNSAFE = sun.misc.Unsafe.getUnsafe();
private static final long pairOffset =
    objectFieldOffset(UNSAFE, "pair", AtomicStampedReference.class);

private boolean casPair(Pair<V> cmp, Pair<V> val) {
    return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
}

static long objectFieldOffset(sun.misc.Unsafe UNSAFE,
                                String field, Class<?> klazz) {
    try {
        return UNSAFE.objectFieldOffset(klazz.getDeclaredField(field));
    } catch (NoSuchFieldException e) {
        // Convert Exception to corresponding Error
        NoSuchFieldError error = new NoSuchFieldError(field);
        error.initCause(e);
        throw error;
    }
}
```

---

## CAS

当且仅当对象引用和 stamp 同时与期望值相等 (或没有发生改变) 时，CAS 才会成功：

```java
/**
 * Atomically sets the value of both the reference and stamp
 * to the given update values if the
 * current reference is {@code ==} to the expected reference
 * and the current stamp is equal to the expected stamp.
 *
 * <p><a href="package-summary.html#weakCompareAndSet">May fail
 * spuriously and does not provide ordering guarantees</a>, so is
 * only rarely an appropriate alternative to {@code compareAndSet}.
 *
 * @param expectedReference the expected value of the reference
 * @param newReference the new value for the reference
 * @param expectedStamp the expected value of the stamp
 * @param newStamp the new value for the stamp
 * @return {@code true} if successful
 */
public boolean weakCompareAndSet(V   expectedReference,
                                    V   newReference,
                                    int expectedStamp,
                                    int newStamp) {
    return compareAndSet(expectedReference, newReference,
                            expectedStamp, newStamp);
}

/**
 * Atomically sets the value of both the reference and stamp
 * to the given update values if the
 * current reference is {@code ==} to the expected reference
 * and the current stamp is equal to the expected stamp.
 *
 * @param expectedReference the expected value of the reference
 * @param newReference the new value for the reference
 * @param expectedStamp the expected value of the stamp
 * @param newStamp the new value for the stamp
 * @return {@code true} if successful
 */
public boolean compareAndSet(V   expectedReference,
                                V   newReference,
                                int expectedStamp,
                                int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
            newStamp == current.stamp) ||
            casPair(current, Pair.of(newReference, newStamp)));
}
```

以下函数试图通过 CAS 原子地设置 pair 中的版本号。当然前提是期望的对象引用需要与 pair 中的对象引用一致，不然说明其它线程修改了对象引用，再设置版本号是不安全的。对象引用一致后，试图将对象引用与新的版本号通过 CAS 更新到内部状态中：

```java
/**
 * Atomically sets the value of the stamp to the given update value
 * if the current reference is {@code ==} to the expected
 * reference.  Any given invocation of this operation may fail
 * (return {@code false}) spuriously, but repeated invocation
 * when the current value holds the expected value and no other
 * thread is also attempting to set the value will eventually
 * succeed.
 *
 * @param expectedReference the expected value of the reference
 * @param newStamp the new value for the stamp
 * @return {@code true} if successful
 */
public boolean attemptStamp(V expectedReference, int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        (newStamp == current.stamp ||
            casPair(current, Pair.of(expectedReference, newStamp)));
}
```

---

## 顺带一提

`java.util.concurrent.atomic.AtomicMarkableReference` 的实现与 `AtomicMarkableReference` 几乎一样。区别在于，`AtomicMarkableReference` 的 pair 内部维护的不是版本号，而是一个 `boolean` 类型的变量，代表对象引用是否被修改过。显然，在实现上会更简单。

`AtomicStampedReference` 基本上可以替代 `AtomicMarkableReference`。因为后者只能记录对象引用是否被修改过，而前者还可以记录对象引用被修改了几次。

```java
/**
 * An {@code AtomicMarkableReference} maintains an object reference
 * along with a mark bit, that can be updated atomically.
 *
 * <p>Implementation note: This implementation maintains markable
 * references by creating internal objects representing "boxed"
 * [reference, boolean] pairs.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <V> The type of object referred to by this reference
 */
public class AtomicMarkableReference<V> {

    private static class Pair<T> {
        final T reference;
        final boolean mark;
        private Pair(T reference, boolean mark) {
            this.reference = reference;
            this.mark = mark;
        }
        static <T> Pair<T> of(T reference, boolean mark) {
            return new Pair<T>(reference, mark);
        }
    }

    private volatile Pair<V> pair;

    // ...
}
```

---

