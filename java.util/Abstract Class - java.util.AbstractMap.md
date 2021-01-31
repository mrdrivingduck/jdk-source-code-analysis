# Abstract Class - java.util.AbstractMap

Created by : Mr Dk.

2019 / 11 / 13 11:53

Nanjing, Jiangsu, China

---

## Definition

```java
public abstract class AbstractMap<K,V> implements Map<K,V> {

}
```

Map 接口的骨架实现

* 为实现一个不可修改的 Map
    * 继承自该类的类需要实现 `entrySet()`
    * 返回的集合应当继承自 `AbstractSet`，但不能提供 `add()` 和 `remove()` 函数
* 为实现一个可修改的 Map
    * 必须实现 `put()` 函数
    * 迭代器必须实现 `remove()` 函数

```java
/**
 * This class provides a skeletal implementation of the <tt>Map</tt>
 * interface, to minimize the effort required to implement this interface.
 *
 * <p>To implement an unmodifiable map, the programmer needs only to extend this
 * class and provide an implementation for the <tt>entrySet</tt> method, which
 * returns a set-view of the map's mappings.  Typically, the returned set
 * will, in turn, be implemented atop <tt>AbstractSet</tt>.  This set should
 * not support the <tt>add</tt> or <tt>remove</tt> methods, and its iterator
 * should not support the <tt>remove</tt> method.
 *
 * <p>To implement a modifiable map, the programmer must additionally override
 * this class's <tt>put</tt> method (which otherwise throws an
 * <tt>UnsupportedOperationException</tt>), and the iterator returned by
 * <tt>entrySet().iterator()</tt> must additionally implement its
 * <tt>remove</tt> method.
 *
 * <p>The programmer should generally provide a void (no argument) and map
 * constructor, as per the recommendation in the <tt>Map</tt> interface
 * specification.
 *
 * <p>The documentation for each non-abstract method in this class describes its
 * implementation in detail.  Each of these methods may be overridden if the
 * map being implemented admits a more efficient implementation.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see Map
 * @see Collection
 * @since 1.2
 */
```

---

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation returns <tt>entrySet().size()</tt>.
 */
public int size() {
    return entrySet().size();
}

/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation returns <tt>size() == 0</tt>.
 */
public boolean isEmpty() {
    return size() == 0;
}
```

---

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation iterates over <tt>entrySet()</tt> searching
 * for an entry with the specified value.  If such an entry is found,
 * <tt>true</tt> is returned.  If the iteration terminates without
 * finding such an entry, <tt>false</tt> is returned.  Note that this
 * implementation requires linear time in the size of the map.
 *
 * @throws ClassCastException   {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public boolean containsValue(Object value) {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (value==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getValue()==null)
                return true;
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (value.equals(e.getValue()))
                return true;
        }
    }
    return false;
}
```

利用 `entrySet()` 的迭代器迭代整个 Map

检验是否存在指定的 value

检验所需要的时间与 Map 大小呈线性关系

---

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation iterates over <tt>entrySet()</tt> searching
 * for an entry with the specified key.  If such an entry is found,
 * <tt>true</tt> is returned.  If the iteration terminates without
 * finding such an entry, <tt>false</tt> is returned.  Note that this
 * implementation requires linear time in the size of the map; many
 * implementations will override this method.
 *
 * @throws ClassCastException   {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public boolean containsKey(Object key) {
    Iterator<Map.Entry<K,V>> i = entrySet().iterator();
    if (key==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                return true;
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                return true;
        }
    }
    return false;
}
```

利用 `entrySet()` 的迭代器迭代整个 Map

检验是否存在指定的 key

检验所需要的时间与 Map 大小呈线性关系

但大部分的具体实现可以重写这个函数

根据具体的实现方式提升性能

---

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation iterates over <tt>entrySet()</tt> searching
 * for an entry with the specified key.  If such an entry is found,
 * the entry's value is returned.  If the iteration terminates without
 * finding such an entry, <tt>null</tt> is returned.  Note that this
 * implementation requires linear time in the size of the map; many
 * implementations will override this method.
 *
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 */
public V get(Object key) {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (key==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                return e.getValue();
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                return e.getValue();
        }
    }
    return null;
}
```

根据 `entrySet()` 迭代所有的 entry

直到找到 key 对应的 value

如果没有找到，则返回 `null`

具体实现一般会重写这个函数

---

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation always throws an
 * <tt>UnsupportedOperationException</tt>.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 */
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```

需要被重写？

---

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation iterates over <tt>entrySet()</tt> searching for an
 * entry with the specified key.  If such an entry is found, its value is
 * obtained with its <tt>getValue</tt> operation, the entry is removed
 * from the collection (and the backing map) with the iterator's
 * <tt>remove</tt> operation, and the saved value is returned.  If the
 * iteration terminates without finding such an entry, <tt>null</tt> is
 * returned.  Note that this implementation requires linear time in the
 * size of the map; many implementations will override this method.
 *
 * <p>Note that this implementation throws an
 * <tt>UnsupportedOperationException</tt> if the <tt>entrySet</tt>
 * iterator does not support the <tt>remove</tt> method and this map
 * contains a mapping for the specified key.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 */
public V remove(Object key) {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    Entry<K,V> correctEntry = null;
    if (key==null) {
        while (correctEntry==null && i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                correctEntry = e;
        }
    } else {
        while (correctEntry==null && i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                correctEntry = e;
        }
    }

    V oldValue = null;
    if (correctEntry !=null) {
        oldValue = correctEntry.getValue();
        i.remove();
    }
    return oldValue;
}
```

