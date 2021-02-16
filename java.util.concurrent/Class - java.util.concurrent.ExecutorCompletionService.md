# Class - java.util.concurrent.ExecutorCompletionService

Created by : Mr Dk.

2021 / 02 / 16 🧧 21:41

Ningbo, Zhejiang, China

---

## Definition

该类提供了 `CompletionService` 接口的轻量级实现。类内使用了一个 `Executor` 来执行任务，在任务执行完成后，将会保存到一个队列中，通过 `take()` 函数来进行访问。

```java
/**
 * A {@link CompletionService} that uses a supplied {@link Executor}
 * to execute tasks.  This class arranges that submitted tasks are,
 * upon completion, placed on a queue accessible using {@code take}.
 * The class is lightweight enough to be suitable for transient use
 * when processing groups of tasks.
 *
 * <p>
 *
 * <b>Usage Examples.</b>
 *
 * Suppose you have a set of solvers for a certain problem, each
 * returning a value of some type {@code Result}, and would like to
 * run them concurrently, processing the results of each of them that
 * return a non-null value, in some method {@code use(Result r)}. You
 * could write this as:
 *
 * <pre> {@code
 * void solve(Executor e,
 *            Collection<Callable<Result>> solvers)
 *     throws InterruptedException, ExecutionException {
 *     CompletionService<Result> ecs
 *         = new ExecutorCompletionService<Result>(e);
 *     for (Callable<Result> s : solvers)
 *         ecs.submit(s);
 *     int n = solvers.size();
 *     for (int i = 0; i < n; ++i) {
 *         Result r = ecs.take().get();
 *         if (r != null)
 *             use(r);
 *     }
 * }}</pre>
 *
 * Suppose instead that you would like to use the first non-null result
 * of the set of tasks, ignoring any that encounter exceptions,
 * and cancelling all other tasks when the first one is ready:
 *
 * <pre> {@code
 * void solve(Executor e,
 *            Collection<Callable<Result>> solvers)
 *     throws InterruptedException {
 *     CompletionService<Result> ecs
 *         = new ExecutorCompletionService<Result>(e);
 *     int n = solvers.size();
 *     List<Future<Result>> futures
 *         = new ArrayList<Future<Result>>(n);
 *     Result result = null;
 *     try {
 *         for (Callable<Result> s : solvers)
 *             futures.add(ecs.submit(s));
 *         for (int i = 0; i < n; ++i) {
 *             try {
 *                 Result r = ecs.take().get();
 *                 if (r != null) {
 *                     result = r;
 *                     break;
 *                 }
 *             } catch (ExecutionException ignore) {}
 *         }
 *     }
 *     finally {
 *         for (Future<Result> f : futures)
 *             f.cancel(true);
 *     }
 *
 *     if (result != null)
 *         use(result);
 * }}</pre>
 */
public class ExecutorCompletionService<V> implements CompletionService<V> {

}
```

## Executor

类内维护了 `Executor` 对象以执行任务。特别地，如果 `Executor` 对象实现了 `AbstractExecutorService` 抽象类，则将使用 `AbstractExecutorService` 提供的函数：

```java
private final Executor executor;
private final AbstractExecutorService aes;

private RunnableFuture<V> newTaskFor(Callable<V> task) {
    if (aes == null)
        return new FutureTask<V>(task);
    else
        return aes.newTaskFor(task);
}

private RunnableFuture<V> newTaskFor(Runnable task, V result) {
    if (aes == null)
        return new FutureTask<V>(task, result);
    else
        return aes.newTaskFor(task, result);
}
```

## Queueing Future

类内定义了一个阻塞队列来维护所有已完成任务的 `Future` 对象。`FutureTask` 类被继承为 `QueueingFuture` 类，重写了任务完成后会被回调的 `done()` 函数 (`FutureTask` 中默认什么都不做)。在 `done()` 中，使已经完成的任务加入阻塞队列。

```java
private final BlockingQueue<Future<V>> completionQueue;

/**
 * FutureTask extension to enqueue upon completion
 */
private class QueueingFuture extends FutureTask<Void> {
    QueueingFuture(RunnableFuture<V> task) {
        super(task, null);
        this.task = task;
    }
    protected void done() { completionQueue.add(task); }
    private final Future<V> task;
}
```

## Constructor

构造函数需要传入一个 `Executor` 对象。根据 `Executor` 是否继承自 `AbstractExecutorService` 决定是否启用成员变量 `aes`。阻塞队列默认会被初始化为 `LinkedBlockingQueue`，但用户也可以自行传入继承自 `BlockingQueue` 的阻塞队列。

```java
/**
 * Creates an ExecutorCompletionService using the supplied
 * executor for base task execution and a
 * {@link LinkedBlockingQueue} as a completion queue.
 *
 * @param executor the executor to use
 * @throws NullPointerException if executor is {@code null}
 */
public ExecutorCompletionService(Executor executor) {
    if (executor == null)
        throw new NullPointerException();
    this.executor = executor;
    this.aes = (executor instanceof AbstractExecutorService) ?
        (AbstractExecutorService) executor : null;
    this.completionQueue = new LinkedBlockingQueue<Future<V>>();
}

/**
 * Creates an ExecutorCompletionService using the supplied
 * executor for base task execution and the supplied queue as its
 * completion queue.
 *
 * @param executor the executor to use
 * @param completionQueue the queue to use as the completion queue
 *        normally one dedicated for use by this service. This
 *        queue is treated as unbounded -- failed attempted
 *        {@code Queue.add} operations for completed tasks cause
 *        them not to be retrievable.
 * @throws NullPointerException if executor or completionQueue are {@code null}
 */
public ExecutorCompletionService(Executor executor,
                                 BlockingQueue<Future<V>> completionQueue) {
    if (executor == null || completionQueue == null)
        throw new NullPointerException();
    this.executor = executor;
    this.aes = (executor instanceof AbstractExecutorService) ?
        (AbstractExecutorService) executor : null;
    this.completionQueue = completionQueue;
}
```

## Submit

将任务包装为可被执行的 `QueueFuture` 对象，然后使用 `executor` 执行任务。任务完成后，其 `done()` 会被调用使其进入完成队列。

```java
public Future<V> submit(Callable<V> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<V> f = newTaskFor(task);
    executor.execute(new QueueingFuture(f));
    return f;
}

public Future<V> submit(Runnable task, V result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<V> f = newTaskFor(task, result);
    executor.execute(new QueueingFuture(f));
    return f;
}
```

## Take / Poll

以阻塞 (`take()`) 或非阻塞超时 (`poll()`) 的方式从完成队列中取出任务的 `FutureTask` 对象。

```java
public Future<V> take() throws InterruptedException {
    return completionQueue.take();
}

public Future<V> poll() {
    return completionQueue.poll();
}

public Future<V> poll(long timeout, TimeUnit unit)
    throws InterruptedException {
    return completionQueue.poll(timeout, unit);
}
```

---

