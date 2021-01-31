# Abstract Class - java.lang.AbstractStringBuilder

Created by : Mr Dk.

2020 / 06 / 05 14:29

Nanjing, Jiangsu, China

---

## Definition

内容和长度都可以被修改的字符序列。与 C 语言的字符串类似，这里实现了所有对于内存空间的维护。从实现的接口可以看出，字符序列是可以被 append 的，同时也需要满足字符序列的所有特性。

```java
/**
 * A mutable sequence of characters.
 * <p>
 * Implements a modifiable string. At any point in time it contains some
 * particular sequence of characters, but the length and content of the
 * sequence can be changed through certain method calls.
 *
 * <p>Unless otherwise noted, passing a {@code null} argument to a constructor
 * or method in this class will cause a {@link NullPointerException} to be
 * thrown.
 *
 * @author      Michael McCloskey
 * @author      Martin Buchholz
 * @author      Ulf Zibis
 * @since       1.5
 */
abstract class AbstractStringBuilder implements Appendable, CharSequence {

}
```

---

维护的内部变量：字符数组内存 + 字符使用的长度 (不一定就是字符数组的内存长度)。

```java
/**
 * The value is used for character storage.
 */
char[] value;

/**
 * The count is the number of characters used.
 */
int count;

/**
 * Needed by {@code String} for the contentEquals method.
 */
final char[] getValue() {
    return value;
}
```

---

构造函数可以指定内部 `char[] value` 的初始容量。

```java
/**
 * This no-arg constructor is necessary for serialization of subclasses.
 */
AbstractStringBuilder() {
}

/**
 * Creates an AbstractStringBuilder of the specified capacity.
 */
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```

---

字符串的 **长度** 与 **容量** 的区别：

```java
/**
 * Returns the length (character count).
 *
 * @return  the length of the sequence of characters currently
 *          represented by this object
 */
@Override
public int length() {
    return count;
}

/**
 * Returns the current capacity. The capacity is the amount of storage
 * available for newly inserted characters, beyond which an allocation
 * will occur.
 *
 * @return  the current capacity
 */
public int capacity() {
    return value.length;
}
```

---

以下是与开辟 `char[]` 相关的函数，主要是为了保证这个数组只要要达到指定的长度。首先尝试将数组的长度扩展到 `2n+2`，如果还没有达到要求，就直接设置为指定的容量，并对一些特殊的大容量进行处理。

```java
/**
 * Ensures that the capacity is at least equal to the specified minimum.
 * If the current capacity is less than the argument, then a new internal
 * array is allocated with greater capacity. The new capacity is the
 * larger of:
 * <ul>
 * <li>The {@code minimumCapacity} argument.
 * <li>Twice the old capacity, plus {@code 2}.
 * </ul>
 * If the {@code minimumCapacity} argument is nonpositive, this
 * method takes no action and simply returns.
 * Note that subsequent operations on this object can reduce the
 * actual capacity below that requested here.
 *
 * @param   minimumCapacity   the minimum desired capacity.
 */
public void ensureCapacity(int minimumCapacity) {
    if (minimumCapacity > 0)
        ensureCapacityInternal(minimumCapacity);
}

/**
 * For positive values of {@code minimumCapacity}, this method
 * behaves like {@code ensureCapacity}, however it is never
 * synchronized.
 * If {@code minimumCapacity} is non positive due to numeric
 * overflow, this method throws {@code OutOfMemoryError}.
 */
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}

/**
 * The maximum size of array to allocate (unless necessary).
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * Returns a capacity at least as large as the given minimum capacity.
 * Returns the current capacity increased by the same amount + 2 if
 * that suffices.
 * Will not return a capacity greater than {@code MAX_ARRAY_SIZE}
 * unless the given minimum capacity is greater than that.
 *
 * @param  minCapacity the desired minimum capacity
 * @throws OutOfMemoryError if minCapacity is less than zero or
 *         greater than Integer.MAX_VALUE
 */
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int newCapacity = (value.length << 1) + 2;
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
        ? hugeCapacity(minCapacity)
        : newCapacity;
}

private int hugeCapacity(int minCapacity) {
    if (Integer.MAX_VALUE - minCapacity < 0) { // overflow
        throw new OutOfMemoryError();
    }
    return (minCapacity > MAX_ARRAY_SIZE)
        ? minCapacity : MAX_ARRAY_SIZE;
}
```

