# Abstract Class - java.io.InputStream

Created by : Mr Dk.

2020 / 07 / 06 21:41

Nanjing, Jiangsu, China

---

## Definition

这个抽象类是所有表示输入字节流的类的 super class。所有想要继承自该类的子类都需要提供一个能够返回下一个输入字节的函数。

```java
/**
 * This abstract class is the superclass of all classes representing
 * an input stream of bytes.
 *
 * <p> Applications that need to define a subclass of <code>InputStream</code>
 * must always provide a method that returns the next byte of input.
 *
 * @author  Arthur van Hoff
 * @see     java.io.BufferedInputStream
 * @see     java.io.ByteArrayInputStream
 * @see     java.io.DataInputStream
 * @see     java.io.FilterInputStream
 * @see     java.io.InputStream#read()
 * @see     java.io.OutputStream
 * @see     java.io.PushbackInputStream
 * @since   JDK1.0
 */
public abstract class InputStream implements Closeable {

}
```

---

## Read

定义了从输入流中读取下一个字节的抽象函数。返回字节的值范围在 `int` 类型的 `0-255` 之间。如果到达流的末尾，那么返回 `-1`。这个函数会一直阻塞，直到有输入数据可用，或检测到流末尾，或异常抛出。

```java
/**
 * Reads the next byte of data from the input stream. The value byte is
 * returned as an <code>int</code> in the range <code>0</code> to
 * <code>255</code>. If no byte is available because the end of the stream
 * has been reached, the value <code>-1</code> is returned. This method
 * blocks until input data is available, the end of the stream is detected,
 * or an exception is thrown.
 *
 * <p> A subclass must provide an implementation of this method.
 *
 * @return     the next byte of data, or <code>-1</code> if the end of the
 *             stream is reached.
 * @exception  IOException  if an I/O error occurs.
 */
public abstract int read() throws IOException;
```

从输入流中读取指定数量的字节，存放到一个字节数组的指定位置中，并返回实际读取的字节数。尽可能多地读取字节，最多读取的字节数为字节数组的长度，实际上读取的字节可能会少一些，因为可能会遇到流末尾。

```java
/**
 * Reads up to <code>len</code> bytes of data from the input stream into
 * an array of bytes.  An attempt is made to read as many as
 * <code>len</code> bytes, but a smaller number may be read.
 * The number of bytes actually read is returned as an integer.
 *
 * <p> This method blocks until input data is available, end of file is
 * detected, or an exception is thrown.
 *
 * <p> If <code>len</code> is zero, then no bytes are read and
 * <code>0</code> is returned; otherwise, there is an attempt to read at
 * least one byte. If no byte is available because the stream is at end of
 * file, the value <code>-1</code> is returned; otherwise, at least one
 * byte is read and stored into <code>b</code>.
 *
 * <p> The first byte read is stored into element <code>b[off]</code>, the
 * next one into <code>b[off+1]</code>, and so on. The number of bytes read
 * is, at most, equal to <code>len</code>. Let <i>k</i> be the number of
 * bytes actually read; these bytes will be stored in elements
 * <code>b[off]</code> through <code>b[off+</code><i>k</i><code>-1]</code>,
 * leaving elements <code>b[off+</code><i>k</i><code>]</code> through
 * <code>b[off+len-1]</code> unaffected.
 *
 * <p> In every case, elements <code>b[0]</code> through
 * <code>b[off]</code> and elements <code>b[off+len]</code> through
 * <code>b[b.length-1]</code> are unaffected.
 *
 * <p> The <code>read(b,</code> <code>off,</code> <code>len)</code> method
 * for class <code>InputStream</code> simply calls the method
 * <code>read()</code> repeatedly. If the first such call results in an
 * <code>IOException</code>, that exception is returned from the call to
 * the <code>read(b,</code> <code>off,</code> <code>len)</code> method.  If
 * any subsequent call to <code>read()</code> results in a
 * <code>IOException</code>, the exception is caught and treated as if it
 * were end of file; the bytes read up to that point are stored into
 * <code>b</code> and the number of bytes read before the exception
 * occurred is returned. The default implementation of this method blocks
 * until the requested amount of input data <code>len</code> has been read,
 * end of file is detected, or an exception is thrown. Subclasses are encouraged
 * to provide a more efficient implementation of this method.
 *
 * @param      b     the buffer into which the data is read.
 * @param      off   the start offset in array <code>b</code>
 *                   at which the data is written.
 * @param      len   the maximum number of bytes to read.
 * @return     the total number of bytes read into the buffer, or
 *             <code>-1</code> if there is no more data because the end of
 *             the stream has been reached.
 * @exception  IOException If the first byte cannot be read for any reason
 * other than end of file, or if the input stream has been closed, or if
 * some other I/O error occurs.
 * @exception  NullPointerException If <code>b</code> is <code>null</code>.
 * @exception  IndexOutOfBoundsException If <code>off</code> is negative,
 * <code>len</code> is negative, or <code>len</code> is greater than
 * <code>b.length - off</code>
 * @see        java.io.InputStream#read()
 */
public int read(byte b[], int off, int len) throws IOException {
    if (b == null) {
        throw new NullPointerException();
    } else if (off < 0 || len < 0 || len > b.length - off) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return 0;
    }

    int c = read();
    if (c == -1) {
        return -1;
    }
    b[off] = (byte)c;

    int i = 1;
    try {
        for (; i < len ; i++) {
            c = read();
            if (c == -1) {
                break;
            }
            b[off + i] = (byte)c;
        }
    } catch (IOException ee) {
    }
    return i;
}
```

