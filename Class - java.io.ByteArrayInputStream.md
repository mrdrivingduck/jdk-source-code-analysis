# Class - java.io.ByteArrayInputStream

Created by : Mr Dk.

2020 / 08 / 06 13:58

Nanjing, Jiangsu, China

---

## Definition

这个类包含了一个内部 buffer，里面存放将被读取的字节。关闭这个流没有任何效果，继续调用其中的成员函数不会产生 `IOException`。

```java
/**
 * A <code>ByteArrayInputStream</code> contains
 * an internal buffer that contains bytes that
 * may be read from the stream. An internal
 * counter keeps track of the next byte to
 * be supplied by the <code>read</code> method.
 * <p>
 * Closing a <tt>ByteArrayInputStream</tt> has no effect. The methods in
 * this class can be called after the stream has been closed without
 * generating an <tt>IOException</tt>.
 *
 * @author  Arthur van Hoff
 * @see     java.io.StringBufferInputStream
 * @since   JDK1.0
 */
public
class ByteArrayInputStream extends InputStream {

}
```

---

## Members

其中包含一个内部字节数组 `buf`。从 `buf[0]` 到 `buf[count-1]` 之间的字节是可以被流读取的字节，`buf[pos]` 是下一个将被读取的字节。

另外还有一个用于支持 `mark()` / `reset()` 的 `mark` 变量。

```java
/**
 * An array of bytes that was provided
 * by the creator of the stream. Elements <code>buf[0]</code>
 * through <code>buf[count-1]</code> are the
 * only bytes that can ever be read from the
 * stream;  element <code>buf[pos]</code> is
 * the next byte to be read.
 */
protected byte buf[];

/**
 * The index of the next character to read from the input stream buffer.
 * This value should always be nonnegative
 * and not larger than the value of <code>count</code>.
 * The next byte to be read from the input stream buffer
 * will be <code>buf[pos]</code>.
 */
protected int pos;

/**
 * The currently marked position in the stream.
 * ByteArrayInputStream objects are marked at position zero by
 * default when constructed.  They may be marked at another
 * position within the buffer by the <code>mark()</code> method.
 * The current buffer position is set to this point by the
 * <code>reset()</code> method.
 * <p>
 * If no mark has been set, then the value of mark is the offset
 * passed to the constructor (or 0 if the offset was not supplied).
 *
 * @since   JDK1.1
 */
protected int mark = 0;

/**
 * The index one greater than the last valid character in the input
 * stream buffer.
 * This value should always be nonnegative
 * and not larger than the length of <code>buf</code>.
 * It  is one greater than the position of
 * the last byte within <code>buf</code> that
 * can ever be read  from the input stream buffer.
 */
protected int count;
```

---

## Constructor

构造函数负责创建对象，其中所有的内部成员都是在对象外部创建并传入的。

```java
/**
 * Creates <code>ByteArrayInputStream</code>
 * that uses <code>buf</code> as its
 * buffer array. The initial value of <code>pos</code>
 * is <code>offset</code> and the initial value
 * of <code>count</code> is the minimum of <code>offset+length</code>
 * and <code>buf.length</code>.
 * The buffer array is not copied. The buffer's mark is
 * set to the specified offset.
 *
 * @param   buf      the input buffer.
 * @param   offset   the offset in the buffer of the first byte to read.
 * @param   length   the maximum number of bytes to read from the buffer.
 */
public ByteArrayInputStream(byte buf[], int offset, int length) {
    this.buf = buf;
    this.pos = offset;
    this.count = Math.min(offset + length, buf.length);
    this.mark = offset;
}
```

---

## Read

读取内部字节数组中的内容。如果已经读取到有效长度尾部，那么返回 `-1`。这里可以看到与 `BufferedInputStream` 中的区别：`BufferedInputStream` 也在内部维护了一个缓冲区，但是长度是提前指定的，当调用 `read()` 把缓冲区中的内容读尽后，会再次调用 `fill()` 从数据源读取数据再次填充缓冲区，从而可以以不占太多内存的方式读取一个较大的数据源；而 `ByteArrayInputStream` 维护的内部数组是外部构造完毕的，读完了就没了。

```java
/**
 * Reads the next byte of data from this input stream. The value
 * byte is returned as an <code>int</code> in the range
 * <code>0</code> to <code>255</code>. If no byte is available
 * because the end of the stream has been reached, the value
 * <code>-1</code> is returned.
 * <p>
 * This <code>read</code> method
 * cannot block.
 *
 * @return  the next byte of data, or <code>-1</code> if the end of the
 *          stream has been reached.
 */
public synchronized int read() {
    return (pos < count) ? (buf[pos++] & 0xff) : -1;
}
```

下面的函数将内部字节数组中的内容读取到目标字节数组指定的开始位置，读取指定的最大长度：