---

以下函数收缩 `char[] value` 的容量，使得 `count` 的值正好等于 `value` 的容量。在内部是通过复制数组的方式实现的。

```java
/**
 * Attempts to reduce storage used for the character sequence.
 * If the buffer is larger than necessary to hold its current sequence of
 * characters, then it may be resized to become more space efficient.
 * Calling this method may, but is not required to, affect the value
 * returned by a subsequent call to the {@link #capacity()} method.
 */
public void trimToSize() {
    if (count < value.length) {
        value = Arrays.copyOf(value, count);
    }
}
```

---

以下函数调整 `value` 的容量。如果目标容量比当前容量小，那么结尾的一些内容将消失；如果目标容量比当前容量大，那么先将 `value` 扩容，然后将多出的空间用 `\0` 填充。

```java
/**
 * Sets the length of the character sequence.
 * The sequence is changed to a new character sequence
 * whose length is specified by the argument. For every nonnegative
 * index <i>k</i> less than {@code newLength}, the character at
 * index <i>k</i> in the new character sequence is the same as the
 * character at index <i>k</i> in the old sequence if <i>k</i> is less
 * than the length of the old character sequence; otherwise, it is the
 * null character {@code '\u005Cu0000'}.
 *
 * In other words, if the {@code newLength} argument is less than
 * the current length, the length is changed to the specified length.
 * <p>
 * If the {@code newLength} argument is greater than or equal
 * to the current length, sufficient null characters
 * ({@code '\u005Cu0000'}) are appended so that
 * length becomes the {@code newLength} argument.
 * <p>
 * The {@code newLength} argument must be greater than or equal
 * to {@code 0}.
 *
 * @param      newLength   the new length
 * @throws     IndexOutOfBoundsException  if the
 *               {@code newLength} argument is negative.
 */
public void setLength(int newLength) {
    if (newLength < 0)
        throw new StringIndexOutOfBoundsException(newLength);
    ensureCapacityInternal(newLength);

    if (count < newLength) {
        Arrays.fill(value, count, newLength, '\0');
    }

    count = newLength;
}
```

---

以下函数将 `value` 中指定范围的内容拷贝到目标字符数组中。

```java
/**
 * Characters are copied from this sequence into the
 * destination character array {@code dst}. The first character to
 * be copied is at index {@code srcBegin}; the last character to
 * be copied is at index {@code srcEnd-1}. The total number of
 * characters to be copied is {@code srcEnd-srcBegin}. The
 * characters are copied into the subarray of {@code dst} starting
 * at index {@code dstBegin} and ending at index:
 * <pre>{@code
 * dstbegin + (srcEnd-srcBegin) - 1
 * }</pre>
 *
 * @param      srcBegin   start copying at this offset.
 * @param      srcEnd     stop copying at this offset.
 * @param      dst        the array to copy the data into.
 * @param      dstBegin   offset into {@code dst}.
 * @throws     IndexOutOfBoundsException  if any of the following is true:
 *             <ul>
 *             <li>{@code srcBegin} is negative
 *             <li>{@code dstBegin} is negative
 *             <li>the {@code srcBegin} argument is greater than
 *             the {@code srcEnd} argument.
 *             <li>{@code srcEnd} is greater than
 *             {@code this.length()}.
 *             <li>{@code dstBegin+srcEnd-srcBegin} is greater than
 *             {@code dst.length}
 *             </ul>
 */
public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)
{
    if (srcBegin < 0)
        throw new StringIndexOutOfBoundsException(srcBegin);
    if ((srcEnd < 0) || (srcEnd > count))
        throw new StringIndexOutOfBoundsException(srcEnd);
    if (srcBegin > srcEnd)
        throw new StringIndexOutOfBoundsException("srcBegin > srcEnd");
    System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
}
```