复用上述函数，将一个字节数组完整地作为输入缓冲：

```java
/**
 * Reads some number of bytes from the input stream and stores them into
 * the buffer array <code>b</code>. The number of bytes actually read is
 * returned as an integer.  This method blocks until input data is
 * available, end of file is detected, or an exception is thrown.
 *
 * <p> If the length of <code>b</code> is zero, then no bytes are read and
 * <code>0</code> is returned; otherwise, there is an attempt to read at
 * least one byte. If no byte is available because the stream is at the
 * end of the file, the value <code>-1</code> is returned; otherwise, at
 * least one byte is read and stored into <code>b</code>.
 *
 * <p> The first byte read is stored into element <code>b[0]</code>, the
 * next one into <code>b[1]</code>, and so on. The number of bytes read is,
 * at most, equal to the length of <code>b</code>. Let <i>k</i> be the
 * number of bytes actually read; these bytes will be stored in elements
 * <code>b[0]</code> through <code>b[</code><i>k</i><code>-1]</code>,
 * leaving elements <code>b[</code><i>k</i><code>]</code> through
 * <code>b[b.length-1]</code> unaffected.
 *
 * <p> The <code>read(b)</code> method for class <code>InputStream</code>
 * has the same effect as: <pre><code> read(b, 0, b.length) </code></pre>
 *
 * @param      b   the buffer into which the data is read.
 * @return     the total number of bytes read into the buffer, or
 *             <code>-1</code> if there is no more data because the end of
 *             the stream has been reached.
 * @exception  IOException  If the first byte cannot be read for any reason
 * other than the end of the file, if the input stream has been closed, or
 * if some other I/O error occurs.
 * @exception  NullPointerException  if <code>b</code> is <code>null</code>.
 * @see        java.io.InputStream#read(byte[], int, int)
 */
public int read(byte b[]) throws IOException {
    return read(b, 0, b.length);
}
```

---

## Skip

从输入流中跳过并丢弃一些字节。代码创建了一个缓冲区，专门用于存放读入的废弃字节，直到指定的字节数达到，或达到流末尾。函数返回实际上跳过的字节数。

```java
/**
 * Skips over and discards <code>n</code> bytes of data from this input
 * stream. The <code>skip</code> method may, for a variety of reasons, end
 * up skipping over some smaller number of bytes, possibly <code>0</code>.
 * This may result from any of a number of conditions; reaching end of file
 * before <code>n</code> bytes have been skipped is only one possibility.
 * The actual number of bytes skipped is returned. If {@code n} is
 * negative, the {@code skip} method for class {@code InputStream} always
 * returns 0, and no bytes are skipped. Subclasses may handle the negative
 * value differently.
 *
 * <p> The <code>skip</code> method of this class creates a
 * byte array and then repeatedly reads into it until <code>n</code> bytes
 * have been read or the end of the stream has been reached. Subclasses are
 * encouraged to provide a more efficient implementation of this method.
 * For instance, the implementation may depend on the ability to seek.
 *
 * @param      n   the number of bytes to be skipped.
 * @return     the actual number of bytes skipped.
 * @exception  IOException  if the stream does not support seek,
 *                          or if some other I/O error occurs.
 */
public long skip(long n) throws IOException {

    long remaining = n;
    int nr;

    if (n <= 0) {
        return 0;
    }

    int size = (int)Math.min(MAX_SKIP_BUFFER_SIZE, remaining);
    byte[] skipBuffer = new byte[size];
    while (remaining > 0) {
        nr = read(skipBuffer, 0, (int)Math.min(size, remaining));
        if (nr < 0) {
            break;
        }
        remaining -= nr;
    }

    return n - remaining;
}
```

---

## Available

返回流中还可以读取或跳过的估计字节数，不阻塞。不同的子类实现不同，需要被子类重写。

```java
/**
 * Returns an estimate of the number of bytes that can be read (or
 * skipped over) from this input stream without blocking by the next
 * invocation of a method for this input stream. The next invocation
 * might be the same thread or another thread.  A single read or skip of this
 * many bytes will not block, but may read or skip fewer bytes.
 *
 * <p> Note that while some implementations of {@code InputStream} will return
 * the total number of bytes in the stream, many will not.  It is
 * never correct to use the return value of this method to allocate
 * a buffer intended to hold all data in this stream.
 *
 * <p> A subclass' implementation of this method may choose to throw an
 * {@link IOException} if this input stream has been closed by
 * invoking the {@link #close()} method.
 *
 * <p> The {@code available} method for class {@code InputStream} always
 * returns {@code 0}.
 *
 * <p> This method should be overridden by subclasses.
 *
 * @return     an estimate of the number of bytes that can be read (or skipped
 *             over) from this input stream without blocking or {@code 0} when
 *             it reaches the end of the input stream.
 * @exception  IOException if an I/O error occurs.
 */
public int available() throws IOException {
    return 0;
}
```

