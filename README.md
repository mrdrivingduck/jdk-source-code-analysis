# jdk-source-code-analysis

☕ Notes of reading JDK 8 source code.

Created by : Mr Dk.

2019 / 11 / 03 @Nanjing, P.R.China

---

Some implementation details of _JDK_ .

---

## Containers

| Interface                | Abstract Class                 | Class                       | Link                                                       |
| ------------------------ | ------------------------------ | --------------------------- | ---------------------------------------------------------- |
| `java.util.Collection`   |                                |                             | [link](./Interface - java.util.Collection.md)              |
|                          | `java.util.AbstractCollection` |                             | [link](./Abstract Class - java.util.AbstractCollection.md) |
|                          | `java.util.AbstractList`       |                             | [link](./Abstract Class - java.util.AbstractList.md)       |
|                          |                                | `java.util.ArrayList`       | [link](./Class - java.util.ArrayList.md)                   |
|                          |                                | `java.util.LinkedList`      | [link](./Class - java.util.LinkedList.md)                  |
| `java.util.Queue`        |                                |                             | [link](./Interface - java.util.Queue.md)                   |
|                          | `java.util.AbstractQueue`      |                             | [link](./Abstract Class - java.util.AbstractQueue.md)      |
| `java.util.Deque`        |                                |                             | [link](./Interface - java.util.Deque.md)                   |
|                          |                                | `java.util.PriorityQueue`   | [link](./Class - java.util.PriorityQueue.md)               |
| `java.util.Iterator`     |                                |                             | [link](./Interface - java.util.Iterator.md)                |
| `java.util.ListIterator` |                                |                             | [link](./Interface - java.util.ListIterator.md)            |
| `java.util.Map`          |                                |                             | [link](./Interface - java.util.Map.md)                     |
|                          | `java.util.AbstractMap`        |                             | [link](./Abstract Class - java.util.AbstractMap.md)        |
| `java.util.SortedMap`    |                                |                             | [link](./Interface - java.util.SortedMap.md)               |
| `java.util.NavigableMap` |                                |                             | [link](./Interface - java.util.NavigableMap.md)            |
|                          |                                | `java.util.TreeMap`         | [link](./Class - java.util.TreeMap.md)                     |
|                          |                                | `java.util.HashMap`         | [link](./Class - java.util.HashMap.md)                     |
|                          |                                | `java.util.LinkedHashMap`   | [link](./Class - java.util.LinkedHashMap.md)               |
|                          |                                | `java.util.IdentityHashMap` | [link](./Class - java.util.IdentityHashMap.md)             |
| `java.util.Set`          |                                |                             | [link](./Interface - java.util.Set.md)                     |
|                          | `java.util.AbstractSet`        |                             | [link](./Abstract Class - java.util.AbstractSet.md)        |
| `java.util.SortedSet`    |                                |                             | [link](./Interface - java.util.SortedSet.md)               |
| `java.util.NavigableSet` |                                |                             | [link](./Interface - java.util.NavigableSet.md)            |
|                          |                                | `java.util.TreeSet`         | [link](./Class - java.util.TreeSet.md)                     |
|                          |                                | `java.util.HashSet`         | [link](./Class - java.util.HashSet.md)                     |
|                          |                                | `java.util.LinkedHashSet`   | [link](./Class - java.util.LinkedHashSet.md)               |

---

## Concurrent

| Interface                         | Abstract Class | Class | Link                                                     |
| --------------------------------- | -------------- | ----- | -------------------------------------------------------- |
| `java.util.concurrent.locks.Lock` |                |       | [link](./Interface - java.util.concurrent.locks.Lock.md) |

---

## License

```
MIT License

Copyright (c) 2019 Mr Dk.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```