---

以下代码将`value` 中指定位置的字符设置为指定字符。

```java
/**
 * The character at the specified index is set to {@code ch}. This
 * sequence is altered to represent a new character sequence that is
 * identical to the old character sequence, except that it contains the
 * character {@code ch} at position {@code index}.
 * <p>
 * The index argument must be greater than or equal to
 * {@code 0}, and less than the length of this sequence.
 *
 * @param      index   the index of the character to modify.
 * @param      ch      the new character.
 * @throws     IndexOutOfBoundsException  if {@code index} is
 *             negative or greater than or equal to {@code length()}.
 */
public void setCharAt(int index, char ch) {
    if ((index < 0) || (index >= count))
        throw new StringIndexOutOfBoundsException(index);
    value[index] = ch;
}
```

---

以下函数向 `value` 中进行追加。首先获取输入的长度，并保证 `value` 的空间足够；然后将待追加的内容复制到 `value` 数组中，并更新 `count` 的数值为字符串长度。

```java
/**
 * Appends the string representation of the {@code Object} argument.
 * <p>
 * The overall effect is exactly as if the argument were converted
 * to a string by the method {@link String#valueOf(Object)},
 * and the characters of that string were then
 * {@link #append(String) appended} to this character sequence.
 *
 * @param   obj   an {@code Object}.
 * @return  a reference to this object.
 */
public AbstractStringBuilder append(Object obj) {
    return append(String.valueOf(obj));
}

/**
 * Appends the specified string to this character sequence.
 * <p>
 * The characters of the {@code String} argument are appended, in
 * order, increasing the length of this sequence by the length of the
 * argument. If {@code str} is {@code null}, then the four
 * characters {@code "null"} are appended.
 * <p>
 * Let <i>n</i> be the length of this character sequence just prior to
 * execution of the {@code append} method. Then the character at
 * index <i>k</i> in the new character sequence is equal to the character
 * at index <i>k</i> in the old character sequence, if <i>k</i> is less
 * than <i>n</i>; otherwise, it is equal to the character at index
 * <i>k-n</i> in the argument {@code str}.
 *
 * @param   str   a string.
 * @return  a reference to this object.
 */
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}

/**
 * @since 1.8
 */
AbstractStringBuilder append(AbstractStringBuilder asb) {
    if (asb == null)
        return appendNull();
    int len = asb.length();
    ensureCapacityInternal(count + len);
    asb.getChars(0, len, value, count);
    count += len;
    return this;
}

private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
```

