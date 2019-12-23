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
| `java.util.Collection`   |                                |                             | [link](Interface%20%2d%20java.util.Collection.md)              |
|                          | `java.util.AbstractCollection` |                             | [link](Abstract%20Class%20%2d%20java.util.AbstractCollection.md) |
|                          | `java.util.AbstractList`       |                             | [link](Abstract%20Class%20%2d%20java.util.AbstractList.md)       |
|                          |                                | `java.util.ArrayList`       | [link](Class%20%2d%20java.util.ArrayList.md)                   |
|                          |                                | `java.util.LinkedList`      | [link](Class%20%2d%20java.util.LinkedList.md)                  |
| `java.util.Queue`        |                                |                             | [link](Interface%20%2d%20java.util.Queue.md)                   |
|                          | `java.util.AbstractQueue`      |                             | [link](Abstract%20Class%20%2d%20java.util.AbstractQueue.md)      |
| `java.util.Deque`        |                                |                             | [link](Interface%20%2d%20java.util.Deque.md)                   |
|                          |                                | `java.util.PriorityQueue`   | [link](Class%20%2d%20java.util.PriorityQueue.md)               |
| `java.util.Iterator`     |                                |                             | [link](Interface%20%2d%20java.util.Iterator.md)                |
| `java.util.ListIterator` |                                |                             | [link](Interface%20%2d%20java.util.ListIterator.md)            |
| `java.util.Map`          |                                |                             | [link](Interface%20%2d%20java.util.Map.md)                     |
|                          | `java.util.AbstractMap`        |                             | [link](Abstract%20Class%20%2d%20java.util.AbstractMap.md)        |
| `java.util.SortedMap`    |                                |                             | [link](Interface%20%2d%20java.util.SortedMap.md)               |
| `java.util.NavigableMap` |                                |                             | [link](Interface%20%2d%20java.util.NavigableMap.md)            |
|                          |                                | `java.util.TreeMap`         | [link](Class%20%2d%20java.util.TreeMap.md)                     |
|                          |                                | `java.util.HashMap`         | [link](Class%20%2d%20java.util.HashMap.md)                     |
|                          |                                | `java.util.LinkedHashMap`   | [link](Class%20%2d%20java.util.LinkedHashMap.md)               |
|                          |                                | `java.util.IdentityHashMap` | [link](Class%20%2d%20java.util.IdentityHashMap.md)             |
| `java.util.Set`          |                                |                             | [link](Interface%20%2d%20java.util.Set.md)                     |
|                          | `java.util.AbstractSet`        |                             | [link](Abstract%20Class%20%2d%20java.util.AbstractSet.md)        |
| `java.util.SortedSet`    |                                |                             | [link](Interface%20%2d%20java.util.SortedSet.md)               |
| `java.util.NavigableSet` |                                |                             | [link](Interface%20%2d%20java.util.NavigableSet.md)            |
|                          |                                | `java.util.TreeSet`         | [link](Class%20%2d%20java.util.TreeSet.md)                     |
|                          |                                | `java.util.HashSet`         | [link](Class%20%2d%20java.util.HashSet.md)                     |
|                          |                                | `java.util.LinkedHashSet`   | [link](Class%20%2d%20java.util.LinkedHashSet.md)               |

---

## Concurrent

| Interface                         | Abstract Class | Class | Link                                                     |
| --------------------------------- | -------------- | ----- | -------------------------------------------------------- |
| `java.util.concurrent.locks.Lock` |                |       | [link](Interface%20%2d%20java.util.concurrent.locks.Lock.md) |

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