---

## Close

```java
/**
 * Closes this input stream and releases any system resources associated
 * with the stream.
 *
 * <p> The <code>close</code> method of <code>InputStream</code> does
 * nothing.
 *
 * @exception  IOException  if an I/O error occurs.
 */
public void close() throws IOException {}
```

---

## Mark / Reset

`mark()` 用于在当前输入流中标记当前的位置，随后调用 `reset()` 时，将会使流复位到最后一次标记的位置，从而能够反复读取相同的字节。其中参数 `readlimit` 指定了在标记位置无效之前，流能够允许读取的最大字节数。也就是说，在调用 `mark()` 之后，如果输入流读取的字节数超过了 `readlimit`，那么调用 `reset()` 就无效了。

```java
/**
 * Marks the current position in this input stream. A subsequent call to
 * the <code>reset</code> method repositions this stream at the last marked
 * position so that subsequent reads re-read the same bytes.
 *
 * <p> The <code>readlimit</code> arguments tells this input stream to
 * allow that many bytes to be read before the mark position gets
 * invalidated.
 *
 * <p> The general contract of <code>mark</code> is that, if the method
 * <code>markSupported</code> returns <code>true</code>, the stream somehow
 * remembers all the bytes read after the call to <code>mark</code> and
 * stands ready to supply those same bytes again if and whenever the method
 * <code>reset</code> is called.  However, the stream is not required to
 * remember any data at all if more than <code>readlimit</code> bytes are
 * read from the stream before <code>reset</code> is called.
 *
 * <p> Marking a closed stream should not have any effect on the stream.
 *
 * <p> The <code>mark</code> method of <code>InputStream</code> does
 * nothing.
 *
 * @param   readlimit   the maximum limit of bytes that can be read before
 *                      the mark position becomes invalid.
 * @see     java.io.InputStream#reset()
 */
public synchronized void mark(int readlimit) {}
```

上述机制是否有效取决于输入流子类是否支持 `mark()`：

```java
/**
 * Tests if this input stream supports the <code>mark</code> and
 * <code>reset</code> methods. Whether or not <code>mark</code> and
 * <code>reset</code> are supported is an invariant property of a
 * particular input stream instance. The <code>markSupported</code> method
 * of <code>InputStream</code> returns <code>false</code>.
 *
 * @return  <code>true</code> if this stream instance supports the mark
 *          and reset methods; <code>false</code> otherwise.
 * @see     java.io.InputStream#mark(int)
 * @see     java.io.InputStream#reset()
 */
public boolean markSupported() {
    return false;
}
```

如果输入流不支持 `mark()`，那么调用 `reset()` 要么抛出异常，要么就将流重置到一个固定状态。

如果输入流支持 `mark()`：

* 如果 `mark()` 没有被调用过，或标记位置已经非法，那么调用 `reset()` 抛出异常
* 否则流就恢复到最近一次调用 `mark()` 的状态

```java
/**
 * Repositions this stream to the position at the time the
 * <code>mark</code> method was last called on this input stream.
 *
 * <p> The general contract of <code>reset</code> is:
 *
 * <ul>
 * <li> If the method <code>markSupported</code> returns
 * <code>true</code>, then:
 *
 *     <ul><li> If the method <code>mark</code> has not been called since
 *     the stream was created, or the number of bytes read from the stream
 *     since <code>mark</code> was last called is larger than the argument
 *     to <code>mark</code> at that last call, then an
 *     <code>IOException</code> might be thrown.
 *
 *     <li> If such an <code>IOException</code> is not thrown, then the
 *     stream is reset to a state such that all the bytes read since the
 *     most recent call to <code>mark</code> (or since the start of the
 *     file, if <code>mark</code> has not been called) will be resupplied
 *     to subsequent callers of the <code>read</code> method, followed by
 *     any bytes that otherwise would have been the next input data as of
 *     the time of the call to <code>reset</code>. </ul>
 *
 * <li> If the method <code>markSupported</code> returns
 * <code>false</code>, then:
 *
 *     <ul><li> The call to <code>reset</code> may throw an
 *     <code>IOException</code>.
 *
 *     <li> If an <code>IOException</code> is not thrown, then the stream
 *     is reset to a fixed state that depends on the particular type of the
 *     input stream and how it was created. The bytes that will be supplied
 *     to subsequent callers of the <code>read</code> method depend on the
 *     particular type of the input stream. </ul></ul>
 *
 * <p>The method <code>reset</code> for class <code>InputStream</code>
 * does nothing except throw an <code>IOException</code>.
 *
 * @exception  IOException  if this stream has not been marked or if the
 *               mark has been invalidated.
 * @see     java.io.InputStream#mark(int)
 * @see     java.io.IOException
 */
public synchronized void reset() throws IOException {
    throw new IOException("mark/reset not supported");
}
```

---