```java
/**
 * Appends a subsequence of the specified {@code CharSequence} to this
 * sequence.
 * <p>
 * Characters of the argument {@code s}, starting at
 * index {@code start}, are appended, in order, to the contents of
 * this sequence up to the (exclusive) index {@code end}. The length
 * of this sequence is increased by the value of {@code end - start}.
 * <p>
 * Let <i>n</i> be the length of this character sequence just prior to
 * execution of the {@code append} method. Then the character at
 * index <i>k</i> in this character sequence becomes equal to the
 * character at index <i>k</i> in this sequence, if <i>k</i> is less than
 * <i>n</i>; otherwise, it is equal to the character at index
 * <i>k+start-n</i> in the argument {@code s}.
 * <p>
 * If {@code s} is {@code null}, then this method appends
 * characters as if the s parameter was a sequence containing the four
 * characters {@code "null"}.
 *
 * @param   s the sequence to append.
 * @param   start   the starting index of the subsequence to be appended.
 * @param   end     the end index of the subsequence to be appended.
 * @return  a reference to this object.
 * @throws     IndexOutOfBoundsException if
 *             {@code start} is negative, or
 *             {@code start} is greater than {@code end} or
 *             {@code end} is greater than {@code s.length()}
 */
@Override
public AbstractStringBuilder append(CharSequence s, int start, int end) {
    if (s == null)
        s = "null";
    if ((start < 0) || (start > end) || (end > s.length()))
        throw new IndexOutOfBoundsException(
            "start " + start + ", end " + end + ", s.length() "
            + s.length());
    int len = end - start;
    ensureCapacityInternal(count + len);
    for (int i = start, j = count; i < end; i++, j++)
        value[j] = s.charAt(i);
    count += len;
    return this;
}

/**
 * Appends the string representation of the {@code char} array
 * argument to this sequence.
 * <p>
 * The characters of the array argument are appended, in order, to
 * the contents of this sequence. The length of this sequence
 * increases by the length of the argument.
 * <p>
 * The overall effect is exactly as if the argument were converted
 * to a string by the method {@link String#valueOf(char[])},
 * and the characters of that string were then
 * {@link #append(String) appended} to this character sequence.
 *
 * @param   str   the characters to be appended.
 * @return  a reference to this object.
 */
public AbstractStringBuilder append(char[] str) {
    int len = str.length;
    ensureCapacityInternal(count + len);
    System.arraycopy(str, 0, value, count, len);
    count += len;
    return this;
}

/**
 * Appends the string representation of a subarray of the
 * {@code char} array argument to this sequence.
 * <p>
 * Characters of the {@code char} array {@code str}, starting at
 * index {@code offset}, are appended, in order, to the contents
 * of this sequence. The length of this sequence increases
 * by the value of {@code len}.
 * <p>
 * The overall effect is exactly as if the arguments were converted
 * to a string by the method {@link String#valueOf(char[],int,int)},
 * and the characters of that string were then
 * {@link #append(String) appended} to this character sequence.
 *
 * @param   str      the characters to be appended.
 * @param   offset   the index of the first {@code char} to append.
 * @param   len      the number of {@code char}s to append.
 * @return  a reference to this object.
 * @throws IndexOutOfBoundsException
 *         if {@code offset < 0} or {@code len < 0}
 *         or {@code offset+len > str.length}
 */
public AbstractStringBuilder append(char str[], int offset, int len) {
    if (len > 0)                // let arraycopy report AIOOBE for len < 0
        ensureCapacityInternal(count + len);
    System.arraycopy(str, offset, value, count, len);
    count += len;
    return this;
}
```

另外，还有支持将各种数据类型 append 到 `value` 的函数。

---

以下函数删除 `value` 中指定范围内的内容。不回收空间，只删除内容，将指定范围之后的内容向前拷贝。

```java
/**
 * Removes the characters in a substring of this sequence.
 * The substring begins at the specified {@code start} and extends to
 * the character at index {@code end - 1} or to the end of the
 * sequence if no such character exists. If
 * {@code start} is equal to {@code end}, no changes are made.
 *
 * @param      start  The beginning index, inclusive.
 * @param      end    The ending index, exclusive.
 * @return     This object.
 * @throws     StringIndexOutOfBoundsException  if {@code start}
 *             is negative, greater than {@code length()}, or
 *             greater than {@code end}.
 */
public AbstractStringBuilder delete(int start, int end) {
    if (start < 0)
        throw new StringIndexOutOfBoundsException(start);
    if (end > count)
        end = count;
    if (start > end)
        throw new StringIndexOutOfBoundsException();
    int len = end - start;
    if (len > 0) {
        System.arraycopy(value, start+len, value, start, count-end);
        count -= len;
    }
    return this;
}

/**
 * Removes the {@code char} at the specified position in this
 * sequence. This sequence is shortened by one {@code char}.
 *
 * <p>Note: If the character at the given index is a supplementary
 * character, this method does not remove the entire character. If
 * correct handling of supplementary characters is required,
 * determine the number of {@code char}s to remove by calling
 * {@code Character.charCount(thisSequence.codePointAt(index))},
 * where {@code thisSequence} is this sequence.
 *
 * @param       index  Index of {@code char} to remove
 * @return      This object.
 * @throws      StringIndexOutOfBoundsException  if the {@code index}
 *              is negative or greater than or equal to
 *              {@code length()}.
 */
public AbstractStringBuilder deleteCharAt(int index) {
    if ((index < 0) || (index >= count))
        throw new StringIndexOutOfBoundsException(index);
    System.arraycopy(value, index+1, value, index, count-index-1);
    count--;
    return this;
}
```

---

