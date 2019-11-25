# jdk-source-code-analysis

☕ Notes of reading JDK 8 source code.

Created by : Mr Dk.

2019 / 11 / 03 @Nanjing, P.R.China

---

I should know some implementation details of _JDK_ .

---

## Containers

* Interface - `java.util.Collection`
    * Abstract Class - `java.util.AbstractCollection`
    * Abstract Class - `java.util.AbstractList`
        * Class - `java.util.ArrayList` 
        * Class - `java.util.LinkedList`
* Interface - `java.util.Queue`
    * Abstract Class - `java.util.AbstractQueue`
    * Interface - `java.util.Deque`
        * Class - `java.util.PriorityQueue`
* Interface - `java.util.Iterator`
    * Interface - `java.util.ListIterator`
* Interface - `java.util.Map`
    * Abstract Class - `java.util.AbstractMap`
    * Interface - `java.util.SortedMap`
    * Interface - `java.util.NavigableMap`
        * Class - `java.util.TreeMap`
    * Class - `java.util.HashMap`
        * Class - `java.util.LinkedHashMap`
* Interface - `java.util.Set`
    * Abstract Class - `java.util.AbstractSet`
    * Interface - `java.util.SortedSet`
    * Interface - `java.util.NavigableSet`
        * Class - `java.util.TreeSet`
    * Class - `java.util.HashSet`
        * Class - `java.util.LinkedHashSet`
    * Class - `java.util.IdentityHashMap`

---

