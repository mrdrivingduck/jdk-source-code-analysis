# Class - java.util.concurrent.atomic.AtomicReference

Created by : Mr Dk.

2020 / 06 / 24 22:50

Nanjing, Jiangsu, China

---

通过 CAS 操作维护一个引用。

```java
/**
 * An object reference that may be updated atomically. See the {@link
 * java.util.concurrent.atomic} package specification for description
 * of the properties of atomic variables.
 * @since 1.5
 * @author Doug Lea
 * @param <V> The type of object referred to by this reference
 */
public class AtomicReference<V> implements java.io.Serializable {

}
```

---

## Internal

内部维护了用于调用 CAS 操作的 `Unsafe` 类，和对象引用的具体信息在对象内存中的偏移。这个偏移会在类加载时被计算出来并保存。另外类中维护了一个 `volatile` 修饰的对象引用，引用类型为泛型。

```java
private static final long serialVersionUID = -1848883965231344442L;

private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicReference.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile V value;
```

---

## Constructor

构造函数很直接，传入一个 `V` 对象的引用并赋值到 `value`；或直接建立一个空的对象引用 (`null`)：

```java
/**
 * Creates a new AtomicReference with the given initial value.
 *
 * @param initialValue the initial value
 */
public AtomicReference(V initialValue) {
    value = initialValue;
}

/**
 * Creates a new AtomicReference with null initial value.
 */
public AtomicReference() {
}
```

---

## Getter and Setter

由于 `value` 由 `volatile` 关键字修饰，因此以下操作会保证线程可见性：

```java
/**
 * Gets the current value.
 *
 * @return the current value
 */
public final V get() {
    return value;
}

/**
 * Sets to the given value.
 *
 * @param newValue the new value
 */
public final void set(V newValue) {
    value = newValue;
}
```

保证线程可见性需要插入内存屏障指令，损失了一定性能。如果不需要保证线程可见性，那么就可以调用以下函数：

```java
/**
 * Eventually sets to the given value.
 *
 * @param newValue the new value
 * @since 1.6
 */
public final void lazySet(V newValue) {
    unsafe.putOrderedObject(this, valueOffset, newValue);
}
```

---

## CAS

通过 CAS 操作试图将一个更新后的对象引用更新到 `value` 中。只有保证 `value` 原有的对象引用与期望中的对象引用相等，CAS 操作才能成功。成功与否通过返回值体现。

```java
/**
 * Atomically sets the value to the given updated value
 * if the current value {@code ==} the expected value.
 * @param expect the expected value
 * @param update the new value
 * @return {@code true} if successful. False return indicates that
 * the actual value was not equal to the expected value.
 */
public final boolean compareAndSet(V expect, V update) {
    return unsafe.compareAndSwapObject(this, valueOffset, expect, update);
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
public final boolean weakCompareAndSet(V expect, V update) {
    return unsafe.compareAndSwapObject(this, valueOffset, expect, update);
}
```

---

## Operation

以下操作原子地把旧值取出来，把新值放进去。与上面 CAS 的区别在于，这个操作肯定会成功，且返回值就是对象引用：

```java
/**
 * Atomically sets to the given value and returns the old value.
 *
 * @param newValue the new value
 * @return the previous value
 */
@SuppressWarnings("unchecked")
public final V getAndSet(V newValue) {
    return (V)unsafe.getAndSetObject(this, valueOffset, newValue);
}
```

下面就是类似 `i++` 和 `++i` 的抽象版本了。支持一个一元运算符，对 `value` 进行原子地更新：

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
public final V getAndUpdate(UnaryOperator<V> updateFunction) {
    V prev, next;
    do {
        prev = get();
        next = updateFunction.apply(prev);
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
public final V updateAndGet(UnaryOperator<V> updateFunction) {
    V prev, next;
    do {
        prev = get();
        next = updateFunction.apply(prev);
    } while (!compareAndSet(prev, next));
    return next;
}
```

对二元运算符的支持：

```java
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
public final V getAndAccumulate(V x,
                                BinaryOperator<V> accumulatorFunction) {
    V prev, next;
    do {
        prev = get();
        next = accumulatorFunction.apply(prev, x);
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
public final V accumulateAndGet(V x,
                                BinaryOperator<V> accumulatorFunction) {
    V prev, next;
    do {
        prev = get();
        next = accumulatorFunction.apply(prev, x);
    } while (!compareAndSet(prev, next));
    return next;
}
```

---

## Serialization

将内部维护的 `V` 对象序列化为字符串：

```java
/**
 * Returns the String representation of the current value.
 * @return the String representation of the current value
 */
public String toString() {
    return String.valueOf(get());
}
```

---