依旧是通过迭代，找到 key 和对应的 entry

然后用迭代器的 `remove()` 函数将 entry 删除，并返回旧的 value

---

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation iterates over the specified map's
 * <tt>entrySet()</tt> collection, and calls this map's <tt>put</tt>
 * operation once for each entry returned by the iteration.
 *
 * <p>Note that this implementation throws an
 * <tt>UnsupportedOperationException</tt> if this map does not support
 * the <tt>put</tt> operation and the specified map is nonempty.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 */
public void putAll(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        put(e.getKey(), e.getValue());
}

/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation calls <tt>entrySet().clear()</tt>.
 *
 * <p>Note that this implementation throws an
 * <tt>UnsupportedOperationException</tt> if the <tt>entrySet</tt>
 * does not support the <tt>clear</tt> operation.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 */
public void clear() {
    entrySet().clear();
}
```

容量操作，全部借助 `entrySet()` 实现

---

```java
/**
 * Each of these fields are initialized to contain an instance of the
 * appropriate view the first time this view is requested.  The views are
 * stateless, so there's no reason to create more than one of each.
 *
 * <p>Since there is no synchronization performed while accessing these fields,
 * it is expected that java.util.Map view classes using these fields have
 * no non-final fields (or any fields at all except for outer-this). Adhering
 * to this rule would make the races on these fields benign.
 *
 * <p>It is also imperative that implementations read the field only once,
 * as in:
 *
 * <pre> {@code
 * public Set<K> keySet() {
 *   Set<K> ks = keySet;  // single racy read
 *   if (ks == null) {
 *     ks = new KeySet();
 *     keySet = ks;
 *   }
 *   return ks;
 * }
 *}</pre>
 */
transient Set<K>        keySet;
transient Collection<V> values;
```

这两个集合会在第一次请求时被构造

之后会在 Map 内被维护

因此之后的请求可以直接返回这个集合

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation returns a set that subclasses {@link AbstractSet}.
 * The subclass's iterator method returns a "wrapper object" over this
 * map's <tt>entrySet()</tt> iterator.  The <tt>size</tt> method
 * delegates to this map's <tt>size</tt> method and the
 * <tt>contains</tt> method delegates to this map's
 * <tt>containsKey</tt> method.
 *
 * <p>The set is created the first time this method is called,
 * and returned in response to all subsequent calls.  No synchronization
 * is performed, so there is a slight chance that multiple calls to this
 * method will not all return the same set.
 */
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new AbstractSet<K>() {
            public Iterator<K> iterator() {
                return new Iterator<K>() {
                    private Iterator<Entry<K,V>> i = entrySet().iterator();

                    public boolean hasNext() {
                        return i.hasNext();
                    }

                    public K next() {
                        return i.next().getKey();
                    }

                    public void remove() {
                        i.remove();
                    }
                };
            }

            public int size() {
                return AbstractMap.this.size();
            }

            public boolean isEmpty() {
                return AbstractMap.this.isEmpty();
            }

            public void clear() {
                AbstractMap.this.clear();
            }

            public boolean contains(Object k) {
                return AbstractMap.this.containsKey(k);
            }
        };
        keySet = ks;
    }
    return ks;
}
```

```java
/**
 * {@inheritDoc}
 *
 * @implSpec
 * This implementation returns a collection that subclasses {@link
 * AbstractCollection}.  The subclass's iterator method returns a
 * "wrapper object" over this map's <tt>entrySet()</tt> iterator.
 * The <tt>size</tt> method delegates to this map's <tt>size</tt>
 * method and the <tt>contains</tt> method delegates to this map's
 * <tt>containsValue</tt> method.
 *
 * <p>The collection is created the first time this method is called, and
 * returned in response to all subsequent calls.  No synchronization is
 * performed, so there is a slight chance that multiple calls to this
 * method will not all return the same collection.
 */
public Collection<V> values() {
    Collection<V> vals = values;
    if (vals == null) {
        vals = new AbstractCollection<V>() {
            public Iterator<V> iterator() {
                return new Iterator<V>() {
                    private Iterator<Entry<K,V>> i = entrySet().iterator();

                    public boolean hasNext() {
                        return i.hasNext();
                    }

                    public V next() {
                        return i.next().getValue();
                    }

                    public void remove() {
                        i.remove();
                    }
                };
            }

            public int size() {
                return AbstractMap.this.size();
            }

            public boolean isEmpty() {
                return AbstractMap.this.isEmpty();
            }

            public void clear() {
                AbstractMap.this.clear();
            }

            public boolean contains(Object v) {
                return AbstractMap.this.containsValue(v);
            }
        };
        values = vals;
    }
    return vals;
}
```

