# Interface - java.util.concurrent.ExecutorService

Created by : Mr Dk.

2021 / 02 / 15 22:34

Ningbo, Zhejiang, China

---

## Definition

该接口扩展了 `Executor`，除了 `execute()` 提供了任务执行的方式外，该接口提供了 **终结管理** 以及 **追踪一个或多个异步任务的状态** 的能力：

* 使执行器能够被终结：停止接受新任务 / 将已有任务暂停并关闭
* 扩展 `execute()` 为 `submit()`，使任务能够返回一个 `Future` 对象，可用于中止任务或等待任务完成

总体来说，该接口定义了 **池化** 所必须的管理函数。

```java
/**
 * An {@link Executor} that provides methods to manage termination and
 * methods that can produce a {@link Future} for tracking progress of
 * one or more asynchronous tasks.
 *
 * <p>An {@code ExecutorService} can be shut down, which will cause
 * it to reject new tasks.  Two different methods are provided for
 * shutting down an {@code ExecutorService}. The {@link #shutdown}
 * method will allow previously submitted tasks to execute before
 * terminating, while the {@link #shutdownNow} method prevents waiting
 * tasks from starting and attempts to stop currently executing tasks.
 * Upon termination, an executor has no tasks actively executing, no
 * tasks awaiting execution, and no new tasks can be submitted.  An
 * unused {@code ExecutorService} should be shut down to allow
 * reclamation of its resources.
 *
 * <p>Method {@code submit} extends base method {@link
 * Executor#execute(Runnable)} by creating and returning a {@link Future}
 * that can be used to cancel execution and/or wait for completion.
 * Methods {@code invokeAny} and {@code invokeAll} perform the most
 * commonly useful forms of bulk execution, executing a collection of
 * tasks and then waiting for at least one, or all, to
 * complete. (Class {@link ExecutorCompletionService} can be used to
 * write customized variants of these methods.)
 *
 * <p>The {@link Executors} class provides factory methods for the
 * executor services provided in this package.
 *
 * <h3>Usage Examples</h3>
 *
 * Here is a sketch of a network service in which threads in a thread
 * pool service incoming requests. It uses the preconfigured {@link
 * Executors#newFixedThreadPool} factory method:
 *
 *  <pre> {@code
 * class NetworkService implements Runnable {
 *   private final ServerSocket serverSocket;
 *   private final ExecutorService pool;
 *
 *   public NetworkService(int port, int poolSize)
 *       throws IOException {
 *     serverSocket = new ServerSocket(port);
 *     pool = Executors.newFixedThreadPool(poolSize);
 *   }
 *
 *   public void run() { // run the service
 *     try {
 *       for (;;) {
 *         pool.execute(new Handler(serverSocket.accept()));
 *       }
 *     } catch (IOException ex) {
 *       pool.shutdown();
 *     }
 *   }
 * }
 *
 * class Handler implements Runnable {
 *   private final Socket socket;
 *   Handler(Socket socket) { this.socket = socket; }
 *   public void run() {
 *     // read and service request on socket
 *   }
 * }}</pre>
 *
 * The following method shuts down an {@code ExecutorService} in two phases,
 * first by calling {@code shutdown} to reject incoming tasks, and then
 * calling {@code shutdownNow}, if necessary, to cancel any lingering tasks:
 *
 *  <pre> {@code
 * void shutdownAndAwaitTermination(ExecutorService pool) {
 *   pool.shutdown(); // Disable new tasks from being submitted
 *   try {
 *     // Wait a while for existing tasks to terminate
 *     if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
 *       pool.shutdownNow(); // Cancel currently executing tasks
 *       // Wait a while for tasks to respond to being cancelled
 *       if (!pool.awaitTermination(60, TimeUnit.SECONDS))
 *           System.err.println("Pool did not terminate");
 *     }
 *   } catch (InterruptedException ie) {
 *     // (Re-)Cancel if current thread also interrupted
 *     pool.shutdownNow();
 *     // Preserve interrupt status
 *     Thread.currentThread().interrupt();
 *   }
 * }}</pre>
 *
 * <p>Memory consistency effects: Actions in a thread prior to the
 * submission of a {@code Runnable} or {@code Callable} task to an
 * {@code ExecutorService}
 * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 * any actions taken by that task, which in turn <i>happen-before</i> the
 * result is retrieved via {@code Future.get()}.
 *
 * @since 1.5
 * @author Doug Lea
 */
public interface ExecutorService extends Executor {
    
}
```

## Shut Down

`shutdown()` 使已经被提交的任务有序完成，但是不再接受任何新的任务。该函数不等待之前提交的任务执行完成。

