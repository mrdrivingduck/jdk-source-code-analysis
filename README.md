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
| `java.util.Collection`   | [link](Interface%20%2d%20java.util.Collection.md)   |
| `java.util.Queue`        | [link](Interface%20%2d%20java.util.Queue.md)        |
| `java.util.Deque`        | [link](Interface%20%2d%20java.util.Deque.md)        |
| `java.util.Iterator`     | [link](Interface%20%2d%20java.util.Iterator.md)     |
| `java.util.ListIterator` | [link](Interface%20%2d%20java.util.ListIterator.md) |
| `java.util.Map`          | [link](Interface%20%2d%20java.util.Map.md)          |
| `java.util.SortedMap`    | [link](Interface%20%2d%20java.util.SortedMap.md)    |
| `java.util.NavigableMap` | [link](Interface%20%2d%20java.util.NavigableMap.md) |
| `java.util.Set`          | [link](Interface%20%2d%20java.util.Set.md)          |
| `java.util.SortedSet`    | [link](Interface%20%2d%20java.util.SortedSet.md)    |
| `java.util.NavigableSet` | [link](Interface%20%2d%20java.util.NavigableSet.md) |

### Abstract Classes

| Class                          | Link                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `java.util.AbstractCollection` | [link](Abstract%20Class%20%2d%20java.util.AbstractCollection.md) |
| `java.util.AbstractList`       | [link](Abstract%20Class%20%2d%20java.util.AbstractList.md)   |
| `java.util.AbstractQueue`      | [link](Abstract%20Class%20%2d%20java.util.AbstractQueue.md)  |
| `java.util.AbstractMap`        | [link](Abstract%20Class%20%2d%20java.util.AbstractMap.md)    |
| `java.util.AbstractSet`        | [link](Abstract%20Class%20%2d%20java.util.AbstractSet.md)    |

### Classes

| Class                       | Link                                               |
| --------------------------- | -------------------------------------------------- |
| `java.util.ArrayList`       | [link](Class%20%2d%20java.util.ArrayList.md)       |
| `java.util.LinkedList`      | [link](Class%20%2d%20java.util.LinkedList.md)      |
| `java.util.PriorityQueue`   | [link](Class%20%2d%20java.util.PriorityQueue.md)   |
| `java.util.TreeMap`         | [link](Class%20%2d%20java.util.TreeMap.md)         |
| `java.util.HashMap`         | [link](Class%20%2d%20java.util.HashMap.md)         |
| `java.util.LinkedHashMap`   | [link](Class%20%2d%20java.util.LinkedHashMap.md)   |
| `java.util.IdentityHashMap` | [link](Class%20%2d%20java.util.IdentityHashMap.md) |
| `java.util.TreeSet`         | [link](Class%20%2d%20java.util.TreeSet.md)         |
| `java.util.HashSet`         | [link](Class%20%2d%20java.util.HashSet.md)         |
| `java.util.LinkedHashSet`   | [link](Class%20%2d%20java.util.LinkedHashSet.md)   |

## Concurrent

### Interfaces

| Class                                      | Link                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| `java.util.concurrent.locks.Lock`          | [link](Interface%20%2d%20java.util.concurrent.locks.Lock.md) |
| `java.util.concurrent.locks.ReadWriteLock` | [link](Interface%20%2d%20java.util.concurrent.locks.ReadWriteLock.md) |
| `java.util.concurrent.BlockingQueue`       | [link](Interface%20%2d%20java.util.concurrent.BlockingQueue.md) |
| `java.util.concurrent.TransferQueue`       | [link](Interface%20%2d%20java.util.concurrent.TransferQueue.md) |

### Abstract Classes

| Class                                                    | Link                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `java.util.concurrent.locks.AbstractOwnableSynchronizer` | [link](Abstract%20Class%20%2d%20java.util.concurrent.locks.AbstractOwnableSynchronizer.md) |
| `java.util.concurrent.locks.AbstractQueuedSynchronizer`  | [link](Abstract%20Class%20%2d%20java.util.concurrent.locks.AbstractQueuedSynchronizer.md) |
| `java.util.concurrent.atomic.AtomicIntegerFieldUpdater`  | [link](Abstract%20Class%20%2d%20java.util.concurrent.atomic.AtomicIntegerFieldUpdater.md) |

### Classes