里面的迭代器类是对 Map 的迭代器做了一次封装

并实现了返回的集合的操作函数

* 因为实例化的是抽象类
* 抽象类中定义的函数需要被实现或重写

---

```java
public abstract Set<Entry<K,V>> entrySet();
```

整个一直被使用的函数需要被具体实现的类实现

---

```java
/**
 * Compares the specified object with this map for equality.  Returns
 * <tt>true</tt> if the given object is also a map and the two maps
 * represent the same mappings.  More formally, two maps <tt>m1</tt> and
 * <tt>m2</tt> represent the same mappings if
 * <tt>m1.entrySet().equals(m2.entrySet())</tt>.  This ensures that the
 * <tt>equals</tt> method works properly across different implementations
 * of the <tt>Map</tt> interface.
 *
 * @implSpec
 * This implementation first checks if the specified object is this map;
 * if so it returns <tt>true</tt>.  Then, it checks if the specified
 * object is a map whose size is identical to the size of this map; if
 * not, it returns <tt>false</tt>.  If so, it iterates over this map's
 * <tt>entrySet</tt> collection, and checks that the specified map
 * contains each mapping that this map contains.  If the specified map
 * fails to contain such a mapping, <tt>false</tt> is returned.  If the
 * iteration completes, <tt>true</tt> is returned.
 *
 * @param o object to be compared for equality with this map
 * @return <tt>true</tt> if the specified object is equal to this map
 */
public boolean equals(Object o) {
    if (o == this)
        return true;

    if (!(o instanceof Map))
        return false;
    Map<?,?> m = (Map<?,?>) o;
    if (m.size() != size())
        return false;

    try {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            if (value == null) {
                if (!(m.get(key)==null && m.containsKey(key)))
                    return false;
            } else {
                if (!value.equals(m.get(key)))
                    return false;
            }
        }
    } catch (ClassCastException unused) {
        return false;
    } catch (NullPointerException unused) {
        return false;
    }

    return true;
}
```

首先要保证被比较的对象也是一个 Map

然后分别比较自身和被比较对象的 `entrySet()` 是否完全相同

* 调用 value 对象的 `equals()`

---

```java
/**
 * Returns the hash code value for this map.  The hash code of a map is
 * defined to be the sum of the hash codes of each entry in the map's
 * <tt>entrySet()</tt> view.  This ensures that <tt>m1.equals(m2)</tt>
 * implies that <tt>m1.hashCode()==m2.hashCode()</tt> for any two maps
 * <tt>m1</tt> and <tt>m2</tt>, as required by the general contract of
 * {@link Object#hashCode}.
 *
 * @implSpec
 * This implementation iterates over <tt>entrySet()</tt>, calling
 * {@link Map.Entry#hashCode hashCode()} on each element (entry) in the
 * set, and adding up the results.
 *
 * @return the hash code value for this map
 * @see Map.Entry#hashCode()
 * @see Object#equals(Object)
 * @see Set#equals(Object)
 */
public int hashCode() {
    int h = 0;
    Iterator<Entry<K,V>> i = entrySet().iterator();
    while (i.hasNext())
        h += i.next().hashCode();
    return h;
}
```

迭代 `entrySet()` 并累加每一个 entry 的 hashcode

保证了 `m1.equals(m2)` 隐含 `m1.hashCode()==m2.hashCode()`

---

```java
/**
 * Returns a string representation of this map.  The string representation
 * consists of a list of key-value mappings in the order returned by the
 * map's <tt>entrySet</tt> view's iterator, enclosed in braces
 * (<tt>"{}"</tt>).  Adjacent mappings are separated by the characters
 * <tt>", "</tt> (comma and space).  Each key-value mapping is rendered as
 * the key followed by an equals sign (<tt>"="</tt>) followed by the
 * associated value.  Keys and values are converted to strings as by
 * {@link String#valueOf(Object)}.
 *
 * @return a string representation of this map
 */
public String toString() {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (! i.hasNext())
        return "{}";

    StringBuilder sb = new StringBuilder();
    sb.append('{');
    for (;;) {
        Entry<K,V> e = i.next();
        K key = e.getKey();
        V value = e.getValue();
        sb.append(key   == this ? "(this Map)" : key);
        sb.append('=');
        sb.append(value == this ? "(this Map)" : value);
        if (! i.hasNext())
            return sb.append('}').toString();
        sb.append(',').append(' ');
    }
}
```

以 `entrySet()` 的迭代器顺序，返回 Map 的字符串表示

---

```java
/**
 * Returns a shallow copy of this <tt>AbstractMap</tt> instance: the keys
 * and values themselves are not cloned.
 *
 * @return a shallow copy of this map
 */
protected Object clone() throws CloneNotSupportedException {
    AbstractMap<?,?> result = (AbstractMap<?,?>)super.clone();
    result.keySet = null;
    result.values = null;
    return result;
}
```

浅拷贝；但是没用搞懂这两个置 `null` 的含义

可能是因为没有搞懂 `keySet` 和 `values` 这两个内部变量的具体维护方式

等看具体实现类的时候再看看

---

