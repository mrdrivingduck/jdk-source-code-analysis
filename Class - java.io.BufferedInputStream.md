# Class - java.io.BufferedInputStream

Created by : Mr Dk.

2020 / 07 / 27 14:36

Nanjing, Jiangsu, China

---

## Definition

这个类为输入流维护了一个内部 buffer，随着字节的读取或跳过，buffer 会不断被重新填充。Buffer 使得 `mark()` 和 `reset()` 操作能够被实现。

```java
/**
 * A <code>BufferedInputStream</code> adds
 * functionality to another input stream-namely,
 * the ability to buffer the input and to
 * support the <code>mark</code> and <code>reset</code>
 * methods. When  the <code>BufferedInputStream</code>
 * is created, an internal buffer array is
 * created. As bytes  from the stream are read
 * or skipped, the internal buffer is refilled
 * as necessary  from the contained input stream,
 * many bytes at a time. The <code>mark</code>
 * operation  remembers a point in the input
 * stream and the <code>reset</code> operation
 * causes all the  bytes read since the most
 * recent <code>mark</code> operation to be
 * reread before new bytes are  taken from
 * the contained input stream.
 *
 * @author  Arthur van Hoff
 * @since   JDK1.0
 */
public
class BufferedInputStream extends FilterInputStream {

}
```

---

## Fields

内部维护的字节 buffer。这个 buffer 数组有默认的大小，也可以通过构造函数指定大小。

```java
private static int DEFAULT_BUFFER_SIZE = 8192;

/**
 * The maximum size of array to allocate.
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static int MAX_BUFFER_SIZE = Integer.MAX_VALUE - 8;

/**
 * The internal buffer array where the data is stored. When necessary,
 * it may be replaced by another array of
 * a different size.
 */
protected volatile byte buf[];

/**
 * Atomic updater to provide compareAndSet for buf. This is
 * necessary because closes can be asynchronous. We use nullness
 * of buf[] as primary indicator that this stream is closed. (The
 * "in" field is also nulled out on close.)
 */
private static final
    AtomicReferenceFieldUpdater<BufferedInputStream, byte[]> bufUpdater =
    AtomicReferenceFieldUpdater.newUpdater
    (BufferedInputStream.class,  byte[].class, "buf");
```

与 stream 相关的状态变量：

* `count` - 表示 buffer 中的有效长度，范围在 `[0]` 到 `[count - 1]` 之间
* `pos` - 流的目前位置，是下一个从流中被读取的字节位置
* `markpos` - 最后一次调用 `mark()` 时的 `pos` 位置，`-1` 代表不存在标记位置
* `marklimit` - 如果 `pos` 和 `markpos` 之间的举例超过了 `marklimit`，那么 `markpos` 将被丢弃

```java
/**
 * The index one greater than the index of the last valid byte in
 * the buffer.
 * This value is always
 * in the range <code>0</code> through <code>buf.length</code>;
 * elements <code>buf[0]</code>  through <code>buf[count-1]
 * </code>contain buffered input data obtained
 * from the underlying  input stream.
 */
protected int count;

/**
 * The current position in the buffer. This is the index of the next
 * character to be read from the <code>buf</code> array.
 * <p>
 * This value is always in the range <code>0</code>
 * through <code>count</code>. If it is less
 * than <code>count</code>, then  <code>buf[pos]</code>
 * is the next byte to be supplied as input;
 * if it is equal to <code>count</code>, then
 * the  next <code>read</code> or <code>skip</code>
 * operation will require more bytes to be
 * read from the contained  input stream.
 *
 * @see     java.io.BufferedInputStream#buf
 */
protected int pos;

/**
 * The value of the <code>pos</code> field at the time the last
 * <code>mark</code> method was called.
 * <p>
 * This value is always
 * in the range <code>-1</code> through <code>pos</code>.
 * If there is no marked position in  the input
 * stream, this field is <code>-1</code>. If
 * there is a marked position in the input
 * stream,  then <code>buf[markpos]</code>
 * is the first byte to be supplied as input
 * after a <code>reset</code> operation. If
 * <code>markpos</code> is not <code>-1</code>,
 * then all bytes from positions <code>buf[markpos]</code>
 * through  <code>buf[pos-1]</code> must remain
 * in the buffer array (though they may be
 * moved to  another place in the buffer array,
 * with suitable adjustments to the values
 * of <code>count</code>,  <code>pos</code>,
 * and <code>markpos</code>); they may not
 * be discarded unless and until the difference
 * between <code>pos</code> and <code>markpos</code>
 * exceeds <code>marklimit</code>.
 *
 * @see     java.io.BufferedInputStream#mark(int)
 * @see     java.io.BufferedInputStream#pos
 */
protected int markpos = -1;

/**
 * The maximum read ahead allowed after a call to the
 * <code>mark</code> method before subsequent calls to the
 * <code>reset</code> method fail.
 * Whenever the difference between <code>pos</code>
 * and <code>markpos</code> exceeds <code>marklimit</code>,
 * then the  mark may be dropped by setting
 * <code>markpos</code> to <code>-1</code>.
 *
 * @see     java.io.BufferedInputStream#mark(int)
 * @see     java.io.BufferedInputStream#reset()
 */
protected int marklimit;
```

