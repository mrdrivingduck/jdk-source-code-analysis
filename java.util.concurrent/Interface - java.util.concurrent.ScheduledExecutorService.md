# Interface - java.util.concurrent.ScheduledExecutorService

Created by : Mr Dk.

2021 / 02 / 16 🧧 15:48

Ningbo, Zhejiang, China

---

## Definition

该接口扩展了 `ExecutorService` 接口，使任务可以被调度至 **稍后执行**，或 **周期性执行**。其中定义的 `schedule()` 函数能够指定在任意延时后执行任务，而 `scheduleAtFixedRate()` 函数指定以固定的周期执行任务。对于父类接口中定义的 `execute()` 等相当于以零延时 (立刻) 执行任务。

所有的调度函数参数接收的都是 **相对时间**，而不是绝对时间。

```java
/**
 * An {@link ExecutorService} that can schedule commands to run after a given
 * delay, or to execute periodically.
 *
 * <p>The {@code schedule} methods create tasks with various delays
 * and return a task object that can be used to cancel or check
 * execution. The {@code scheduleAtFixedRate} and
 * {@code scheduleWithFixedDelay} methods create and execute tasks
 * that run periodically until cancelled.
 *
 * <p>Commands submitted using the {@link Executor#execute(Runnable)}
 * and {@link ExecutorService} {@code submit} methods are scheduled
 * with a requested delay of zero. Zero and negative delays (but not
 * periods) are also allowed in {@code schedule} methods, and are
 * treated as requests for immediate execution.
 *
 * <p>All {@code schedule} methods accept <em>relative</em> delays and
 * periods as arguments, not absolute times or dates. It is a simple
 * matter to transform an absolute time represented as a {@link
 * java.util.Date} to the required form. For example, to schedule at
 * a certain future {@code date}, you can use: {@code schedule(task,
 * date.getTime() - System.currentTimeMillis(),
 * TimeUnit.MILLISECONDS)}. Beware however that expiration of a
 * relative delay need not coincide with the current {@code Date} at
 * which the task is enabled due to network time synchronization
 * protocols, clock drift, or other factors.
 *
 * <p>The {@link Executors} class provides convenient factory methods for
 * the ScheduledExecutorService implementations provided in this package.
 *
 * <h3>Usage Example</h3>
 *
 * Here is a class with a method that sets up a ScheduledExecutorService
 * to beep every ten seconds for an hour:
 *
 *  <pre> {@code
 * import static java.util.concurrent.TimeUnit.*;
 * class BeeperControl {
 *   private final ScheduledExecutorService scheduler =
 *     Executors.newScheduledThreadPool(1);
 *
 *   public void beepForAnHour() {
 *     final Runnable beeper = new Runnable() {
 *       public void run() { System.out.println("beep"); }
 *     };
 *     final ScheduledFuture<?> beeperHandle =
 *       scheduler.scheduleAtFixedRate(beeper, 10, 10, SECONDS);
 *     scheduler.schedule(new Runnable() {
 *       public void run() { beeperHandle.cancel(true); }
 *     }, 60 * 60, SECONDS);
 *   }
 * }}</pre>
 *
 * @since 1.5
 * @author Doug Lea
 */
public interface ScheduledExecutorService extends ExecutorService {
    
}
```

## Schedule

在指定的延时后创建并执行任务。返回一个用于查询状态或停止执行的 `Future` 对象。

```java
/**
 * Creates and executes a one-shot action that becomes enabled
 * after the given delay.
 *
 * @param command the task to execute
 * @param delay the time from now to delay execution
 * @param unit the time unit of the delay parameter
 * @return a ScheduledFuture representing pending completion of
 *         the task and whose {@code get()} method will return
 *         {@code null} upon completion
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if command is null
 */
public ScheduledFuture<?> schedule(Runnable command,
                                   long delay, TimeUnit unit);

/**
 * Creates and executes a ScheduledFuture that becomes enabled after the
 * given delay.
 *
 * @param callable the function to execute
 * @param delay the time from now to delay execution
 * @param unit the time unit of the delay parameter
 * @param <V> the type of the callable's result
 * @return a ScheduledFuture that can be used to extract result or cancel
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if callable is null
 */
public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                       long delay, TimeUnit unit);
```

## Schedule at Fixed Rate

任务将在一段 **初始延时** 之后，以固定的周期被启动执行。任务只能通过取消或关闭执行器的方式终结。如果任务的执行超过了执行周期，随后的任务将会暂缓开始，而不会并发执行。

注意，下一个任务的启动时间只和任务的执行周期有关，与任务执行期间耗费的时间无关。这里的 **周期** 指的是 **前一个任务的开始时刻** 到 **下一个任务的开始时刻** 的时间差。

```java
/**
 * Creates and executes a periodic action that becomes enabled first
 * after the given initial delay, and subsequently with the given
 * period; that is executions will commence after
 * {@code initialDelay} then {@code initialDelay+period}, then
 * {@code initialDelay + 2 * period}, and so on.
 * If any execution of the task
 * encounters an exception, subsequent executions are suppressed.
 * Otherwise, the task will only terminate via cancellation or
 * termination of the executor.  If any execution of this task
 * takes longer than its period, then subsequent executions
 * may start late, but will not concurrently execute.
 *
 * @param command the task to execute
 * @param initialDelay the time to delay first execution
 * @param period the period between successive executions
 * @param unit the time unit of the initialDelay and period parameters
 * @return a ScheduledFuture representing pending completion of
 *         the task, and whose {@code get()} method will throw an
 *         exception upon cancellation
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if command is null
 * @throws IllegalArgumentException if period less than or equal to zero
 */
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                              long initialDelay,
                                              long period,
                                              TimeUnit unit);
```

## Schedule with Fixed Delay

任务将在一段 **初始延时** 之后，以固定的 **间隔** 被执行。间隔的具体含义是 **前一个任务的结束时刻** 到 **下一个任务的开始时刻** 之间的时间差。

```java
/**
 * Creates and executes a periodic action that becomes enabled first
 * after the given initial delay, and subsequently with the
 * given delay between the termination of one execution and the
 * commencement of the next.  If any execution of the task
 * encounters an exception, subsequent executions are suppressed.
 * Otherwise, the task will only terminate via cancellation or
 * termination of the executor.
 *
 * @param command the task to execute
 * @param initialDelay the time to delay first execution
 * @param delay the delay between the termination of one
 * execution and the commencement of the next
 * @param unit the time unit of the initialDelay and delay parameters
 * @return a ScheduledFuture representing pending completion of
 *         the task, and whose {@code get()} method will throw an
 *         exception upon cancellation
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if command is null
 * @throws IllegalArgumentException if delay less than or equal to zero
 */
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                 long initialDelay,
                                                 long delay,
                                                 TimeUnit unit);
```

---

