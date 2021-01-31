# Class - java.util.concurrent.ThreadPoolExecutor

Created by : Mr Dk.

2020 / 11 / 02 21:12

Nanjing, Jiangsu, China

---

## Definition

线程池的实现类。由于这个类的源码和注释较多，因此决定直接根据功能分模块进行简要分析。线程池的概要可以参阅另一篇 [笔记](https://mrdrivingduck.github.io/blog/#/markdown?repo=notes&path=Java%2FJava%20Thread%20Pool.md)。

```java
public class ThreadPoolExecutor extends AbstractExecutorService {

}
```

---

## Thread Pool Life Cycle

线程池的运行状态由线程池对象自行维护，主要包含两个数值：

* 线程池运行状态
* 线程池内的线程数量

这两者共同使用一个原子 Integer 变量，其中线程池运行状态使用 **高 3 位**，线程池内线程数量使用 **低 29 位**，两个数值在变量内互相独立不干扰。共用一个变量省去了为了维护两个数值的一致性而需要占用的锁资源。

```java
/**
 * The main pool control state, ctl, is an atomic integer packing
 * two conceptual fields
 *   workerCount, indicating the effective number of threads
 *   runState,    indicating whether running, shutting down etc
 *
 * In order to pack them into one int, we limit workerCount to
 * (2^29)-1 (about 500 million) threads rather than (2^31)-1 (2
 * billion) otherwise representable. If this is ever an issue in
 * the future, the variable can be changed to be an AtomicLong,
 * and the shift/mask constants below adjusted. But until the need
 * arises, this code is a bit faster and simpler using an int.
 *
 * The workerCount is the number of workers that have been
 * permitted to start and not permitted to stop.  The value may be
 * transiently different from the actual number of live threads,
 * for example when a ThreadFactory fails to create a thread when
 * asked, and when exiting threads are still performing
 * bookkeeping before terminating. The user-visible pool size is
 * reported as the current size of the workers set.
 *
 * The runState provides the main lifecycle control, taking on values:
 *
 *   RUNNING:  Accept new tasks and process queued tasks
 *   SHUTDOWN: Don't accept new tasks, but process queued tasks
 *   STOP:     Don't accept new tasks, don't process queued tasks,
 *             and interrupt in-progress tasks
 *   TIDYING:  All tasks have terminated, workerCount is zero,
 *             the thread transitioning to state TIDYING
 *             will run the terminated() hook method
 *   TERMINATED: terminated() has completed
 *
 * The numerical order among these values matters, to allow
 * ordered comparisons. The runState monotonically increases over
 * time, but need not hit each state. The transitions are:
 *
 * RUNNING -> SHUTDOWN
 *    On invocation of shutdown(), perhaps implicitly in finalize()
 * (RUNNING or SHUTDOWN) -> STOP
 *    On invocation of shutdownNow()
 * SHUTDOWN -> TIDYING
 *    When both queue and pool are empty
 * STOP -> TIDYING
 *    When pool is empty
 * TIDYING -> TERMINATED
 *    When the terminated() hook method has completed
 *
 * Threads waiting in awaitTermination() will return when the
 * state reaches TERMINATED.
 *
 * Detecting the transition from SHUTDOWN to TIDYING is less
 * straightforward than you'd like because the queue may become
 * empty after non-empty and vice versa during SHUTDOWN state, but
 * we can only terminate if, after seeing that it is empty, we see
 * that workerCount is 0 (which sometimes entails a recheck -- see
 * below).
 */
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }

/*
 * Bit field accessors that don't require unpacking ctl.
 * These depend on the bit layout and on workerCount being never negative.
 */

private static boolean runStateLessThan(int c, int s) {
    return c < s;
}

private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
}

private static boolean isRunning(int c) {
    return c < SHUTDOWN;
}
```

可以看到，根据 `ctl` 域高三位的定义，线程池的状态可以为如下几种：

* `RUNNING` - 线程池运行中，能接收新提交的任务，也能处理工作队列中的任务
* `SHUTDOWN` - 关闭状态，停止接收新任务，但可以继续处理工作队列中滞留的任务
* `STOP` - 停止状态，不接收新任务，也不处理工作队列中的任务，并中断所有正在处理中的线程
* `TIDYING` - 所有任务都已终止
* `TERMINATED`

在线程池正常运行 (`RUNNING`) 状态后，调用 `shutdown()` 和 `shutdownNoew()` 后将分别进入 `SHUTDOWN` 或 `STOP` 状态，然后同时进入 `TIDYING` 状态。在 `terminated()` 函数结束后，线程池进入 `TERMINATED` 状态。

## Task Scheduling

最核心的 `execute()` 函数实现自 `Executor` 接口，完成了线程池调度任务和核心功能。

1. 如果线程池内线程数量低于 `corePoolSize`，那么创建新的线程处理输入的任务
2. 如果线程数量达到 `corePoolSize`，则任务进入工作队列暂存 (需要 recheck)
3. 如果线程数量达到 `corePoolSize`，且任务无法进入工作队列 (队列已满)，则试图建立新的线程
4. 如果无法建立新的线程 (达到 `maxPoolSize`)，那么用相应的策略拒绝任务

以这个逻辑看 `execute()` 函数就很明白了：

```java
/**
 * Executes the given task sometime in the future.  The task
 * may execute in a new thread or in an existing pooled thread.
 *
 * If the task cannot be submitted for execution, either because this
 * executor has been shutdown or because its capacity has been reached,
 * the task is handled by the current {@code RejectedExecutionHandler}.
 *
 * @param command the task to execute
 * @throws RejectedExecutionException at discretion of
 *         {@code RejectedExecutionHandler}, if the task
 *         cannot be accepted for execution
 * @throws NullPointerException if {@code command} is null
 */
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

## Worker Management

### Add Worker

线程池内部的工作线程被称为 worker。在上述函数中可以看到，`addWorker()` 函数负责新建线程：

* `firstTask` - 创建线程后直接开始执行的初始任务
* `core` - 判断是否可以继续创建线程的标准是 `corePoolSize` 还是 `maximumPoolSize`

首先在一个死循环中，根据线程池的当前运行状态和线程数来决定是否可以创建新线程。成功通过 CAS 将线程数 +1 的线程将退出死循环。之后，新建一个线程，在获得线程池的 `mainLock` 锁后，将线程加入到线程池的维护中。同时还会更新 `largestPoolSize`，该变量追踪线程池达到过的最大尺寸。

```java
/**
 * Checks if a new worker can be added with respect to current
 * pool state and the given bound (either core or maximum). If so,
 * the worker count is adjusted accordingly, and, if possible, a
 * new worker is created and started, running firstTask as its
 * first task. This method returns false if the pool is stopped or
 * eligible to shut down. It also returns false if the thread
 * factory fails to create a thread when asked.  If the thread
 * creation fails, either due to the thread factory returning
 * null, or due to an exception (typically OutOfMemoryError in
 * Thread.start()), we roll back cleanly.
 *
 * @param firstTask the task the new thread should run first (or
 * null if none). Workers are created with an initial first task
 * (in method execute()) to bypass queuing when there are fewer
 * than corePoolSize threads (in which case we always start one),
 * or when the queue is full (in which case we must bypass queue).
 * Initially idle threads are usually created via
 * prestartCoreThread or to replace other dying workers.
 *
 * @param core if true use corePoolSize as bound, else
 * maximumPoolSize. (A boolean indicator is used here rather than a
 * value to ensure reads of fresh values after checking other pool
 * state).
 * @return true if successful
 */
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
                firstTask == null &&
                ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

### Worker Definition

在 worker 的具体定义中可以看到，worker 结点继承自 AQS，并且在对象内部持有一个线程与初始任务。使用 AQS 的目的是为了简化工作线程执行每个任务前后的加锁和解锁。没有使用 `ReentrantLock` 的原因是，在调用线程池控制函数时，不希望锁能被重入。

```java
/**
 * Class Worker mainly maintains interrupt control state for
 * threads running tasks, along with other minor bookkeeping.
 * This class opportunistically extends AbstractQueuedSynchronizer
 * to simplify acquiring and releasing a lock surrounding each
 * task execution.  This protects against interrupts that are
 * intended to wake up a worker thread waiting for a task from
 * instead interrupting a task being run.  We implement a simple
 * non-reentrant mutual exclusion lock rather than use
 * ReentrantLock because we do not want worker tasks to be able to
 * reacquire the lock when they invoke pool control methods like
 * setCorePoolSize.  Additionally, to suppress interrupts until
 * the thread actually starts running tasks, we initialize lock
 * state to a negative value, and clear it upon start (in
 * runWorker).
 */
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    /**
        * This class will never be serialized, but we provide a
        * serialVersionUID to suppress a javac warning.
        */
    private static final long serialVersionUID = 6138294804551838833L;

    /** Thread this worker is running in.  Null if factory fails. */
    final Thread thread;
    /** Initial task to run.  Possibly null. */
    Runnable firstTask;
    /** Per-thread task counter */
    volatile long completedTasks;

}
```

### Worker Maintainess

线程池通过一个 `HashSet` 来持有所有 worker 的引用。这个集合只有在持有线程池 `mainLock` 的前提下才能被操作。

```java
/**
 * Set containing all worker threads in pool. Accessed only when
 * holding mainLock.
 */
private final HashSet<Worker> workers = new HashSet<Worker>();
```

### Get Task

对每一个 worker 来说，获取任务的过程中，需要判断线程池的状态以及目前线程池中的线程数量。如下几种情况会造成 worker 的退出并返回 `null`：

1. 当前线程数超出 `maximumPoolSize`
2. 线程池被停止
3. 线程池被关闭，且工作队列已经为空
4. Worker 等待任务超时，主动终结 (`allowCoreThreadTimeOut || workerCount > corePoolSize`)

```java
/**
 * Performs blocking or timed wait for a task, depending on
 * current configuration settings, or returns null if this worker
 * must exit because of any of:
 * 1. There are more than maximumPoolSize workers (due to
 *    a call to setMaximumPoolSize).
 * 2. The pool is stopped.
 * 3. The pool is shutdown and the queue is empty.
 * 4. This worker timed out waiting for a task, and timed-out
 *    workers are subject to termination (that is,
 *    {@code allowCoreThreadTimeOut || workerCount > corePoolSize})
 *    both before and after the timed wait, and if the queue is
 *    non-empty, this worker is not the last thread in the pool.
 *
 * @return task, or null if the worker must exit, in which case
 *         workerCount is decremented
 */
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

`allowCoreThreadTimeout` 变量如果为 `true`，那么核心线程在空闲超过 `keepAliveTime` 后会自动终结：

```java
/**
 * Timeout in nanoseconds for idle threads waiting for work.
 * Threads use this timeout when there are more than corePoolSize
 * present or if allowCoreThreadTimeOut. Otherwise they wait
 * forever for new work.
 */
private volatile long keepAliveTime;

/**
 * If false (default), core threads stay alive even when idle.
 * If true, core threads use keepAliveTime to time out waiting
 * for work.
 */
private volatile boolean allowCoreThreadTimeOut;
```

### Worker Main Loop

每个 worker 要执行的任务来源于两处：

1. 初始任务
2. 工作队列

在一个循环中，不断将当前任务执行完毕，然后从工作队列中获取新的任务执行。直到 worker 消亡，进行资源清理。

```java
/**
 * Main worker run loop.  Repeatedly gets tasks from queue and
 * executes them, while coping with a number of issues:
 *
 * 1. We may start out with an initial task, in which case we
 * don't need to get the first one. Otherwise, as long as pool is
 * running, we get tasks from getTask. If it returns null then the
 * worker exits due to changed pool state or configuration
 * parameters.  Other exits result from exception throws in
 * external code, in which case completedAbruptly holds, which
 * usually leads processWorkerExit to replace this thread.
 *
 * 2. Before running any task, the lock is acquired to prevent
 * other pool interrupts while the task is executing, and then we
 * ensure that unless pool is stopping, this thread does not have
 * its interrupt set.
 *
 * 3. Each task run is preceded by a call to beforeExecute, which
 * might throw an exception, in which case we cause thread to die
 * (breaking loop with completedAbruptly true) without processing
 * the task.
 *
 * 4. Assuming beforeExecute completes normally, we run the task,
 * gathering any of its thrown exceptions to send to afterExecute.
 * We separately handle RuntimeException, Error (both of which the
 * specs guarantee that we trap) and arbitrary Throwables.
 * Because we cannot rethrow Throwables within Runnable.run, we
 * wrap them within Errors on the way out (to the thread's
 * UncaughtExceptionHandler).  Any thrown exception also
 * conservatively causes thread to die.
 *
 * 5. After task.run completes, we call afterExecute, which may
 * also throw an exception, which will also cause thread to
 * die. According to JLS Sec 14.20, this exception is the one that
 * will be in effect even if task.run throws.
 *
 * The net effect of the exception mechanics is that afterExecute
 * and the thread's UncaughtExceptionHandler have as accurate
 * information as we can provide about any problems encountered by
 * user code.
 *
 * @param w the worker
 */
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                    (Thread.interrupted() &&
                    runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

在 `finally` 块中执行的 `processWorkerExit()` 对线程进行清理。将 worker 的引用从线程池中解除，并将线程完成执行的任务数统计到线程池中。

```java
/**
 * Performs cleanup and bookkeeping for a dying worker. Called
 * only from worker threads. Unless completedAbruptly is set,
 * assumes that workerCount has already been adjusted to account
 * for exit.  This method removes thread from worker set, and
 * possibly terminates the pool or replaces the worker if either
 * it exited due to user task exception or if fewer than
 * corePoolSize workers are running or queue is non-empty but
 * there are no workers.
 *
 * @param w the worker
 * @param completedAbruptly if the worker died due to user exception
 */
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        completedTaskCount += w.completedTasks;
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }

    tryTerminate();

    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {
        if (!completedAbruptly) {
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        addWorker(null, false);
    }
}
```

## Reject Strategies

当线程池中的线程数量已经达到 `maxPoolSize`，且工作队列已满，此时线程池不再接收任何新任务，而是使用指定的策略拒绝调用线程。拒绝策略需要实现 `RejectedExecutionHandler` 接口，其中的 `rejectedExecution()` 函数会被线程池对象调用：

```java
/**
 * Method that may be invoked by a {@link ThreadPoolExecutor} when
 * {@link ThreadPoolExecutor#execute execute} cannot accept a
 * task.  This may occur when no more threads or queue slots are
 * available because their bounds would be exceeded, or upon
 * shutdown of the Executor.
 *
 * <p>In the absence of other alternatives, the method may throw
 * an unchecked {@link RejectedExecutionException}, which will be
 * propagated to the caller of {@code execute}.
 *
 * @param r the runnable task requested to be executed
 * @param executor the executor attempting to execute this task
 * @throws RejectedExecutionException if there is no remedy
 */
void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
```

其中，线程池默认实现了四种策略：

* `AbortPolicy` - 抛出异常，拒绝执行任务
* `DiscardPolicy` - 丢弃请求
* `DiscardOldestPolicy` - 丢弃最老的未处理请求 (工作队列队头) 并重试
* `CallerRunsPolicy` - 直接由调用者线程执行任务

这几种策略的实现方式都还挺简单的：

```java
/* Predefined RejectedExecutionHandlers */

/**
 * A handler for rejected tasks that runs the rejected task
 * directly in the calling thread of the {@code execute} method,
 * unless the executor has been shut down, in which case the task
 * is discarded.
 */
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code CallerRunsPolicy}.
     */
    public CallerRunsPolicy() { }

    /**
     * Executes task r in the caller's thread, unless the executor
     * has been shut down, in which case the task is discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}

/**
 * A handler for rejected tasks that throws a
 * {@code RejectedExecutionException}.
 */
public static class AbortPolicy implements RejectedExecutionHandler {
    /**
     * Creates an {@code AbortPolicy}.
     */
    public AbortPolicy() { }

    /**
     * Always throws RejectedExecutionException.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     * @throws RejectedExecutionException always
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                                " rejected from " +
                                                e.toString());
    }
}

/**
 * A handler for rejected tasks that silently discards the
 * rejected task.
 */
public static class DiscardPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code DiscardPolicy}.
     */
    public DiscardPolicy() { }

    /**
     * Does nothing, which has the effect of discarding task r.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}

/**
 * A handler for rejected tasks that discards the oldest unhandled
 * request and then retries {@code execute}, unless the executor
 * is shut down, in which case the task is discarded.
 */
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    /**
     * Creates a {@code DiscardOldestPolicy} for the given executor.
     */
    public DiscardOldestPolicy() { }

    /**
     * Obtains and ignores the next task that the executor
     * would otherwise execute, if one is immediately available,
     * and then retries execution of task r, unless the executor
     * is shut down, in which case task r is instead discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```

---

## References

[知乎专栏 - 美团技术团队 - Java 线程池实现原理及其在美团业务中的实践](https://zhuanlan.zhihu.com/p/123328822)

---