---

## 获取底层数据结构

获取父类 `FilterInputStream` 中维护的 `InputStream`。获取本类中的 buffer。

```java
/**
 * Check to make sure that underlying input stream has not been
 * nulled out due to close; if not return it;
 */
private InputStream getInIfOpen() throws IOException {
    InputStream input = in;
    if (input == null)
        throw new IOException("Stream closed");
    return input;
}

/**
 * Check to make sure that buffer has not been nulled out due to
 * close; if not return it;
 */
private byte[] getBufIfOpen() throws IOException {
    byte[] buffer = buf;
    if (buffer == null)
        throw new IOException("Stream closed");
    return buffer;
}
```

---

## 构造函数

```java
/**
 * Creates a <code>BufferedInputStream</code>
 * and saves its  argument, the input stream
 * <code>in</code>, for later use. An internal
 * buffer array is created and  stored in <code>buf</code>.
 *
 * @param   in   the underlying input stream.
 */
public BufferedInputStream(InputStream in) {
    this(in, DEFAULT_BUFFER_SIZE);
}

/**
 * Creates a <code>BufferedInputStream</code>
 * with the specified buffer size,
 * and saves its  argument, the input stream
 * <code>in</code>, for later use.  An internal
 * buffer array of length  <code>size</code>
 * is created and stored in <code>buf</code>.
 *
 * @param   in     the underlying input stream.
 * @param   size   the buffer size.
 * @exception IllegalArgumentException if {@code size <= 0}.
 */
public BufferedInputStream(InputStream in, int size) {
    super(in);
    if (size <= 0) {
        throw new IllegalArgumentException("Buffer size <= 0");
    }
    buf = new byte[size];
}
```

---

## 填充 buffer

用新的数据填充 buffer。这里显然需要考虑带有 mark 标记的情况，至少需要保留从 mark 标记开始的字节，直到超过 `marklimit`。

```java
/**
 * Fills the buffer with more data, taking into account
 * shuffling and other tricks for dealing with marks.
 * Assumes that it is being called by a synchronized method.
 * This method also assumes that all data has already been read in,
 * hence pos > count.
 */
private void fill() throws IOException {
    byte[] buffer = getBufIfOpen();
    if (markpos < 0)
        // 没有 mark，那么直接丢弃数组中所有的内容
        pos = 0;            /* no mark: throw away the buffer */
    else if (pos >= buffer.length)  /* no room left in buffer */
        // 有 mark，且当前位置已经超出 buffer 长度，开始按情况处理
        if (markpos > 0) {  /* can throw away early part of the buffer */
            // 如果 markpos 的位置不在开头，那么至少可以丢弃 buffer 开头部分到 markpos 的字节
            // 把从 markpos 开始的字节移动到 buffer 开头处
            int sz = pos - markpos;
            System.arraycopy(buffer, markpos, buffer, 0, sz);
            pos = sz;
            markpos = 0;
        } else if (buffer.length >= marklimit) {
            // 如果 marklimit 小于 buffer 的空间，说明 buffer 开辟大了
            // 那么直接丢弃缓存的字节，将下一个读取的位置 pos 设置到数组开头
            markpos = -1;   /* buffer got too big, invalidate mark */
            pos = 0;        /* drop buffer contents */
        } else if (buffer.length >= MAX_BUFFER_SIZE) {
            // buffer 大小超出极限
            throw new OutOfMemoryError("Required array size too large");
        } else {            /* grow buffer */
            // buffer 扩大两倍，同时保证不超过 marklimit
            // 重新开辟 buffer，并从 pos 位置开始将字节复制到新数组
            // 用 CAS 操作替换新数组
            int nsz = (pos <= MAX_BUFFER_SIZE - pos) ?
                    pos * 2 : MAX_BUFFER_SIZE;
            if (nsz > marklimit)
                nsz = marklimit;
            byte nbuf[] = new byte[nsz];
            System.arraycopy(buffer, 0, nbuf, 0, pos);
            if (!bufUpdater.compareAndSet(this, buffer, nbuf)) {
                // Can't replace buf if there was an async close.
                // Note: This would need to be changed if fill()
                // is ever made accessible to multiple threads.
                // But for now, the only way CAS can fail is via close.
                // assert buf == null;
                throw new IOException("Stream closed");
            }
            buffer = nbuf;
        }
    // 将有效长度 count 设置到 pos 的位置，读取字节存放到 buffer 中
    // 如果读取到了新字节，那么更新 buffer 的有效长度 count
    count = pos;
    int n = getInIfOpen().read(buffer, pos, buffer.length - pos);
    if (n > 0)
        count = n + pos;
}
```