| Class                                                 | Link                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| `java.util.concurrent.locks.ReentrantLock`            | [link](Class%20%2d%20java.util.concurrent.locks.ReentrantLock.md) |
| `java.util.concurrent.locks.ReentrantReadWriteLock`   | [link](Class%20%2d%20java.util.concurrent.locks.ReentrantReadWriteLock.md) |
| `java.util.concurrent.atomic.AtomicInteger`           | [link](Class%20%2d%20java.util.concurrent.atomic.AtomicInteger.md) |
| `java.util.concurrent.atomic.AtomicIntegerArray`      | [link](Class%20%2d%20java.util.concurrent.atomic.AtomicIntegerArray.md) |
| `java.util.concurrent.atomic.AtomicReference`         | [link](Class%20%2d%20java.util.concurrent.atomic.AtomicReference.md) |
| `java.util.concurrent.atomic.AtomicStampedReference`  | [link](Class%20%2d%20java.util.concurrent.atomic.AtomicStampedReference.md) |
| `java.util.concurrent.atomic.AtomicMarkableReference` | [link](Class%20%2d%20java.util.concurrent.atomic.AtomicStampedReference.md) |
| `java.util.concurrent.ConcurrentHashMap`              | [link](Class%20%2d%20java.util.concurrent.ConcurrentHashMap.md) |
| `java.util.concurrent.LinkedBlockingQueue`            | [link](Class%20%2d%20java.util.concurrent.LinkedBlockingQueue.md) |
| `java.util.concurrent.LinkedBlockingDeque`            | / |
| `java.util.concurrent.ArrayBlockingQueue`             | [link](Class%20%2d%20java.util.concurrent.ArrayBlockingQueue.md) |
| `java.util.concurrent.PriorityBlockingQueue`          | / |
| `java.util.concurrent.LinkedTransferQueue`            | [link](Class%20%2d%20java.util.concurrent.LinkedTransferQueue.md) |
| `java.util.concurrent.SynchronousQueue`               | [link](Class%20%2d%20java.util.concurrent.SynchronousQueue.md) |
| `java.util.concurrent.DelayQueue         `            | [link](Class%20%2d%20java.util.concurrent.DelayQueue.md) |
| `java.util.concurrent.ConcurrentLinkedQueue`          | [link](Class%20%2d%20java.util.concurrent.ConcurrentLinkedQueue.md) |
| `java.util.concurrent.ConcurrentLinkedDeque`          | / |
| `java.util.concurrent.ThreadPoolExecutor`             | [link](Class%20%2d%20java.util.concurrent.ThreadPoolExecutor.md) |

## Java Lang Class

### Abstract Classes

| Class                             | Link                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| `java.lang.AbstractStringBuilder` | [link](Abstract%20Class%20%2d%20java.lang.AbstractStringBuilder.md) |

### Classes

| Class               | Link                                       |
| ------------------- | ------------------------------------------ |
| `java.lang.Integer` | [link](Class%20%2d%20java.lang.Integer.md) |
| `java.lang.String`  | [link](Class%20%2d%20java.lang.String.md)  |

## I/O

### Interfaces

| Class               | Link                                           |
| ------------------- | ---------------------------------------------- |
| `java.io.Closeable` | [link](Interface%20%2d%20java.io.Closeable.md) |

### Abstract Classes

| Class                 | Link                                                    |
| --------------------- | ------------------------------------------------------- |
| `java.io.InputStream` | [link](Abstract%20Class%20%2d%20java.io.InputStream.md) |
| `java.io.Reader`      | [link](Abstract%20Class%20%2d%20java.io.Reader.md) |

### Classes

| Class                     | Link                                             |
| ------------------------- | ------------------------------------------------ |
| `java.io.FileInputStream` | [link](Class%20%2d%20java.io.FileInputStream.md) |
| `java.io.FilterInputStream` | [link](Class%20%2d%20java.io.FilterInputStream.md) |
| `java.io.DataInputStream` | [link](Class%20%2d%20java.io.DataInputStream.md) |
| `java.io.BufferedInputStream` | [link](Class%20%2d%20java.io.BufferedInputStream.md) |
| `java.io.ByteArrayInputStream` | [link](Class%20%2d%20java.io.ByteArrayInputStream.md) |
| `java.io.PushbackInputStream` | [link](Class%20%2d%20java.io.PushbackInputStream.md) |
| `java.io.SequenceInputStream` | [link](Class%20%2d%20java.io.SequenceInputStream.md) |
| `java.io.InputStreamReader` | [link](Class%20%2d%20java.io.InputStreamReader.md) |
| `java.io.FileReader` | [link](Class%20%2d%20java.io.FileReader.md) |
| `java.io.BufferedReader` | [link](Class%20%2d%20java.io.BufferedReader.md) |

---

## License

Copyright © 2019-2020, Jingtang Zhang. ([MIT License](LICENSE))

---

