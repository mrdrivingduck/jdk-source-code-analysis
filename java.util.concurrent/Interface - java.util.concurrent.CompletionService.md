# Interface - java.util.concurrent.CompletionService

Created by : Mr Dk.

2021 / 02 / 16 🧧 19:03

Ningbo, Zhejiang, China

---

## Definition

该接口将 **提交新的异步任务** 与 **消费已完成任务产生的结果** 解耦。生产者调用 `submit()` 提交任务，消费者调用 `take()` 按照任务完成的顺序获取完成的任务并处理结果。该接口的实现类可用于管理异步 I/O：

* 读取任务在程序的某个部分被提交到该服务中
* 程序的另一部分处理读取完成后的结果

本接口的实现类内部依赖于一个独立的执行器来真正执行任务，本接口定义的函数仅用于管理内部的 **完成队列** - 队列中存放所有已完成的任务。

```java
/**
 * A service that decouples the production of new asynchronous tasks
 * from the consumption of the results of completed tasks.  Producers
 * {@code submit} tasks for execution. Consumers {@code take}
 * completed tasks and process their results in the order they
 * complete.  A {@code CompletionService} can for example be used to
 * manage asynchronous I/O, in which tasks that perform reads are
 * submitted in one part of a program or system, and then acted upon
 * in a different part of the program when the reads complete,
 * possibly in a different order than they were requested.
 *
 * <p>Typically, a {@code CompletionService} relies on a separate
 * {@link Executor} to actually execute the tasks, in which case the
 * {@code CompletionService} only manages an internal completion
 * queue. The {@link ExecutorCompletionService} class provides an
 * implementation of this approach.
 *
 * <p>Memory consistency effects: Actions in a thread prior to
 * submitting a task to a {@code CompletionService}
 * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 * actions taken by that task, which in turn <i>happen-before</i>
 * actions following a successful return from the corresponding {@code take()}.
 */
public interface CompletionService<V> {

}
```

## Submit

将任务提交给执行器执行，并返回一个 `Future` 对象。当任务完成后，将可以从完成队列中取出。

```java
/**
 * Submits a value-returning task for execution and returns a Future
 * representing the pending results of the task.  Upon completion,
 * this task may be taken or polled.
 *
 * @param task the task to submit
 * @return a Future representing pending completion of the task
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if the task is null
 */
Future<V> submit(Callable<V> task);

/**
 * Submits a Runnable task for execution and returns a Future
 * representing that task.  Upon completion, this task may be
 * taken or polled.
 *
 * @param task the task to submit
 * @param result the result to return upon successful completion
 * @return a Future representing pending completion of the task,
 *         and whose {@code get()} method will return the given
 *         result value upon completion
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if the task is null
 */
Future<V> submit(Runnable task, V result);
```

## Take

从完成队列中取得并移除下一个已经完成的任务。如果队列中暂时没有任何已完成任务，则阻塞等待。

```java
/**
 * Retrieves and removes the Future representing the next
 * completed task, waiting if none are yet present.
 *
 * @return the Future representing the next completed task
 * @throws InterruptedException if interrupted while waiting
 */
Future<V> take() throws InterruptedException;
```

## Poll

从完成队列中取得并移除下一个已经完成的任务。如果队列中暂时没有任何已完成任务，则返回 `null`。可选超时参数，使函数阻塞到超时时间再返回。

```java
/**
 * Retrieves and removes the Future representing the next
 * completed task, or {@code null} if none are present.
 *
 * @return the Future representing the next completed task, or
 *         {@code null} if none are present
 */
Future<V> poll();

/**
 * Retrieves and removes the Future representing the next
 * completed task, waiting if necessary up to the specified wait
 * time if none are yet present.
 *
 * @param timeout how long to wait before giving up, in units of
 *        {@code unit}
 * @param unit a {@code TimeUnit} determining how to interpret the
 *        {@code timeout} parameter
 * @return the Future representing the next completed task or
 *         {@code null} if the specified waiting time elapses
 *         before one is present
 * @throws InterruptedException if interrupted while waiting
 */
Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
```

---