```java
/**
 * Initiates an orderly shutdown in which previously submitted
 * tasks are executed, but no new tasks will be accepted.
 * Invocation has no additional effect if already shut down.
 *
 * <p>This method does not wait for previously submitted tasks to
 * complete execution.  Use {@link #awaitTermination awaitTermination}
 * to do that.
 *
 * @throws SecurityException if a security manager exists and
 *         shutting down this ExecutorService may manipulate
 *         threads that the caller is not permitted to modify
 *         because it does not hold {@link
 *         java.lang.RuntimePermission}{@code ("modifyThread")},
 *         or the security manager's {@code checkAccess} method
 *         denies access.
 */
void shutdown();
```

`shutdownNow()` 试图停止所有正在执行的任务，停止处理正在等待的任务，并返回所有等待被执行的任务。该函数不等待之前提交的任务执行完成。

```java
/**
 * Attempts to stop all actively executing tasks, halts the
 * processing of waiting tasks, and returns a list of the tasks
 * that were awaiting execution.
 *
 * <p>This method does not wait for actively executing tasks to
 * terminate.  Use {@link #awaitTermination awaitTermination} to
 * do that.
 *
 * <p>There are no guarantees beyond best-effort attempts to stop
 * processing actively executing tasks.  For example, typical
 * implementations will cancel via {@link Thread#interrupt}, so any
 * task that fails to respond to interrupts may never terminate.
 *
 * @return list of tasks that never commenced execution
 * @throws SecurityException if a security manager exists and
 *         shutting down this ExecutorService may manipulate
 *         threads that the caller is not permitted to modify
 *         because it does not hold {@link
 *         java.lang.RuntimePermission}{@code ("modifyThread")},
 *         or the security manager's {@code checkAccess} method
 *         denies access.
 */
List<Runnable> shutdownNow();
```

返回执行器是否已被关闭。

```java
/**
 * Returns {@code true} if this executor has been shut down.
 *
 * @return {@code true} if this executor has been shut down
 */
boolean isShutdown();
```

返回执行器中的所有任务是否已经停止。

```java
/**
 * Returns {@code true} if all tasks have completed following shut down.
 * Note that {@code isTerminated} is never {@code true} unless
 * either {@code shutdown} or {@code shutdownNow} was called first.
 *
 * @return {@code true} if all tasks have completed following shut down
 */
boolean isTerminated();
```

阻塞等待执行器中的所有任务执行完成，或超时 / 中断。

```java
/**
 * Blocks until all tasks have completed execution after a shutdown
 * request, or the timeout occurs, or the current thread is
 * interrupted, whichever happens first.
 *
 * @param timeout the maximum time to wait
 * @param unit the time unit of the timeout argument
 * @return {@code true} if this executor terminated and
 *         {@code false} if the timeout elapsed before termination
 * @throws InterruptedException if interrupted while waiting
 */
boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException;
```

## Submit

提交一个指定类型返回值的任务。`Future` 可被视为是任务未来执行结果的 **占位符**，在任务成功完成后，将会返回任务的结果。该函数不会阻塞，如果想要阻塞获取结果，可以立刻调用返回的 `Future` 对象的 `get()` 函数。

```java
/**
 * Submits a value-returning task for execution and returns a
 * Future representing the pending results of the task. The
 * Future's {@code get} method will return the task's result upon
 * successful completion.
 *
 * <p>
 * If you would like to immediately block waiting
 * for a task, you can use constructions of the form
 * {@code result = exec.submit(aCallable).get();}
 *
 * <p>Note: The {@link Executors} class includes a set of methods
 * that can convert some other common closure-like objects,
 * for example, {@link java.security.PrivilegedAction} to
 * {@link Callable} form so they can be submitted.
 *
 * @param task the task to submit
 * @param <T> the type of the task's result
 * @return a Future representing pending completion of the task
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if the task is null
 */
<T> Future<T> submit(Callable<T> task);

/**
 * Submits a Runnable task for execution and returns a Future
 * representing that task. The Future's {@code get} method will
 * return {@code null} upon <em>successful</em> completion.
 *
 * @param task the task to submit
 * @return a Future representing pending completion of the task
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if the task is null
 */
Future<?> submit(Runnable task);
```

下面的函数与上述函数类似，但是多了一个 `T` 类型的参数。在未来任务完成时，将返回这个参数的值。

```java
/**
 * Submits a Runnable task for execution and returns a Future
 * representing that task. The Future's {@code get} method will
 * return the given result upon successful completion.
 *
 * @param task the task to submit
 * @param result the result to return
 * @param <T> the type of the result
 * @return a Future representing pending completion of the task
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if the task is null
 */
<T> Future<T> submit(Runnable task, T result);
```

