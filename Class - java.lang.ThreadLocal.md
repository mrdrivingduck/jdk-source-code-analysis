# Class - java.lang.ThreadLocal

Created by : Mr Dk.

2020 / 11 / 10 15:15

Nanjing, Jiangsu, China

---

## Definition

这个类提供了线程局部变量，每个线程能够通过 `get()` 或 `set()` 访问其内部独立的变量。因此，这个类可以用于保存私有、静态的与线程相关的状态变量。每个线程持有对变量的隐式引用，在线程结束后，所有引用的线程局部变量将会被主动 GC，除非这些变量还被额外引用。

> 该类存在的意义。比如线程会从数据库连接池获取连接，并执行事务。对一个线程来说，如果每次取得的数据库连接不是同一个，将无法执行事务。通过 `ThreadLocal` 对象，线程可以把同一个数据库连接保存到线程局部变量中，这样可以保证每次取得的是同一个数据库连接。

```java
/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 *
 * <p>For example, the class below generates unique identifiers local to each
 * thread.
 * A thread's id is assigned the first time it invokes {@code ThreadId.get()}
 * and remains unchanged on subsequent calls.
 * <pre>
 * import java.util.concurrent.atomic.AtomicInteger;
 *
 * public class ThreadId {
 *     // Atomic integer containing the next thread ID to be assigned
 *     private static final AtomicInteger nextId = new AtomicInteger(0);
 *
 *     // Thread local variable containing each thread's ID
 *     private static final ThreadLocal&lt;Integer&gt; threadId =
 *         new ThreadLocal&lt;Integer&gt;() {
 *             &#64;Override protected Integer initialValue() {
 *                 return nextId.getAndIncrement();
 *         }
 *     };
 *
 *     // Returns the current thread's unique ID, assigning it if necessary
 *     public static int get() {
 *         return threadId.get();
 *     }
 * }
 * </pre>
 * <p>Each thread holds an implicit reference to its copy of a thread-local
 * variable as long as the thread is alive and the {@code ThreadLocal}
 * instance is accessible; after a thread goes away, all of its copies of
 * thread-local instances are subject to garbage collection (unless other
 * references to these copies exist).
 *
 * @author  Josh Bloch and Doug Lea
 * @since   1.2
 */
public class ThreadLocal<T> {

}
```

---

## Fields

在每一个线程对象 `Thread` 中，维护着一个 `ThreadLocalMap`，即与每个线程关联的线性探测 hash map - 其中，`ThreadLocal` 对象是这个 hash map 的 key，以 `threadLocalHashCode` 作为 hash 值。在大部分场合中，由于 `ThreadLocal` 对象会被连续创建，这个 hash code 用于消除碰撞；但同时在较为少见的场合中也表现良好。

```java
/**
 * ThreadLocals rely on per-thread linear-probe hash maps attached
 * to each thread (Thread.threadLocals and
 * inheritableThreadLocals).  The ThreadLocal objects act as keys,
 * searched via threadLocalHashCode.  This is a custom hash code
 * (useful only within ThreadLocalMaps) that eliminates collisions
 * in the common case where consecutively constructed ThreadLocals
 * are used by the same threads, while remaining well-behaved in
 * less common cases.
 */
private final int threadLocalHashCode = nextHashCode();

/**
 * The next hash code to be given out. Updated atomically. Starts at
 * zero.
 */
private static AtomicInteger nextHashCode =
    new AtomicInteger();

/**
 * The difference between successively generated hash codes - turns
 * implicit sequential thread-local IDs into near-optimally spread
 * multiplicative hash values for power-of-two-sized tables.
 */
private static final int HASH_INCREMENT = 0x61c88647;

/**
 * Returns the next hash code.
 */
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

## Set

将一个特定的值赋值到当前线程的局部变量中。首先取得当前的线程对象，然后取得线程对象中的 `ThreadLocalMap`。如果不为空，就以当前 `ThreadLocal` 对象作为 key 设置到 map 中；如果为空，那么就先建立 map 再赋值。

```java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

