# Class - java.io.PushbackInputStream

Created by : Mr Dk.

2020 / 08 / 15 0:21

Nanjing, Jiangsu, China

---

## Definition

这个类也在内部维护了一个 buffer，但这个类的特点是，当用户调用 `read()` 不断从流中读取字节后，该类提供了一个 `unread()` 函数可以往流中塞回一些字节，也就是所谓的 *push back*。这样，在下一次调用 `read()` 时，将会返回刚才塞进去的字节。

```java
/**
 * A <code>PushbackInputStream</code> adds
 * functionality to another input stream, namely
 * the  ability to "push back" or "unread"
 * one byte. This is useful in situations where
 * it is  convenient for a fragment of code
 * to read an indefinite number of data bytes
 * that  are delimited by a particular byte
 * value; after reading the terminating byte,
 * the  code fragment can "unread" it, so that
 * the next read operation on the input stream
 * will reread the byte that was pushed back.
 * For example, bytes representing the  characters
 * constituting an identifier might be terminated
 * by a byte representing an  operator character;
 * a method whose job is to read just an identifier
 * can read until it  sees the operator and
 * then push the operator back to be re-read.
 *
 * @author  David Connelly
 * @author  Jonathan Payne
 * @since   JDK1.0
 */
public
class PushbackInputStream extends FilterInputStream {

}
```

---

## Fields

内部维护的变量如下：

* 一个字节数组 `buf`
* 下一个被读取字节的位置 `pos`，数值随着 `read()` 的调用逐渐增大

```java
/**
 * The pushback buffer.
 * @since   JDK1.1
 */
protected byte[] buf;

/**
 * The position within the pushback buffer from which the next byte will
 * be read.  When the buffer is empty, <code>pos</code> is equal to
 * <code>buf.length</code>; when the buffer is full, <code>pos</code> is
 * equal to zero.
 *
 * @since   JDK1.1
 */
protected int pos;
```

---

## Constructor

构造函数很简单，需要传入一个 `InputStream`，另外还需要给定内部字节数组的内存长度。在 `buf` 的内存分配完毕后，`pos` 指针指向数组的最尾部。

```java
/**
 * Creates a <code>PushbackInputStream</code>
 * with a pushback buffer of the specified <code>size</code>,
 * and saves its  argument, the input stream
 * <code>in</code>, for later use. Initially,
 * there is no pushed-back byte  (the field
 * <code>pushBack</code> is initialized to
 * <code>-1</code>).
 *
 * @param  in    the input stream from which bytes will be read.
 * @param  size  the size of the pushback buffer.
 * @exception IllegalArgumentException if {@code size <= 0}
 * @since  JDK1.1
 */
public PushbackInputStream(InputStream in, int size) {
    super(in);
    if (size <= 0) {
        throw new IllegalArgumentException("size <= 0");
    }
    this.buf = new byte[size];
    this.pos = size;
}

/**
 * Creates a <code>PushbackInputStream</code>
 * and saves its  argument, the input stream
 * <code>in</code>, for later use. Initially,
 * there is no pushed-back byte  (the field
 * <code>pushBack</code> is initialized to
 * <code>-1</code>).
 *
 * @param   in   the input stream from which bytes will be read.
 */
public PushbackInputStream(InputStream in) {
    this(in, 1);
}
```

---

## Read

在初始情况下，`pos` 指向 `buf` 的末尾，表示当前没有任何被 push back 的字节。调用 `read()` 时，当 `buf` 中没有任何 push back 字节时，将直接调用 `in` 的 `read()` 函数；如果 `buf` 中有字节，那么优先消费 `buf` 中的字节，再调用 `in` 的 `read()`。

```java
/**
 * Reads the next byte of data from this input stream. The value
 * byte is returned as an <code>int</code> in the range
 * <code>0</code> to <code>255</code>. If no byte is available
 * because the end of the stream has been reached, the value
 * <code>-1</code> is returned. This method blocks until input data
 * is available, the end of the stream is detected, or an exception
 * is thrown.
 *
 * <p> This method returns the most recently pushed-back byte, if there is
 * one, and otherwise calls the <code>read</code> method of its underlying
 * input stream and returns whatever value that method returns.
 *
 * @return     the next byte of data, or <code>-1</code> if the end of the
 *             stream has been reached.
 * @exception  IOException  if this input stream has been closed by
 *             invoking its {@link #close()} method,
 *             or an I/O error occurs.
 * @see        java.io.InputStream#read()
 */
public int read() throws IOException {
    ensureOpen();
    if (pos < buf.length) {
        return buf[pos++] & 0xff;
    }
    return super.read();
}

/**
 * Reads up to <code>len</code> bytes of data from this input stream into
 * an array of bytes.  This method first reads any pushed-back bytes; after
 * that, if fewer than <code>len</code> bytes have been read then it
 * reads from the underlying input stream. If <code>len</code> is not zero, the method
 * blocks until at least 1 byte of input is available; otherwise, no
 * bytes are read and <code>0</code> is returned.
 *
 * @param      b     the buffer into which the data is read.
 * @param      off   the start offset in the destination array <code>b</code>
 * @param      len   the maximum number of bytes read.
 * @return     the total number of bytes read into the buffer, or
 *             <code>-1</code> if there is no more data because the end of
 *             the stream has been reached.
 * @exception  NullPointerException If <code>b</code> is <code>null</code>.
 * @exception  IndexOutOfBoundsException If <code>off</code> is negative,
 * <code>len</code> is negative, or <code>len</code> is greater than
 * <code>b.length - off</code>
 * @exception  IOException  if this input stream has been closed by
 *             invoking its {@link #close()} method,
 *             or an I/O error occurs.
 * @see        java.io.InputStream#read(byte[], int, int)
 */
public int read(byte[] b, int off, int len) throws IOException {
    ensureOpen();
    if (b == null) {
        throw new NullPointerException();
    } else if (off < 0 || len < 0 || len > b.length - off) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return 0;
    }

    int avail = buf.length - pos;
    if (avail > 0) {
        if (len < avail) {
            avail = len;
        }
        System.arraycopy(buf, pos, b, off, avail);
        pos += avail;
        off += avail;
        len -= avail;
    }
    if (len > 0) {
        len = super.read(b, off, len);
        if (len == -1) {
            return avail == 0 ? -1 : avail;
        }
        return avail + len;
    }
    return avail;
}
```

