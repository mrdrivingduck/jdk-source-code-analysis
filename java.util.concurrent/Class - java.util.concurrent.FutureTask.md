# Class - java.util.concurrent.FutureTask

Created by : Mr Dk.

2021 / 02 / 16 🧧 18:37

Ningbo, Zhejiang, China

---

## Definition

该类表示一个 **可被取消的异步计算**，是 `Future` 接口的默认实现，提供了如下的基本功能实现：

* 开始 / 取消计算
* 查询计算是否已经完成
* 获取计算结果

计算结果只有在任务完成之后才能被获取。计算一旦完成，将无法被重新启动或取消，除非计算是被 `runAndReset()` 调用的。

`FutureTask` 可以包装 `Callable` 或 `Runnable` 对象，交由执行器执行，因为 `FutureTask` 本身实现了 `Runnable` 接口。

```java
/**
 * A cancellable asynchronous computation.  This class provides a base
 * implementation of {@link Future}, with methods to start and cancel
 * a computation, query to see if the computation is complete, and
 * retrieve the result of the computation.  The result can only be
 * retrieved when the computation has completed; the {@code get}
 * methods will block if the computation has not yet completed.  Once
 * the computation has completed, the computation cannot be restarted
 * or cancelled (unless the computation is invoked using
 * {@link #runAndReset}).
 *
 * <p>A {@code FutureTask} can be used to wrap a {@link Callable} or
 * {@link Runnable} object.  Because {@code FutureTask} implements
 * {@code Runnable}, a {@code FutureTask} can be submitted to an
 * {@link Executor} for execution.
 *
 * <p>In addition to serving as a standalone class, this class provides
 * {@code protected} functionality that may be useful when creating
 * customized task classes.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <V> The result type returned by this FutureTask's {@code get} methods
 */
public class FutureTask<V> implements RunnableFuture<V> {

}
```

## Running State

定义了任务的运行状态，初始为 `NEW`。只有在 `set()` / `setException()` / `cancel()` 函数中才会将状态转移为 **终结**。

```java
/**
 * The run state of this task, initially NEW.  The run state
 * transitions to a terminal state only in methods set,
 * setException, and cancel.  During completion, state may take on
 * transient values of COMPLETING (while outcome is being set) or
 * INTERRUPTING (only while interrupting the runner to satisfy a
 * cancel(true)). Transitions from these intermediate to final
 * states use cheaper ordered/lazy writes because values are unique
 * and cannot be further modified.
 *
 * Possible state transitions:
 * NEW -> COMPLETING -> NORMAL
 * NEW -> COMPLETING -> EXCEPTIONAL
 * NEW -> CANCELLED
 * NEW -> INTERRUPTING -> INTERRUPTED
 */
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

## Fields

维护的变量包含：

* `callable` - 被执行的任务
* `outcome` - 任务执行完毕后的返回结果
* `runner` - 执行任务的线程
* `waiters` - 等待任务执行完毕的线程

```java
/** The underlying callable; nulled out after running */
private Callable<V> callable;
/** The result to return or exception to throw from get() */
private Object outcome; // non-volatile, protected by state reads/writes
/** The thread running the callable; CASed during run() */
private volatile Thread runner;
/** Treiber stack of waiting threads */
private volatile WaitNode waiters;
```

等待任务执行完毕的线程实际上是一个链表，链表结点定义如下：

```java
/**
 * Simple linked list nodes to record waiting threads in a Treiber
 * stack.  See other classes such as Phaser and SynchronousQueue
 * for more detailed explanation.
 */
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```

## Constructor

将想要执行的任务设置为成员变量 `callable`，并将任务的状态设置为 `NEW`。

```java
/**
 * Creates a {@code FutureTask} that will, upon running, execute the
 * given {@code Callable}.
 *
 * @param  callable the callable task
 * @throws NullPointerException if the callable is null
 */
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

/**
 * Creates a {@code FutureTask} that will, upon running, execute the
 * given {@code Runnable}, and arrange that {@code get} will return the
 * given result on successful completion.
 *
 * @param runnable the runnable task
 * @param result the result to return on successful completion. If
 * you don't need a particular result, consider using
 * constructions of the form:
 * {@code Future<?> f = new FutureTask<Void>(runnable, null)}
 * @throws NullPointerException if the runnable is null
 */
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

## Report

返回任务当前的状态。

```java
/**
 * Returns result or throws exception for completed task.
 *
 * @param s completed state value
 */
@SuppressWarnings("unchecked")
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}

public boolean isCancelled() {
    return state >= CANCELLED;
}

public boolean isDone() {
    return state != NEW;
}
```

## Finish Completion

多个操作的公用方法，用于结束任务的运行。主要工作包含：

* 依次对等待任务结束的线程发送信号，并移出等待链表
* 调用 `done()` 处理任务完成之后的审计及回调
* 将 `callable` 置空

```java
/**
 * Removes and signals all waiting threads, invokes done(), and
 * nulls out callable.
 */
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }

    done();

    callable = null;        // to reduce footprint
}
```