以下函数将 `value` 中指定范围内的字符替换为输入的字符串。首先将指定范围之后的字符向前或向后移，中间留出输入字符串的长度；同时，如果字符串长度增加，需要保证 `value` 的空间足够。然后将输入字符串复制到 `value` 的指定范围中。

```java
/**
 * Replaces the characters in a substring of this sequence
 * with characters in the specified {@code String}. The substring
 * begins at the specified {@code start} and extends to the character
 * at index {@code end - 1} or to the end of the
 * sequence if no such character exists. First the
 * characters in the substring are removed and then the specified
 * {@code String} is inserted at {@code start}. (This
 * sequence will be lengthened to accommodate the
 * specified String if necessary.)
 *
 * @param      start    The beginning index, inclusive.
 * @param      end      The ending index, exclusive.
 * @param      str   String that will replace previous contents.
 * @return     This object.
 * @throws     StringIndexOutOfBoundsException  if {@code start}
 *             is negative, greater than {@code length()}, or
 *             greater than {@code end}.
 */
public AbstractStringBuilder replace(int start, int end, String str) {
    if (start < 0)
        throw new StringIndexOutOfBoundsException(start);
    if (start > count)
        throw new StringIndexOutOfBoundsException("start > length()");
    if (start > end)
        throw new StringIndexOutOfBoundsException("start > end");

    if (end > count)
        end = count;
    int len = str.length();
    int newCount = count + len - (end - start);
    ensureCapacityInternal(newCount);

    System.arraycopy(value, end, value, start + len, count - end);
    str.getChars(value, start);
    count = newCount;
    return this;
}
```

---

以下函数返回当前 `value` 中字符串的子串。首先进行边界检查，然后构造新的 `String` 对象：

```java
/**
 * Returns a new {@code String} that contains a subsequence of
 * characters currently contained in this character sequence. The
 * substring begins at the specified index and extends to the end of
 * this sequence.
 *
 * @param      start    The beginning index, inclusive.
 * @return     The new string.
 * @throws     StringIndexOutOfBoundsException  if {@code start} is
 *             less than zero, or greater than the length of this object.
 */
public String substring(int start) {
    return substring(start, count);
}

/**
 * Returns a new character sequence that is a subsequence of this sequence.
 *
 * <p> An invocation of this method of the form
 *
 * <pre>{@code
 * sb.subSequence(begin,&nbsp;end)}</pre>
 *
 * behaves in exactly the same way as the invocation
 *
 * <pre>{@code
 * sb.substring(begin,&nbsp;end)}</pre>
 *
 * This method is provided so that this class can
 * implement the {@link CharSequence} interface.
 *
 * @param      start   the start index, inclusive.
 * @param      end     the end index, exclusive.
 * @return     the specified subsequence.
 *
 * @throws  IndexOutOfBoundsException
 *          if {@code start} or {@code end} are negative,
 *          if {@code end} is greater than {@code length()},
 *          or if {@code start} is greater than {@code end}
 * @spec JSR-51
 */
@Override
public CharSequence subSequence(int start, int end) {
    return substring(start, end);
}

/**
 * Returns a new {@code String} that contains a subsequence of
 * characters currently contained in this sequence. The
 * substring begins at the specified {@code start} and
 * extends to the character at index {@code end - 1}.
 *
 * @param      start    The beginning index, inclusive.
 * @param      end      The ending index, exclusive.
 * @return     The new string.
 * @throws     StringIndexOutOfBoundsException  if {@code start}
 *             or {@code end} are negative or greater than
 *             {@code length()}, or {@code start} is
 *             greater than {@code end}.
 */
public String substring(int start, int end) {
    if (start < 0)
        throw new StringIndexOutOfBoundsException(start);
    if (end > count)
        throw new StringIndexOutOfBoundsException(end);
    if (start > end)
        throw new StringIndexOutOfBoundsException(end - start);
    return new String(value, start, end - start);
}
```

---

以下函数将字符序列插入到 `value` 的某个偏移开始的位置，在实现上与 `replace()` 有些类似。先将指定偏移之后的内容向后复制，然后将待插入的字符序列复制到空出来的空间中。