```java
/**
 * Reads up to <code>len</code> bytes of data into an array of bytes
 * from this input stream.
 * If <code>pos</code> equals <code>count</code>,
 * then <code>-1</code> is returned to indicate
 * end of file. Otherwise, the  number <code>k</code>
 * of bytes read is equal to the smaller of
 * <code>len</code> and <code>count-pos</code>.
 * If <code>k</code> is positive, then bytes
 * <code>buf[pos]</code> through <code>buf[pos+k-1]</code>
 * are copied into <code>b[off]</code>  through
 * <code>b[off+k-1]</code> in the manner performed
 * by <code>System.arraycopy</code>. The
 * value <code>k</code> is added into <code>pos</code>
 * and <code>k</code> is returned.
 * <p>
 * This <code>read</code> method cannot block.
 *
 * @param   b     the buffer into which the data is read.
 * @param   off   the start offset in the destination array <code>b</code>
 * @param   len   the maximum number of bytes read.
 * @return  the total number of bytes read into the buffer, or
 *          <code>-1</code> if there is no more data because the end of
 *          the stream has been reached.
 * @exception  NullPointerException If <code>b</code> is <code>null</code>.
 * @exception  IndexOutOfBoundsException If <code>off</code> is negative,
 * <code>len</code> is negative, or <code>len</code> is greater than
 * <code>b.length - off</code>
 */
public synchronized int read(byte b[], int off, int len) {
    if (b == null) {
        throw new NullPointerException();
    } else if (off < 0 || len < 0 || len > b.length - off) {
        throw new IndexOutOfBoundsException();
    }

    // 内部字节数组已经读完
    if (pos >= count) {
        return -1;
    }

    // 剩余可读字节
    int avail = count - pos;
    if (len > avail) {
        len = avail;
    }
    if (len <= 0) {
        return 0;
    }
    // 拷贝字节
    System.arraycopy(buf, pos, b, off, len);
    pos += len;
    return len;
}
```

---

## Skip

跳过指定数量的当前可读取字节。如果到达了流尾部，那么可能跳过的字节数会变小。具体是通过前移 `pos` 指针来实现的。

```java
/**
 * Skips <code>n</code> bytes of input from this input stream. Fewer
 * bytes might be skipped if the end of the input stream is reached.
 * The actual number <code>k</code>
 * of bytes to be skipped is equal to the smaller
 * of <code>n</code> and  <code>count-pos</code>.
 * The value <code>k</code> is added into <code>pos</code>
 * and <code>k</code> is returned.
 *
 * @param   n   the number of bytes to be skipped.
 * @return  the actual number of bytes skipped.
 */
public synchronized long skip(long n) {
    long k = count - pos; // 流中目前还有的字节数
    if (n < k) {
        k = n < 0 ? 0 : n;
    }

    pos += k;
    return k;
}
```

---

## Available

返回剩余可读的字节数。

```java
/**
 * Returns the number of remaining bytes that can be read (or skipped over)
 * from this input stream.
 * <p>
 * The value returned is <code>count&nbsp;- pos</code>,
 * which is the number of bytes remaining to be read from the input buffer.
 *
 * @return  the number of remaining bytes that can be read (or skipped
 *          over) from this input stream without blocking.
 */
public synchronized int available() {
    return count - pos;
}
```

---

## Mark Support

这个类也是支持 mark 功能的。有专门的成员 `mark` 用于标记 mark 的位置。

```java
/**
 * Tests if this <code>InputStream</code> supports mark/reset. The
 * <code>markSupported</code> method of <code>ByteArrayInputStream</code>
 * always returns <code>true</code>.
 *
 * @since   JDK1.1
 */
public boolean markSupported() {
    return true;
}

/**
 * Set the current marked position in the stream.
 * ByteArrayInputStream objects are marked at position zero by
 * default when constructed.  They may be marked at another
 * position within the buffer by this method.
 * <p>
 * If no mark has been set, then the value of the mark is the
 * offset passed to the constructor (or 0 if the offset was not
 * supplied).
 *
 * <p> Note: The <code>readAheadLimit</code> for this class
 *  has no meaning.
 *
 * @since   JDK1.1
 */
public void mark(int readAheadLimit) {
    mark = pos;
}

/**
 * Resets the buffer to the marked position.  The marked position
 * is 0 unless another position was marked or an offset was specified
 * in the constructor.
 */
public synchronized void reset() {
    pos = mark;
}
```

---

## Close

关闭这个流没有任何实质性的效果。

```java
/**
 * Closing a <tt>ByteArrayInputStream</tt> has no effect. The methods in
 * this class can be called after the stream has been closed without
 * generating an <tt>IOException</tt>.
 */
public void close() throws IOException {
}
```

---

