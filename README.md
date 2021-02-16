# jdk-source-code-analysis

☕ Notes of reading JDK 8 source code.

Created by : Mr Dk.

2019 / 11 / 03 @Nanjing, P.R.China

---

Analyze the implementation details of JDK 8.

## Containers

### Interfaces

| Class                    | Link                                                |
| ------------------------ | --------------------------------------------------- |
| `java.util.Collection`   | [link](java.util/Interface%20%2d%20java.util.Collection.md)   |
| `java.util.Queue`        | [link](java.util/Interface%20%2d%20java.util.Queue.md)        |
| `java.util.Deque`        | [link](java.util/Interface%20%2d%20java.util.Deque.md)        |
| `java.util.Iterator`     | [link](java.util/Interface%20%2d%20java.util.Iterator.md)     |
| `java.util.ListIterator` | [link](java.util/Interface%20%2d%20java.util.ListIterator.md) |
| `java.util.Map`          | [link](java.util/Interface%20%2d%20java.util.Map.md)          |
| `java.util.SortedMap`    | [link](java.util/Interface%20%2d%20java.util.SortedMap.md)    |
| `java.util.NavigableMap` | [link](java.util/Interface%20%2d%20java.util.NavigableMap.md) |
| `java.util.Set`          | [link](java.util/Interface%20%2d%20java.util.Set.md)          |
| `java.util.SortedSet`    | [link](java.util/Interface%20%2d%20java.util.SortedSet.md)    |
| `java.util.NavigableSet` | [link](java.util/Interface%20%2d%20java.util.NavigableSet.md) |

### Abstract Classes

| Class                          | Link                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `java.util.AbstractCollection` | [link](java.util/Abstract%20Class%20%2d%20java.util.AbstractCollection.md) |
| `java.util.AbstractList`       | [link](java.util/Abstract%20Class%20%2d%20java.util.AbstractList.md)   |
| `java.util.AbstractQueue`      | [link](java.util/Abstract%20Class%20%2d%20java.util.AbstractQueue.md)  |
| `java.util.AbstractMap`        | [link](java.util/Abstract%20Class%20%2d%20java.util.AbstractMap.md)    |
| `java.util.AbstractSet`        | [link](java.util/Abstract%20Class%20%2d%20java.util.AbstractSet.md)    |

### Classes

| Class                       | Link                                               |
| --------------------------- | -------------------------------------------------- |
| `java.util.ArrayList`       | [link](java.util/Class%20%2d%20java.util.ArrayList.md)       |
| `java.util.LinkedList`      | [link](java.util/Class%20%2d%20java.util.LinkedList.md)      |
| `java.util.PriorityQueue`   | [link](java.util/Class%20%2d%20java.util.PriorityQueue.md)   |
| `java.util.TreeMap`         | [link](java.util/Class%20%2d%20java.util.TreeMap.md)         |
| `java.util.HashMap`         | [link](java.util/Class%20%2d%20java.util.HashMap.md)         |
| `java.util.LinkedHashMap`   | [link](java.util/Class%20%2d%20java.util.LinkedHashMap.md)   |
| `java.util.IdentityHashMap` | [link](java.util/Class%20%2d%20java.util.IdentityHashMap.md) |
| `java.util.TreeSet`         | [link](java.util/Class%20%2d%20java.util.TreeSet.md)         |
| `java.util.HashSet`         | [link](java.util/Class%20%2d%20java.util.HashSet.md)         |
| `java.util.LinkedHashSet`   | [link](java.util/Class%20%2d%20java.util.LinkedHashSet.md)   |

## Concurrent

### Interfaces

| Class                                           | Link                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `java.util.concurrent.locks.Lock`               | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.locks.Lock.md) |
| `java.util.concurrent.locks.ReadWriteLock`      | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.locks.ReadWriteLock.md) |
| `java.util.concurrent.BlockingQueue`            | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.BlockingQueue.md) |
| `java.util.concurrent.TransferQueue`            | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.TransferQueue.md) |
| `java.util.concurrent.Future`                   | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.Future.md) |
| `java.util.concurrent.Executor`                 | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.Executor.md) |
| `java.util.concurrent.ExecutorService`          | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.ExecutorService.md) |
| `java.util.concurrent.ScheduledExecutorService` | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.ScheduledExecutorService.md) |
| `java.util.concurrent.CompletionService`        | [link](java.util.concurrent/Interface%20%2d%20java.util.concurrent.CompletionService.md) |

### Abstract Classes

| Class                                                    | Link                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `java.util.concurrent.locks.AbstractOwnableSynchronizer` | [link](java.util.concurrent/Abstract%20Class%20%2d%20java.util.concurrent.locks.AbstractOwnableSynchronizer.md) |
| `java.util.concurrent.locks.AbstractQueuedSynchronizer`  | [link](java.util.concurrent/Abstract%20Class%20%2d%20java.util.concurrent.locks.AbstractQueuedSynchronizer.md) |
| `java.util.concurrent.atomic.AtomicIntegerFieldUpdater`  | [link](java.util.concurrent/Abstract%20Class%20%2d%20java.util.concurrent.atomic.AtomicIntegerFieldUpdater.md) |
| `java.util.concurrent.atomic.AbstractExecutorService`    | [link](java.util.concurrent/Abstract%20Class%20%2d%20java.util.concurrent.atomic.AbstractExecutorService.md) |

### Classes

