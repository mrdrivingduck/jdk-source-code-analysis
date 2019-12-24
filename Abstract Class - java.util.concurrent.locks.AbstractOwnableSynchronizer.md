# Abstract Class - java.util.concurrent.locks.AbstractOwnableSynchronizer

Created by : Mr Dk.

2019 / 12 / 24 15:49

Nanjing, Jiangsu, China

---

## Definition

```java
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {
    
}
```

被线程互斥持有的 __同步器__

这个类中提供了创建锁以及锁持有者的信息的基类

由其子类维护这些信息，用于控制并监视锁的使用

```java
/**
 * A synchronizer that may be exclusively owned by a thread.  This
 * class provides a basis for creating locks and related synchronizers
 * that may entail a notion of ownership.  The
 * {@code AbstractOwnableSynchronizer} class itself does not manage or
 * use this information. However, subclasses and tools may use
 * appropriately maintained values to help control and monitor access
 * and provide diagnostics.
 *
 * @since 1.6
 * @author Doug Lea
 */
```

---

```java
/** Use serial ID even though all fields transient. */
private static final long serialVersionUID = 3737899427754241961L;

/**
 * Empty constructor for use by subclasses.
 */
protected AbstractOwnableSynchronizer() { }
```

---

```java
/**
 * The current owner of exclusive mode synchronization.
 */
private transient Thread exclusiveOwnerThread;
```

维护当前具有互斥访问资格的线程

并由以下两个函数修改该变量

```java
/**
 * Sets the thread that currently owns exclusive access.
 * A {@code null} argument indicates that no thread owns access.
 * This method does not otherwise impose any synchronization or
 * {@code volatile} field accesses.
 * @param thread the owner thread
 */
protected final void setExclusiveOwnerThread(Thread thread) {
    exclusiveOwnerThread = thread;
}

/**
 * Returns the thread last set by {@code setExclusiveOwnerThread},
 * or {@code null} if never set.  This method does not otherwise
 * impose any synchronization or {@code volatile} field accesses.
 * @return the owner thread
 */
protected final Thread getExclusiveOwnerThread() {
    return exclusiveOwnerThread;
}
```

---