## Invoke

执行给定的所有任务，在 **所有任务完成后**，返回一个列表的 `Future` 对象，其中持有每个任务的状态和结果。每个 `Future` 的 `isDone()` 都是 `true`。在任务执行期间对集合进行修改后，该函数的结果未知。

```java
/**
 * Executes the given tasks, returning a list of Futures holding
 * their status and results when all complete.
 * {@link Future#isDone} is {@code true} for each
 * element of the returned list.
 * Note that a <em>completed</em> task could have
 * terminated either normally or by throwing an exception.
 * The results of this method are undefined if the given
 * collection is modified while this operation is in progress.
 *
 * @param tasks the collection of tasks
 * @param <T> the type of the values returned from the tasks
 * @return a list of Futures representing the tasks, in the same
 *         sequential order as produced by the iterator for the
 *         given task list, each of which has completed
 * @throws InterruptedException if interrupted while waiting, in
 *         which case unfinished tasks are cancelled
 * @throws NullPointerException if tasks or any of its elements are {@code null}
 * @throws RejectedExecutionException if any task cannot be
 *         scheduled for execution
 */
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
    throws InterruptedException;
```

上述函数的超时版本。如果阻塞时间已过，则函数立刻返回。返回时，如果任务没有完成则将被取消。同样，每个 `Future` 的 `isDone()` 都将返回 `true`。

```java
/**
 * Executes the given tasks, returning a list of Futures holding
 * their status and results
 * when all complete or the timeout expires, whichever happens first.
 * {@link Future#isDone} is {@code true} for each
 * element of the returned list.
 * Upon return, tasks that have not completed are cancelled.
 * Note that a <em>completed</em> task could have
 * terminated either normally or by throwing an exception.
 * The results of this method are undefined if the given
 * collection is modified while this operation is in progress.
 *
 * @param tasks the collection of tasks
 * @param timeout the maximum time to wait
 * @param unit the time unit of the timeout argument
 * @param <T> the type of the values returned from the tasks
 * @return a list of Futures representing the tasks, in the same
 *         sequential order as produced by the iterator for the
 *         given task list. If the operation did not time out,
 *         each task will have completed. If it did time out, some
 *         of these tasks will not have completed.
 * @throws InterruptedException if interrupted while waiting, in
 *         which case unfinished tasks are cancelled
 * @throws NullPointerException if tasks, any of its elements, or
 *         unit are {@code null}
 * @throws RejectedExecutionException if any task cannot be scheduled
 *         for execution
 */
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                              long timeout, TimeUnit unit)
    throws InterruptedException;
```

执行给定的所有任务，当其中任意一个成功完成时，返回结果。函数返回后，还没有完成的任务将会被取消。

```java
/**
 * Executes the given tasks, returning the result
 * of one that has completed successfully (i.e., without throwing
 * an exception), if any do. Upon normal or exceptional return,
 * tasks that have not completed are cancelled.
 * The results of this method are undefined if the given
 * collection is modified while this operation is in progress.
 *
 * @param tasks the collection of tasks
 * @param <T> the type of the values returned from the tasks
 * @return the result returned by one of the tasks
 * @throws InterruptedException if interrupted while waiting
 * @throws NullPointerException if tasks or any element task
 *         subject to execution is {@code null}
 * @throws IllegalArgumentException if tasks is empty
 * @throws ExecutionException if no task successfully completes
 * @throws RejectedExecutionException if tasks cannot be scheduled
 *         for execution
 */
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
    throws InterruptedException, ExecutionException;
```

上述任务的超时版本。

```java
/**
 * Executes the given tasks, returning the result
 * of one that has completed successfully (i.e., without throwing
 * an exception), if any do before the given timeout elapses.
 * Upon normal or exceptional return, tasks that have not
 * completed are cancelled.
 * The results of this method are undefined if the given
 * collection is modified while this operation is in progress.
 *
 * @param tasks the collection of tasks
 * @param timeout the maximum time to wait
 * @param unit the time unit of the timeout argument
 * @param <T> the type of the values returned from the tasks
 * @return the result returned by one of the tasks
 * @throws InterruptedException if interrupted while waiting
 * @throws NullPointerException if tasks, or unit, or any element
 *         task subject to execution is {@code null}
 * @throws TimeoutException if the given timeout elapses before
 *         any task successfully completes
 * @throws ExecutionException if no task successfully completes
 * @throws RejectedExecutionException if tasks cannot be scheduled
 *         for execution
 */
<T> T invokeAny(Collection<? extends Callable<T>> tasks,
                long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;
```

---