| Class                                                 | Link                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| `java.util.concurrent.locks.ReentrantLock`            | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.locks.ReentrantLock.md) |
| `java.util.concurrent.locks.ReentrantReadWriteLock`   | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.locks.ReentrantReadWriteLock.md) |
| `java.util.concurrent.atomic.AtomicInteger`           | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.atomic.AtomicInteger.md) |
| `java.util.concurrent.atomic.AtomicIntegerArray`      | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.atomic.AtomicIntegerArray.md) |
| `java.util.concurrent.atomic.AtomicReference`         | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.atomic.AtomicReference.md) |
| `java.util.concurrent.atomic.AtomicStampedReference`  | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.atomic.AtomicStampedReference.md) |
| `java.util.concurrent.atomic.AtomicMarkableReference` | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.atomic.AtomicStampedReference.md) |
| `java.util.concurrent.ConcurrentHashMap`              | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.ConcurrentHashMap.md) |
| `java.util.concurrent.LinkedBlockingQueue`            | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.LinkedBlockingQueue.md) |
| `java.util.concurrent.LinkedBlockingDeque`            | /                                                            |
| `java.util.concurrent.ArrayBlockingQueue`             | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.ArrayBlockingQueue.md) |
| `java.util.concurrent.PriorityBlockingQueue`          | /                                                            |
| `java.util.concurrent.LinkedTransferQueue`            | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.LinkedTransferQueue.md) |
| `java.util.concurrent.SynchronousQueue`               | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.SynchronousQueue.md) |
| `java.util.concurrent.DelayQueue`                     | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.DelayQueue.md) |
| `java.util.concurrent.ConcurrentLinkedQueue`          | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.ConcurrentLinkedQueue.md) |
| `java.util.concurrent.ConcurrentLinkedDeque`          | /                                                            |
| `java.util.concurrent.ThreadPoolExecutor`             | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.ThreadPoolExecutor.md) |
| `java.util.concurrent.FutureTask`                     | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.FutureTask.md) |
| `java.util.concurrent.ExecutorCompletionService`      | [link](java.util.concurrent/Class%20%2d%20java.util.concurrent.ExecutorCompletionService.md) |

## Java Language Class

### Abstract Classes

| Class                             | Link                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| `java.lang.AbstractStringBuilder` | [link](java.lang/Abstract%20Class%20%2d%20java.lang.AbstractStringBuilder.md) |

### Classes

| Class                    | Link                                            |
| ------------------------ | ----------------------------------------------- |
| `java.lang.Integer`      | [link](java.lang/Class%20%2d%20java.lang.Integer.md)      |
| `java.lang.String`       | [link](java.lang/Class%20%2d%20java.lang.String.md)       |
| `java.lang.ThreadLocal`  | [link](java.lang/Class%20%2d%20java.lang.ThreadLocal.md)  |

## I/O

### Interfaces

| Class               | Link                                           |
| ------------------- | ---------------------------------------------- |
| `java.io.Closeable` | [link](java.io/Interface%20%2d%20java.io.Closeable.md) |

### Abstract Classes

| Class                  | Link                                                     |
| ---------------------  | -------------------------------------------------------  |
| `java.io.InputStream`  | [link](java.io/Abstract%20Class%20%2d%20java.io.InputStream.md)  |
| `java.io.OutputStream` | [link](java.io/Abstract%20Class%20%2d%20java.io.OutputStream.md) |
| `java.io.Reader`       | [link](java.io/Abstract%20Class%20%2d%20java.io.Reader.md)       |

### Classes

| Class                                | Link                                             |
| ------------------------------------ | ------------------------------------------------ |
| `java.io.FileInputStream`            | [link](java.io/Class%20%2d%20java.io.FileInputStream.md) |
| `java.io.FileOutputStream`           | [link](java.io/Class%20%2d%20java.io.FileOutputStream.md) |
| `java.io.FilterInputStream`          | [link](java.io/Class%20%2d%20java.io.FilterInputStream.md) |
| `java.io.FilterOutputStream`         | [link](java.io/Class%20%2d%20java.io.FilterOutputStream.md) |
| `java.io.DataInputStream`            | [link](java.io/Class%20%2d%20java.io.DataInputStream.md) |
| `java.io.DataOutputStream`           | [link](java.io/Class%20%2d%20java.io.DataOutputStream.md) |
| `java.io.BufferedInputStream`        | [link](java.io/Class%20%2d%20java.io.BufferedInputStream.md) |
| `java.io.BufferedOutputStream`       | [link](java.io/Class%20%2d%20java.io.BufferedOutputStream.md) |
| `java.io.ByteArrayInputStream`       | [link](java.io/Class%20%2d%20java.io.ByteArrayInputStream.md) |
| `java.io.ByteArrayOutputStream`      | [link](java.io/Class%20%2d%20java.io.ByteArrayOutputStream.md) |
| `java.io.PushbackInputStream`        | [link](java.io/Class%20%2d%20java.io.PushbackInputStream.md) |
| `java.io.SequenceInputStream`        | [link](java.io/Class%20%2d%20java.io.SequenceInputStream.md) |
| `java.io.PipedInputStream`           | [link](java.io/Class%20%2d%20java.io.PipedInputStream.md) |
| `java.io.PipedOutputStream`          | [link](java.io/Class%20%2d%20java.io.PipedOutputStream.md) |
| `java.io.InputStreamReader`          | [link](java.io/Class%20%2d%20java.io.InputStreamReader.md) |
| `java.io.FileReader`                 | [link](java.io/Class%20%2d%20java.io.FileReader.md) |
| `java.io.BufferedReader`             | [link](java.io/Class%20%2d%20java.io.BufferedReader.md) |

## NIO

### Abstract Classes

| Class                  | Link                                                     |
| ---------------------  | -------------------------------------------------------  |
| `java.nio.Buffer`  | [link](java.nio/Abstract%20Class%20%2d%20java.nio.Buffer.md)  |

---

## License

Copyright © 2019-2021, Jingtang Zhang. ([MIT License](LICENSE))

---