---

## Read

读取 stream 中的下一个字节。如果读取的位置已经超出了 buffer 有效长度，那么调用一次 `fill()` 重新填充 buffer。

```java
/**
 * See
 * the general contract of the <code>read</code>
 * method of <code>InputStream</code>.
 *
 * @return     the next byte of data, or <code>-1</code> if the end of the
 *             stream is reached.
 * @exception  IOException  if this input stream has been closed by
 *                          invoking its {@link #close()} method,
 *                          or an I/O error occurs.
 * @see        java.io.FilterInputStream#in
 */
public synchronized int read() throws IOException {
    if (pos >= count) {
        fill();
        if (pos >= count)
            return -1;
    }
    return getBufIfOpen()[pos++] & 0xff;
}
```

将 buffer 中指定数量的字节读取到一个字节数组的指定偏移位置。如果 buffer 中已经没有字节可以读取，那么就调用一次 `fill()` 再读取。`fill()` 最多只能调用一次。最后更新 `pos` 的位置。这是一个内部函数。

```java
/**
 * Read characters into a portion of an array, reading from the underlying
 * stream at most once if necessary.
 */
private int read1(byte[] b, int off, int len) throws IOException {
    int avail = count - pos; // 计算 buffer 中还可以读取的字节数
    if (avail <= 0) {
        /* If the requested length is at least as large as the buffer, and
           if there is no mark/reset activity, do not bother to copy the
           bytes into the local buffer.  In this way buffered streams will
           cascade harmlessly. */
        if (len >= getBufIfOpen().length && markpos < 0) {
            return getInIfOpen().read(b, off, len);
        }
        fill();
        avail = count - pos;
        if (avail <= 0) return -1;
    }
    int cnt = (avail < len) ? avail : len;
    System.arraycopy(getBufIfOpen(), pos, b, off, cnt);
    pos += cnt;
    return cnt;
}
```

以下函数是真正实现 `read()` 接口的函数。该函数会循环调用底层的 `read()`，直到以下条件之一满足：

1. 指定长度的字节已经被读取 (`len`)
2. 底层 `read()` 返回 `-1`，说明读到了结尾
3. 底层 stream 的 `available()` 函数返回 `0`，说明读取之后的内容将会阻塞

```java
/**
 * Reads bytes from this byte-input stream into the specified byte array,
 * starting at the given offset.
 *
 * <p> This method implements the general contract of the corresponding
 * <code>{@link InputStream#read(byte[], int, int) read}</code> method of
 * the <code>{@link InputStream}</code> class.  As an additional
 * convenience, it attempts to read as many bytes as possible by repeatedly
 * invoking the <code>read</code> method of the underlying stream.  This
 * iterated <code>read</code> continues until one of the following
 * conditions becomes true: <ul>
 *
 *   <li> The specified number of bytes have been read,
 *
 *   <li> The <code>read</code> method of the underlying stream returns
 *   <code>-1</code>, indicating end-of-file, or
 *
 *   <li> The <code>available</code> method of the underlying stream
 *   returns zero, indicating that further input requests would block.
 *
 * </ul> If the first <code>read</code> on the underlying stream returns
 * <code>-1</code> to indicate end-of-file then this method returns
 * <code>-1</code>.  Otherwise this method returns the number of bytes
 * actually read.
 *
 * <p> Subclasses of this class are encouraged, but not required, to
 * attempt to read as many bytes as possible in the same fashion.
 *
 * @param      b     destination buffer.
 * @param      off   offset at which to start storing bytes.
 * @param      len   maximum number of bytes to read.
 * @return     the number of bytes read, or <code>-1</code> if the end of
 *             the stream has been reached.
 * @exception  IOException  if this input stream has been closed by
 *                          invoking its {@link #close()} method,
 *                          or an I/O error occurs.
 */
public synchronized int read(byte b[], int off, int len)
    throws IOException
{
    getBufIfOpen(); // Check for closed stream
    if ((off | len | (off + len) | (b.length - (off + len))) < 0) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return 0;
    }

    int n = 0;
    for (;;) {
        int nread = read1(b, off + n, len - n);
        if (nread <= 0)
            return (n == 0) ? nread : n;
        n += nread;
        if (n >= len)
            return n;
        // if not closed but no bytes available, return
        InputStream input = in;
        if (input != null && input.available() <= 0)
            return n;
    }
}
```

---

## Skip

跳过指定数量的字节。同样，如果接触到了 buffer 的尾部，那么调用一次 `fill()` 读取新内容。