通过线程对象获得 map 的函数：

```java
/**
 * Get the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param  t the current thread
 * @return the map
 */
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

Map 本身是懒创建的，只有第一次调用 `set()` 时才会创建：

```java
/**
 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

## Get

从当前线程的 `ThreadLocalMap` 中取得局部变量。通过当前线程的线程对象，取得 `ThreadLocalMap`。如果 map 不为空，则以当前 `ThreadLocal` 对象为 key 取出相应的值；如果 map 为空，那么先创建一个 map，然后在 map 中设置默认的初始值 (`null`)。

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

```java
/**
 * Variant of set() to establish initialValue. Used instead
 * of set() in case user has overridden the set() method.
 *
 * @return the initial value
 */
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

```java
/**
 * Returns the current thread's "initial value" for this
 * thread-local variable.  This method will be invoked the first
 * time a thread accesses the variable with the {@link #get}
 * method, unless the thread previously invoked the {@link #set}
 * method, in which case the {@code initialValue} method will not
 * be invoked for the thread.  Normally, this method is invoked at
 * most once per thread, but it may be invoked again in case of
 * subsequent invocations of {@link #remove} followed by {@link #get}.
 *
 * <p>This implementation simply returns {@code null}; if the
 * programmer desires thread-local variables to have an initial
 * value other than {@code null}, {@code ThreadLocal} must be
 * subclassed, and this method overridden.  Typically, an
 * anonymous inner class will be used.
 *
 * @return the initial value for this thread-local
 */
protected T initialValue() {
    return null;
}
```

## Remove

通过当前线程的 `Thread` 对象取得 `ThreadLocalMap`，然后以当前 `ThreadLocal` 对象为 key 移除 map 中的值：

```java
/**
 * Removes the current thread's value for this thread-local
 * variable.  If this thread-local variable is subsequently
 * {@linkplain #get read} by the current thread, its value will be
 * reinitialized by invoking its {@link #initialValue} method,
 * unless its value is {@linkplain #set set} by the current thread
 * in the interim.  This may result in multiple invocations of the
 * {@code initialValue} method in the current thread.
 *
 * @since 1.5
 */
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

---

## Thread Local Map

这个 map 被定义在 `ThreadLocal` 类内，不会被外部类使用。

其中有一个显著的特点是：该 map 的 key 是 **弱引用** 类型的。也就是说，当程序中没有其它 **强引用** 指向堆上的 `ThreadLocal` 对象时，`ThreadLocalMap` 中的 `ThreadLocal` 引用将会被 GC。如果这里是强引用，那么 `ThreadLocal` 对象将会一直随线程存在 (可能存在内存泄漏，假设这是一个长期运行的后台线程)，哪怕程序中已经不存在任何对 `ThreadLocal` 对象的引用。

注意，就算 map 内弱引用指向的 `ThreadLocal` 对象已经被 GC，相应的 value 实际上并没有被 GC (因为 map 还持有对 value 对象的引用)，而此时已经没有引用可以访问到这个 value 了，久而久之，这些无法被引用的 value 会产生内存泄漏。因此，如果 `ThreadLocal` 对象不用了，需要调用 `ThreadLocal` 的 `remove()` 函数将 value 的引用从 map 中移除，这样 value 对应的内存也会被 GC。

```java
/**
 * ThreadLocalMap is a customized hash map suitable only for
 * maintaining thread local values. No operations are exported
 * outside of the ThreadLocal class. The class is package private to
 * allow declaration of fields in class Thread.  To help deal with
 * very large and long-lived usages, the hash table entries use
 * WeakReferences for keys. However, since reference queues are not
 * used, stale entries are guaranteed to be removed only when
 * the table starts running out of space.
 */
static class ThreadLocalMap {

    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;

    /**
     * The table, resized as necessary.
     * table.length MUST always be a power of two.
     */
    private Entry[] table;

    // ...
}
```

---