---

## Unread

调用 `unread()` 可以将一个或多个字符塞到 `buf` 数组中。当然，塞入字符的长度显然不能多于 `buf` 的长度，不然放哪儿呢。

```java
/**
 * Pushes back a byte by copying it to the front of the pushback buffer.
 * After this method returns, the next byte to be read will have the value
 * <code>(byte)b</code>.
 *
 * @param      b   the <code>int</code> value whose low-order
 *                  byte is to be pushed back.
 * @exception IOException If there is not enough room in the pushback
 *            buffer for the byte, or this input stream has been closed by
 *            invoking its {@link #close()} method.
 */
public void unread(int b) throws IOException {
    ensureOpen();
    if (pos == 0) {
        throw new IOException("Push back buffer is full");
    }
    buf[--pos] = (byte)b;
}

/**
 * Pushes back a portion of an array of bytes by copying it to the front
 * of the pushback buffer.  After this method returns, the next byte to be
 * read will have the value <code>b[off]</code>, the byte after that will
 * have the value <code>b[off+1]</code>, and so forth.
 *
 * @param b the byte array to push back.
 * @param off the start offset of the data.
 * @param len the number of bytes to push back.
 * @exception IOException If there is not enough room in the pushback
 *            buffer for the specified number of bytes,
 *            or this input stream has been closed by
 *            invoking its {@link #close()} method.
 * @since     JDK1.1
 */
public void unread(byte[] b, int off, int len) throws IOException {
    ensureOpen();
    if (len > pos) {
        throw new IOException("Push back buffer is full");
    }
    pos -= len;
    System.arraycopy(b, off, buf, pos, len);
}
```

---

## Available

在调用 `available()` 时，也要优先计算 `buf` 中的字节数：

```java
/**
 * Returns an estimate of the number of bytes that can be read (or
 * skipped over) from this input stream without blocking by the next
 * invocation of a method for this input stream. The next invocation might be
 * the same thread or another thread.  A single read or skip of this
 * many bytes will not block, but may read or skip fewer bytes.
 *
 * <p> The method returns the sum of the number of bytes that have been
 * pushed back and the value returned by {@link
 * java.io.FilterInputStream#available available}.
 *
 * @return     the number of bytes that can be read (or skipped over) from
 *             the input stream without blocking.
 * @exception  IOException  if this input stream has been closed by
 *             invoking its {@link #close()} method,
 *             or an I/O error occurs.
 * @see        java.io.FilterInputStream#in
 * @see        java.io.InputStream#available()
 */
public int available() throws IOException {
    ensureOpen();
    int n = buf.length - pos;
    int avail = super.available();
    return n > (Integer.MAX_VALUE - avail)
                ? Integer.MAX_VALUE
                : n + avail;
}
```

> 这里用到了一个防溢出技巧，上周在 LeetCode 中遇到过。

---

## Skip

跳过要读取的字节。也是优先跳过 `buf` 中的剩余字节。

```java
/**
 * Skips over and discards <code>n</code> bytes of data from this
 * input stream. The <code>skip</code> method may, for a variety of
 * reasons, end up skipping over some smaller number of bytes,
 * possibly zero.  If <code>n</code> is negative, no bytes are skipped.
 *
 * <p> The <code>skip</code> method of <code>PushbackInputStream</code>
 * first skips over the bytes in the pushback buffer, if any.  It then
 * calls the <code>skip</code> method of the underlying input stream if
 * more bytes need to be skipped.  The actual number of bytes skipped
 * is returned.
 *
 * @param      n  {@inheritDoc}
 * @return     {@inheritDoc}
 * @exception  IOException  if the stream does not support seek,
 *            or the stream has been closed by
 *            invoking its {@link #close()} method,
 *            or an I/O error occurs.
 * @see        java.io.FilterInputStream#in
 * @see        java.io.InputStream#skip(long n)
 * @since      1.2
 */
public long skip(long n) throws IOException {
    ensureOpen();
    if (n <= 0) {
        return 0;
    }

    long pskip = buf.length - pos;
    if (pskip > 0) {
        if (n < pskip) {
            pskip = n;
        }
        pos += pskip;
        n -= pskip;
    }
    if (n > 0) {
        pskip += super.skip(n);
    }
    return pskip;
}
```

---

## Mark Support

这个类不支持 mark。

---

## Close

将内部维护的变量置空：

```java
/**
 * Closes this input stream and releases any system resources
 * associated with the stream.
 * Once the stream has been closed, further read(), unread(),
 * available(), reset(), or skip() invocations will throw an IOException.
 * Closing a previously closed stream has no effect.
 *
 * @exception  IOException  if an I/O error occurs.
 */
public synchronized void close() throws IOException {
    if (in == null)
        return;
    in.close();
    in = null;
    buf = null;
}
```

---