```java
    /**
     * See the general contract of the <code>skip</code>
     * method of <code>InputStream</code>.
     *
     * @exception  IOException  if the stream does not support seek,
     *                          or if this input stream has been closed by
     *                          invoking its {@link #close()} method, or an
     *                          I/O error occurs.
     */
    public synchronized long skip(long n) throws IOException {
        getBufIfOpen(); // Check for closed stream
        if (n <= 0) {
            return 0;
        }
        long avail = count - pos;

        if (avail <= 0) {
            // If no mark position set then don't keep in buffer
            if (markpos <0)
                return getInIfOpen().skip(n);

            // Fill in buffer to save bytes for reset
            fill();
            avail = count - pos;
            if (avail <= 0)
                return 0;
        }

        long skipped = (avail < n) ? avail : n;
        pos += skipped;
        return skipped;
    }
```

---

## Available

返回剩余还可以不阻塞地被读取或跳过的字节数。不考虑边界条件，应当会返回输入流中的可读字节数 + buffer 中剩余可读字节数。

```java
/**
 * Returns an estimate of the number of bytes that can be read (or
 * skipped over) from this input stream without blocking by the next
 * invocation of a method for this input stream. The next invocation might be
 * the same thread or another thread.  A single read or skip of this
 * many bytes will not block, but may read or skip fewer bytes.
 * <p>
 * This method returns the sum of the number of bytes remaining to be read in
 * the buffer (<code>count&nbsp;- pos</code>) and the result of calling the
 * {@link java.io.FilterInputStream#in in}.available().
 *
 * @return     an estimate of the number of bytes that can be read (or skipped
 *             over) from this input stream without blocking.
 * @exception  IOException  if this input stream has been closed by
 *                          invoking its {@link #close()} method,
 *                          or an I/O error occurs.
 */
public synchronized int available() throws IOException {
    int n = count - pos;
    int avail = getInIfOpen().available();
    return n > (Integer.MAX_VALUE - avail)
                ? Integer.MAX_VALUE
                : n + avail;
}
```

---

## Mark && Reset

`mark()` 将当前的 `pos` 记录到 `markpos` 中。在 `reset()` 中，将 `pos` 重新设置为 `markpos`。

```java
/**
 * See the general contract of the <code>mark</code>
 * method of <code>InputStream</code>.
 *
 * @param   readlimit   the maximum limit of bytes that can be read before
 *                      the mark position becomes invalid.
 * @see     java.io.BufferedInputStream#reset()
 */
public synchronized void mark(int readlimit) {
    marklimit = readlimit;
    markpos = pos;
}

/**
 * See the general contract of the <code>reset</code>
 * method of <code>InputStream</code>.
 * <p>
 * If <code>markpos</code> is <code>-1</code>
 * (no mark has been set or the mark has been
 * invalidated), an <code>IOException</code>
 * is thrown. Otherwise, <code>pos</code> is
 * set equal to <code>markpos</code>.
 *
 * @exception  IOException  if this stream has not been marked or,
 *                  if the mark has been invalidated, or the stream
 *                  has been closed by invoking its {@link #close()}
 *                  method, or an I/O error occurs.
 * @see        java.io.BufferedInputStream#mark(int)
 */
public synchronized void reset() throws IOException {
    getBufIfOpen(); // Cause exception if closed
    if (markpos < 0)
        throw new IOException("Resetting to invalid mark");
    pos = markpos;
}
```

---

## Mark Support

显然，该类支持 mark/reset 操作。

```java
/**
 * Tests if this input stream supports the <code>mark</code>
 * and <code>reset</code> methods. The <code>markSupported</code>
 * method of <code>BufferedInputStream</code> returns
 * <code>true</code>.
 *
 * @return  a <code>boolean</code> indicating if this stream type supports
 *          the <code>mark</code> and <code>reset</code> methods.
 * @see     java.io.InputStream#mark(int)
 * @see     java.io.InputStream#reset()
 */
public boolean markSupported() {
    return true;
}
```

---

## Close

通过 CAS 操作将 buffer 数组设置为 `null`。另外将底层的 InputStream 也设置为 null。如果流还没有关闭，那么调用流的 `close()` 函数。

```java
/**
 * Closes this input stream and releases any system resources
 * associated with the stream.
 * Once the stream has been closed, further read(), available(), reset(),
 * or skip() invocations will throw an IOException.
 * Closing a previously closed stream has no effect.
 *
 * @exception  IOException  if an I/O error occurs.
 */
public void close() throws IOException {
    byte[] buffer;
    while ( (buffer = buf) != null) {
        if (bufUpdater.compareAndSet(this, buffer, null)) {
            InputStream input = in;
            in = null;
            if (input != null)
                input.close();
            return;
        }
        // Else retry in case a new buf was CASed in fill()
    }
}
```

---

