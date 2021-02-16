# Interface - java.util.concurrent.Future

Created by : Mr Dk.

2021 / 02 / 16 🧧 16:47

Ningbo, Zhejiang, China

---

## Definition

该接口定义了 **异步计算的结果**，可以被视为是一个占位符。接口内定义了以下函数：

* 检测异步计算是否完成
* 等待异步计算结束
* 获取异步计算的结果

```java
/**
 * A {@code Future} represents the result of an asynchronous
 * computation.  Methods are provided to check if the computation is
 * complete, to wait for its completion, and to retrieve the result of
 * the computation.  The result can only be retrieved using method
 * {@code get} when the computation has completed, blocking if
 * necessary until it is ready.  Cancellation is performed by the
 * {@code cancel} method.  Additional methods are provided to
 * determine if the task completed normally or was cancelled. Once a
 * computation has completed, the computation cannot be cancelled.
 * If you would like to use a {@code Future} for the sake
 * of cancellability but not provide a usable result, you can
 * declare types of the form {@code Future<?>} and
 * return {@code null} as a result of the underlying task.
 *
 * <p>
 * <b>Sample Usage</b> (Note that the following classes are all
 * made-up.)
 * <pre> {@code
 * interface ArchiveSearcher { String search(String target); }
 * class App {
 *   ExecutorService executor = ...
 *   ArchiveSearcher searcher = ...
 *   void showSearch(final String target)
 *       throws InterruptedException {
 *     Future<String> future
 *       = executor.submit(new Callable<String>() {
 *         public String call() {
 *             return searcher.search(target);
 *         }});
 *     displayOtherThings(); // do other things while searching
 *     try {
 *       displayText(future.get()); // use future
 *     } catch (ExecutionException ex) { cleanup(); return; }
 *   }
 * }}</pre>
 *
 * The {@link FutureTask} class is an implementation of {@code Future} that
 * implements {@code Runnable}, and so may be executed by an {@code Executor}.
 * For example, the above construction with {@code submit} could be replaced by:
 *  <pre> {@code
 * FutureTask<String> future =
 *   new FutureTask<String>(new Callable<String>() {
 *     public String call() {
 *       return searcher.search(target);
 *   }});
 * executor.execute(future);}</pre>
 *
 * <p>Memory consistency effects: Actions taken by the asynchronous computation
 * <a href="package-summary.html#MemoryVisibility"> <i>happen-before</i></a>
 * actions following the corresponding {@code Future.get()} in another thread.
 *
 * @see FutureTask
 * @see Executor
 * @since 1.5
 * @author Doug Lea
 * @param <V> The result type returned by this Future's {@code get} method
 */
public interface Future<V> {
    
}
```

## Cancel

试图取消任务的执行。如果任意已经执行完毕 / 已经被取消 / 无法被取消，那么尝试取消将会失败。如果任务还没有开始执行，那么取消成功后，任务将不会被执行；如果任务已经开始执行了，那么根据 `mayInterruptIfRunning` 参数可以决定是否中断正在执行的任务。

在这个函数被调用后，之后调用 `isDone()` 将总会返回 `true`；如果本函数返回 `true`，那么随后对本函数的再次调用也会返回 `true`。

```java
/**
 * Attempts to cancel execution of this task.  This attempt will
 * fail if the task has already completed, has already been cancelled,
 * or could not be cancelled for some other reason. If successful,
 * and this task has not started when {@code cancel} is called,
 * this task should never run.  If the task has already started,
 * then the {@code mayInterruptIfRunning} parameter determines
 * whether the thread executing this task should be interrupted in
 * an attempt to stop the task.
 *
 * <p>After this method returns, subsequent calls to {@link #isDone} will
 * always return {@code true}.  Subsequent calls to {@link #isCancelled}
 * will always return {@code true} if this method returned {@code true}.
 *
 * @param mayInterruptIfRunning {@code true} if the thread executing this
 * task should be interrupted; otherwise, in-progress tasks are allowed
 * to complete
 * @return {@code false} if the task could not be cancelled,
 * typically because it has already completed normally;
 * {@code true} otherwise
 */
boolean cancel(boolean mayInterruptIfRunning);
```

## Is Cancelled

返回当前任务是否已被取消。

```java
/**
 * Returns {@code true} if this task was cancelled before it completed
 * normally.
 *
 * @return {@code true} if this task was cancelled before it completed
 */
boolean isCancelled();
```

## Is Done

返回当前任务是否已经完成。

```java
/**
 * Returns {@code true} if this task completed.
 *
 * Completion may be due to normal termination, an exception, or
 * cancellation -- in all of these cases, this method will return
 * {@code true}.
 *
 * @return {@code true} if this task completed
 */
boolean isDone();
```

## Get

阻塞当前函数直到任务完成或超时，并取得任务的执行结果。

```java
/**
 * Waits if necessary for the computation to complete, and then
 * retrieves its result.
 *
 * @return the computed result
 * @throws CancellationException if the computation was cancelled
 * @throws ExecutionException if the computation threw an
 * exception
 * @throws InterruptedException if the current thread was interrupted
 * while waiting
 */
V get() throws InterruptedException, ExecutionException;

/**
 * Waits if necessary for at most the given time for the computation
 * to complete, and then retrieves its result, if available.
 *
 * @param timeout the maximum time to wait
 * @param unit the time unit of the timeout argument
 * @return the computed result
 * @throws CancellationException if the computation was cancelled
 * @throws ExecutionException if the computation threw an
 * exception
 * @throws InterruptedException if the current thread was interrupted
 * while waiting
 * @throws TimeoutException if the wait timed out
 */
V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;
```

---

