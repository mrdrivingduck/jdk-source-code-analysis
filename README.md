# jdk-source-code-analysis

☕ Notes of reading JDK 8 source code.

Created by : Mr Dk.

2019 / 11 / 03 @Nanjing, P.R.China

---

Analyze the implementation details of JDK 8.

## Containers

| Link                                                         | Class                       | Abstract Class                 | Interface                |
| ------------------------------------------------------------ | --------------------------- | ------------------------------ | ------------------------ |
| [link](Interface%20%2d%20java.util.Collection.md)            |                             |                                | `java.util.Collection`   |
| [link](Abstract%20Class%20%2d%20java.util.AbstractCollection.md) |                             | `java.util.AbstractCollection` |                          |
| [link](Abstract%20Class%20%2d%20java.util.AbstractList.md)   |                             | `java.util.AbstractList`       |                          |
| [link](Class%20%2d%20java.util.ArrayList.md)                 | `java.util.ArrayList`       |                                |                          |
| [link](Class%20%2d%20java.util.LinkedList.md)                | `java.util.LinkedList`      |                                |                          |
| [link](Interface%20%2d%20java.util.Queue.md)                 |                             |                                | `java.util.Queue`        |
| [link](Abstract%20Class%20%2d%20java.util.AbstractQueue.md)  |                             | `java.util.AbstractQueue`      |                          |
| [link](Interface%20%2d%20java.util.Deque.md)                 |                             |                                | `java.util.Deque`        |
| [link](Class%20%2d%20java.util.PriorityQueue.md)             | `java.util.PriorityQueue`   |                                |                          |
| [link](Interface%20%2d%20java.util.Iterator.md)              |                             |                                | `java.util.Iterator`     |
| [link](Interface%20%2d%20java.util.ListIterator.md)          |                             |                                | `java.util.ListIterator` |
| [link](Interface%20%2d%20java.util.Map.md)                   |                             |                                | `java.util.Map`          |
| [link](Abstract%20Class%20%2d%20java.util.AbstractMap.md)    |                             | `java.util.AbstractMap`        |                          |
| [link](Interface%20%2d%20java.util.SortedMap.md)             |                             |                                | `java.util.SortedMap`    |
| [link](Interface%20%2d%20java.util.NavigableMap.md)          |                             |                                | `java.util.NavigableMap` |
| [link](Class%20%2d%20java.util.TreeMap.md)                   | `java.util.TreeMap`         |                                |                          |
| [link](Class%20%2d%20java.util.HashMap.md)                   | `java.util.HashMap`         |                                |                          |
| [link](Class%20%2d%20java.util.LinkedHashMap.md)             | `java.util.LinkedHashMap`   |                                |                          |
| [link](Class%20%2d%20java.util.IdentityHashMap.md)           | `java.util.IdentityHashMap` |                                |                          |
| [link](Interface%20%2d%20java.util.Set.md)                   |                             |                                | `java.util.Set`          |
| [link](Abstract%20Class%20%2d%20java.util.AbstractSet.md)    |                             | `java.util.AbstractSet`        |                          |
| [link](Interface%20%2d%20java.util.SortedSet.md)             |                             |                                | `java.util.SortedSet`    |
| [link](Interface%20%2d%20java.util.NavigableSet.md)          |                             |                                | `java.util.NavigableSet` |
| [link](Class%20%2d%20java.util.TreeSet.md)                   | `java.util.TreeSet`         |                                |                          |
| [link](Class%20%2d%20java.util.HashSet.md)                   | `java.util.HashSet`         |                                |                          |
| [link](Class%20%2d%20java.util.LinkedHashSet.md)             | `java.util.LinkedHashSet`   |                                |                          |

## Concurrent

| Link                                                         | Class                                                 | Abstract Class                                           | Interface                                  |
| ------------------------------------------------------------ | ----------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------ |
| [link](Interface%20%2d%20java.util.concurrent.locks.Lock.md) |                                                       |                                                          | `java.util.concurrent.locks.Lock`          |
| [link](Abstract%20Class%20%2d%20java.util.concurrent.locks.AbstractOwnableSynchronizer.md) |                                                       | `java.util.concurrent.locks.AbstractOwnableSynchronizer` |                                            |
| [link](Abstract%20Class%20%2d%20java.util.concurrent.locks.AbstractQueuedSynchronizer.md) |                                                       | `java.util.concurrent.locks.AbstractQueuedSynchronizer`  |                                            |
| [link](Class%20%2d%20java.util.concurrent.locks.ReentrantLock.md) | `java.util.concurrent.locks.ReentrantLock`            |                                                          |                                            |
| [link](Interface%20%2d%20java.util.concurrent.locks.ReadWriteLock.md) |                                                       |                                                          | `java.util.concurrent.locks.ReadWriteLock` |
| [link](Class%20%2d%20java.util.concurrent.locks.ReentrantReadWriteLock.md) | `java.util.concurrent.locks.ReentrantReadWriteLock`   |                                                          |                                            |
| [link](Class%20%2d%20java.util.concurrent.atomic.AtomicInteger.md) | `java.util.concurrent.atomic.AtomicInteger`           |                                                          |                                            |
| [link](Class%20%2d%20java.util.concurrent.atomic.AtomicIntegerArray.md) | `java.util.concurrent.atomic.AtomicIntegerArray`      |                                                          |                                            |
| [link](Abstract%20Class%20%2d%20java.util.concurrent.atomic.AtomicIntegerFieldUpdater.md) |                                                       | `java.util.concurrent.atomic.AtomicIntegerFieldUpdater`  |                                            |
| [link](Class%20%2d%20java.util.concurrent.atomic.AtomicReference.md) | `java.util.concurrent.atomic.AtomicReference`         |                                                          |                                            |
| [link](Class%20%2d%20java.util.concurrent.atomic.AtomicStampedReference.md) | `java.util.concurrent.atomic.AtomicStampedReference`  |                                                          |                                            |
| [link](Class%20%2d%20java.util.concurrent.atomic.AtomicStampedReference.md) | `java.util.concurrent.atomic.AtomicMarkableReference` |                                                          |                                            |

## Java Lang Class

| Link                                                         | Class               | Abstract Class                    | Interface |      |
| ------------------------------------------------------------ | ------------------- | --------------------------------- | --------- | ---- |
| [link](Class%20%2d%20java.lang.Integer.md)                   | `java.lang.Integer` |                                   |           |      |
| [link](Class%20%2d%20java.lang.String.md)                    | `java.lang.String`  |                                   |           |      |
| [link](Abstract%20Class%20%2d%20java.lang.AbstractStringBuilder.md) |                     | `java.lang.AbstractStringBuilder` |           |      |

---

## License

Copyright © 2019-2020, Jingtang Zhang. ([MIT License](LICENSE))

---