`done()` 函数在本类中没有任何逻辑，可以由子类重写。

```java
/**
 * Protected method invoked when this task transitions to state
 * {@code isDone} (whether normally or via cancellation). The
 * default implementation does nothing.  Subclasses may override
 * this method to invoke completion callbacks or perform
 * bookkeeping. Note that you can query status inside the
 * implementation of this method to determine whether this task
 * has been cancelled.
 */
protected void done() { }
```

## Cancel

取消当前任务的执行。如果任务已经开始执行，那么由参数 `mayInterruptIfRunning` 来决定是否中断任务的执行。通过 CAS 操作将任务的状态由 `NEW` 置为 `INTERRUPTING` 或 `CANCELLED`，然后对正在执行任务的线程发送中断。最终，终结任务的运行。

```java
public boolean cancel(boolean mayInterruptIfRunning) {
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
                                   mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {    // in case call to interrupt throws exception
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { // final state
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        finishCompletion();
    }
    return true;
}
```

## Await Done

等待任务完成，或因为中断而退出，或超时。在一个死循环中不断测试任务状态，并不断清除正在等待的线程。

```java
/**
 * Awaits completion or aborts on interrupt or timeout.
 *
 * @param timed true if use timed waits
 * @param nanos time to wait, if timed
 * @return state upon completion
 */
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}
```

## Get

等待任务执行完成，并返回任务的执行结果。可选是否超时等待。

```java
/**
 * @throws CancellationException {@inheritDoc}
 */
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}

/**
 * @throws CancellationException {@inheritDoc}
 */
public V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();
    int s = state;
    if (s <= COMPLETING &&
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)
        throw new TimeoutException();
    return report(s);
}
```

## Set

设置任务成功执行后的执行结果。该函数只会被 `run()` 在任务成功完成时调用。通过 CAS 操作将任务执行状态设置为 `COMPLETING`，在设置结束后再将任务设置为 `NORMAL` (这里不再需要 CAS，因为没有竞争)。最终结束任务的执行。

注意，只有在任务状态为 `NEW` 时，CAS 才可以成功。因此任务被取消后，该函数无效。

```java
/**
 * Sets the result of this future to the given value unless
 * this future has already been set or has been cancelled.
 *
 * <p>This method is invoked internally by the {@link #run} method
 * upon successful completion of the computation.
 *
 * @param v the value
 */
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}
```

设置任务执行异常后的执行结果，返回值是一个 `Throwable` 对象。CAS 操作会将任务状态设为值 `COMPLETING`，在结果设置完毕后，再将任务状态设置为 `EXCEPTIONAL`。最终结束任务的执行

```java
/**
 * Causes this future to report an {@link ExecutionException}
 * with the given throwable as its cause, unless this future has
 * already been set or has been cancelled.
 *
 * <p>This method is invoked internally by the {@link #run} method
 * upon failure of the computation.
 *
 * @param t the cause of failure
 */
protected void setException(Throwable t) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = t;
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        finishCompletion();
    }
}
```

## Run

执行任务。确认任务状态允许执行后，调用任务的 `call()` 函数，并在任务完成后设置任务运行的成功 / 失败结果。最终，要将执行任务的线程置空，防止多个线程并发调用 `run()`。

```java
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

```java
/**
 * Ensures that any interrupt from a possible cancel(true) is only
 * delivered to a task while in run or runAndReset.
 */
private void handlePossibleCancellationInterrupt(int s) {
    // It is possible for our interrupter to stall before getting a
    // chance to interrupt us.  Let's spin-wait patiently.
    if (s == INTERRUPTING)
        while (state == INTERRUPTING)
            Thread.yield(); // wait out pending interrupt

    // assert state == INTERRUPTED;

    // We want to clear any interrupt we may have received from
    // cancel(true).  However, it is permissible to use interrupts
    // as an independent mechanism for a task to communicate with
    // its caller, and there is no way to clear only the
    // cancellation interrupt.
    //
    // Thread.interrupted();
}
```

## Run And Reset

执行任务，但不设置任务的执行结果，然后恢复该对象到初始状态，使任务能够再次被执行。当异步计算引发异常或被取消时，不会进行 reset 操作。这个函数被设计用于天然就会被调用多次的任务。返回值含义为是否成功进行了 run 和 reset。

```java
/**
 * Executes the computation without setting its result, and then
 * resets this future to initial state, failing to do so if the
 * computation encounters an exception or is cancelled.  This is
 * designed for use with tasks that intrinsically execute more
 * than once.
 *
 * @return {@code true} if successfully run and reset
 */
protected boolean runAndReset() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return false;
    boolean ran = false;
    int s = state;
    try {
        Callable<V> c = callable;
        if (c != null && s == NEW) {
            try {
                c.call(); // don't set result
                ran = true;
            } catch (Throwable ex) {
                setException(ex);
            }
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
    return ran && s == NEW;
}
```

---

