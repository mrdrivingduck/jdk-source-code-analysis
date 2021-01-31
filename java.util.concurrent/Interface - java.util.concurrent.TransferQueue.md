# Interface - java.util.concurrent.TransferQueue

Created by : Mr Dk.

2020 / 10 / 22 13:44

Nanjing, Jiangsu, China

---

## Definition

继承自 BlockingQueue。BlockingQueue 的出队和入队需要上锁后进行操作，而 TransferQueue 会额外维护正在等待资源的线程，当资源可用时，资源将不需要入队而是直接被转移给等待资源的线程。

```java
/**
 * A {@link BlockingQueue} in which producers may wait for consumers
 * to receive elements.  A {@code TransferQueue} may be useful for
 * example in message passing applications in which producers
 * sometimes (using method {@link #transfer}) await receipt of
 * elements by consumers invoking {@code take} or {@code poll}, while
 * at other times enqueue elements (via method {@code put}) without
 * waiting for receipt.
 * {@linkplain #tryTransfer(Object) Non-blocking} and
 * {@linkplain #tryTransfer(Object,long,TimeUnit) time-out} versions of
 * {@code tryTransfer} are also available.
 * A {@code TransferQueue} may also be queried, via {@link
 * #hasWaitingConsumer}, whether there are any threads waiting for
 * items, which is a converse analogy to a {@code peek} operation.
 *
 * <p>Like other blocking queues, a {@code TransferQueue} may be
 * capacity bounded.  If so, an attempted transfer operation may
 * initially block waiting for available space, and/or subsequently
 * block waiting for reception by a consumer.  Note that in a queue
 * with zero capacity, such as {@link SynchronousQueue}, {@code put}
 * and {@code transfer} are effectively synonymous.
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.7
 * @author Doug Lea
 * @param <E> the type of elements held in this collection
 */
public interface TransferQueue<E> extends BlockingQueue<E> {

}
```

## Transfer

非阻塞地尝试将元素直接转移给一个正在等待中的消费者。如果没有等待中的消费者，那么不进行入队操作，直接返回 `false`。

```java
/**
 * Transfers the element to a waiting consumer immediately, if possible.
 *
 * <p>More precisely, transfers the specified element immediately
 * if there exists a consumer already waiting to receive it (in
 * {@link #take} or timed {@link #poll(long,TimeUnit) poll}),
 * otherwise returning {@code false} without enqueuing the element.
 *
 * @param e the element to transfer
 * @return {@code true} if the element was transferred, else
 *         {@code false}
 * @throws ClassCastException if the class of the specified element
 *         prevents it from being added to this queue
 * @throws NullPointerException if the specified element is null
 * @throws IllegalArgumentException if some property of the specified
 *         element prevents it from being added to this queue
 */
boolean tryTransfer(E e);
```

阻塞定时版本，在超时后如果还没有被消费则返回 `false`。

```java
/**
 * Transfers the element to a consumer if it is possible to do so
 * before the timeout elapses.
 *
 * <p>More precisely, transfers the specified element immediately
 * if there exists a consumer already waiting to receive it (in
 * {@link #take} or timed {@link #poll(long,TimeUnit) poll}),
 * else waits until the element is received by a consumer,
 * returning {@code false} if the specified wait time elapses
 * before the element can be transferred.
 *
 * @param e the element to transfer
 * @param timeout how long to wait before giving up, in units of
 *        {@code unit}
 * @param unit a {@code TimeUnit} determining how to interpret the
 *        {@code timeout} parameter
 * @return {@code true} if successful, or {@code false} if
 *         the specified waiting time elapses before completion,
 *         in which case the element is not left enqueued
 * @throws InterruptedException if interrupted while waiting,
 *         in which case the element is not left enqueued
 * @throws ClassCastException if the class of the specified element
 *         prevents it from being added to this queue
 * @throws NullPointerException if the specified element is null
 * @throws IllegalArgumentException if some property of the specified
 *         element prevents it from being added to this queue
 */
boolean tryTransfer(E e, long timeout, TimeUnit unit)
    throws InterruptedException;
```

阻塞版本。将一直阻塞直到被消费者消费。

```java
/**
 * Transfers the element to a consumer, waiting if necessary to do so.
 *
 * <p>More precisely, transfers the specified element immediately
 * if there exists a consumer already waiting to receive it (in
 * {@link #take} or timed {@link #poll(long,TimeUnit) poll}),
 * else waits until the element is received by a consumer.
 *
 * @param e the element to transfer
 * @throws InterruptedException if interrupted while waiting,
 *         in which case the element is not left enqueued
 * @throws ClassCastException if the class of the specified element
 *         prevents it from being added to this queue
 * @throws NullPointerException if the specified element is null
 * @throws IllegalArgumentException if some property of the specified
 *         element prevents it from being added to this queue
 */
void transfer(E e) throws InterruptedException;
```

## Consumer

返回是否有正在等待的消费者：

```java
/**
 * Returns {@code true} if there is at least one consumer waiting
 * to receive an element via {@link #take} or
 * timed {@link #poll(long,TimeUnit) poll}.
 * The return value represents a momentary state of affairs.
 *
 * @return {@code true} if there is at least one waiting consumer
 */
boolean hasWaitingConsumer();
```

返回正在等待的消费者个数：

```java
/**
 * Returns an estimate of the number of consumers waiting to
 * receive elements via {@link #take} or timed
 * {@link #poll(long,TimeUnit) poll}.  The return value is an
 * approximation of a momentary state of affairs, that may be
 * inaccurate if consumers have completed or given up waiting.
 * The value may be useful for monitoring and heuristics, but
 * not for synchronization control.  Implementations of this
 * method are likely to be noticeably slower than those for
 * {@link #hasWaitingConsumer}.
 *
 * @return the number of consumers waiting to receive elements
 */
int getWaitingConsumerCount();
```

---