```java
/**
 * Inserts the string representation of a subarray of the {@code str}
 * array argument into this sequence. The subarray begins at the
 * specified {@code offset} and extends {@code len} {@code char}s.
 * The characters of the subarray are inserted into this sequence at
 * the position indicated by {@code index}. The length of this
 * sequence increases by {@code len} {@code char}s.
 *
 * @param      index    position at which to insert subarray.
 * @param      str       A {@code char} array.
 * @param      offset   the index of the first {@code char} in subarray to
 *             be inserted.
 * @param      len      the number of {@code char}s in the subarray to
 *             be inserted.
 * @return     This object
 * @throws     StringIndexOutOfBoundsException  if {@code index}
 *             is negative or greater than {@code length()}, or
 *             {@code offset} or {@code len} are negative, or
 *             {@code (offset+len)} is greater than
 *             {@code str.length}.
 */
public AbstractStringBuilder insert(int index, char[] str, int offset,
                                    int len)
{
    if ((index < 0) || (index > length()))
        throw new StringIndexOutOfBoundsException(index);
    if ((offset < 0) || (len < 0) || (offset > str.length - len))
        throw new StringIndexOutOfBoundsException(
            "offset " + offset + ", len " + len + ", str.length "
            + str.length);
    ensureCapacityInternal(count + len);
    System.arraycopy(value, index, value, index + len, count - index);
    System.arraycopy(str, offset, value, index, len);
    count += len;
    return this;
}
```

类似地，还有 `insert()` 针对各种数据类型的重载。

---

以下函数用于对字符串进行颠倒。其中需要特殊处理的一个情况是占据两个 `char` 长度的字符 - 在颠倒过后，字符内部的两个 `char` 的顺序不应当被颠倒。函数先按独立 `char` 的方式颠倒字符串，如果在其中发现了占据两个 `char` 的字符，则再扫描一遍字符串将代表高低位的两个颠倒的 `char` 的顺序复原。

```java
/**
 * Causes this character sequence to be replaced by the reverse of
 * the sequence. If there are any surrogate pairs included in the
 * sequence, these are treated as single characters for the
 * reverse operation. Thus, the order of the high-low surrogates
 * is never reversed.
 *
 * Let <i>n</i> be the character length of this character sequence
 * (not the length in {@code char} values) just prior to
 * execution of the {@code reverse} method. Then the
 * character at index <i>k</i> in the new character sequence is
 * equal to the character at index <i>n-k-1</i> in the old
 * character sequence.
 *
 * <p>Note that the reverse operation may result in producing
 * surrogate pairs that were unpaired low-surrogates and
 * high-surrogates before the operation. For example, reversing
 * "\u005CuDC00\u005CuD800" produces "\u005CuD800\u005CuDC00" which is
 * a valid surrogate pair.
 *
 * @return  a reference to this object.
 */
public AbstractStringBuilder reverse() {
    boolean hasSurrogates = false;
    int n = count - 1;
    for (int j = (n-1) >> 1; j >= 0; j--) {
        int k = n - j;
        char cj = value[j];
        char ck = value[k];
        value[j] = ck;
        value[k] = cj;
        if (Character.isSurrogate(cj) ||
            Character.isSurrogate(ck)) {
            hasSurrogates = true;
        }
    }
    if (hasSurrogates) {
        reverseAllValidSurrogatePairs();
    }
    return this;
}

/** Outlined helper method for reverse() */
private void reverseAllValidSurrogatePairs() {
    for (int i = 0; i < count - 1; i++) {
        char c2 = value[i];
        if (Character.isLowSurrogate(c2)) {
            char c1 = value[i + 1];
            if (Character.isHighSurrogate(c1)) {
                value[i++] = c1;
                value[i] = c2;
            }
        }
    }
}
```

---

`java.lang.StringBuilder` 和 `java.lang.StringBuffer` 全部继承自这个类，其中的函数基本都在这个函数中实现。`StringBuffer` 在每个函数上套了一层 `synchronized` 关键字以实现线程安全。

---

