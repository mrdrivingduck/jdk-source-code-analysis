# Class - java.io.PipedInputStream

Created by : Mr Dk.

2021 / 02 / 03 16:12

Ningbo, Zhejiang, China

---

## Definition

Java 对 OS 管道的封装。一个 `PipedInputStream` 需要与一个 `PipedOutputStream` 配对使用，一个线程向 `PipedOutputStream` 写入数据，一个线程从 `PipedInputStream` 中读取数据，从而实现线程间的通信。如果输入输出流同属一个线程，则可能会造成死锁。

```java
/**
 * A piped input stream should be connected
 * to a piped output stream; the piped  input
 * stream then provides whatever data bytes
 * are written to the piped output  stream.
 * Typically, data is read from a <code>PipedInputStream</code>
 * object by one thread  and data is written
 * to the corresponding <code>PipedOutputStream</code>
 * by some  other thread. Attempting to use
 * both objects from a single thread is not
 * recommended, as it may deadlock the thread.
 * The piped input stream contains a buffer,
 * decoupling read operations from write operations,
 * within limits.
 * A pipe is said to be <a name="BROKEN"> <i>broken</i> </a> if a
 * thread that was providing data bytes to the connected
 * piped output stream is no longer alive.
 *
 * @author  James Gosling
 * @see     java.io.PipedOutputStream
 * @since   JDK1.0
 */
public class PipedInputStream extends InputStream {

}
```

## Buffer

类内维护一个 buffer，用于存放管道中待读取的数据。具体实现形式：

```java
private static final int DEFAULT_PIPE_SIZE = 1024;

/**
 * The default size of the pipe's circular input buffer.
 * @since   JDK1.1
 */
// This used to be a constant before the pipe size was allowed
// to change. This field will continue to be maintained
// for backward compatibility.
protected static final int PIPE_SIZE = DEFAULT_PIPE_SIZE;

/**
 * The circular buffer into which incoming data is placed.
 * @since   JDK1.1
 */
protected byte buffer[];

/**
 * The index of the position in the circular buffer at which the
 * next byte of data will be stored when received from the connected
 * piped output stream. <code>in&lt;0</code> implies the buffer is empty,
 * <code>in==out</code> implies the buffer is full
 * @since   JDK1.1
 */
protected int in = -1;

/**
 * The index of the position in the circular buffer at which the next
 * byte of data will be read by this piped input stream.
 * @since   JDK1.1
 */
protected int out = 0;
```

Buffer 底层由一个字节数组实现。`in` 指示从管道中过来的数据将被存放的开始位置，`out` 指示输入流被读取的字节位置。如果 `in == out`，那么意味着 buffer 已满。

Buffer 的初始化工作就是为字节数组分配内存：

```java
private void initPipe(int pipeSize) {
    if (pipeSize <= 0) {
        throw new IllegalArgumentException("Pipe Size <= 0");
    }
    buffer = new byte[pipeSize];
}
```

## Constructor

构造函数主要有两个工作：

1. 为 buffer 分配缓冲区
2. 与一个 `PipedOutputStream` 对象连接

```java
/**
 * Creates a <code>PipedInputStream</code> so
 * that it is connected to the piped output
 * stream <code>src</code>. Data bytes written
 * to <code>src</code> will then be  available
 * as input from this stream.
 *
 * @param      src   the stream to connect to.
 * @exception  IOException  if an I/O error occurs.
 */
public PipedInputStream(PipedOutputStream src) throws IOException {
    this(src, DEFAULT_PIPE_SIZE);
}

/**
 * Creates a <code>PipedInputStream</code> so that it is
 * connected to the piped output stream
 * <code>src</code> and uses the specified pipe size for
 * the pipe's buffer.
 * Data bytes written to <code>src</code> will then
 * be available as input from this stream.
 *
 * @param      src   the stream to connect to.
 * @param      pipeSize the size of the pipe's buffer.
 * @exception  IOException  if an I/O error occurs.
 * @exception  IllegalArgumentException if {@code pipeSize <= 0}.
 * @since      1.6
 */
public PipedInputStream(PipedOutputStream src, int pipeSize)
        throws IOException {
        initPipe(pipeSize);
        connect(src);
}

/**
 * Creates a <code>PipedInputStream</code> so
 * that it is not yet {@linkplain #connect(java.io.PipedOutputStream)
 * connected}.
 * It must be {@linkplain java.io.PipedOutputStream#connect(
 * java.io.PipedInputStream) connected} to a
 * <code>PipedOutputStream</code> before being used.
 */
public PipedInputStream() {
    initPipe(DEFAULT_PIPE_SIZE);
}

/**
 * Creates a <code>PipedInputStream</code> so that it is not yet
 * {@linkplain #connect(java.io.PipedOutputStream) connected} and
 * uses the specified pipe size for the pipe's buffer.
 * It must be {@linkplain java.io.PipedOutputStream#connect(
 * java.io.PipedInputStream)
 * connected} to a <code>PipedOutputStream</code> before being used.
 *
 * @param      pipeSize the size of the pipe's buffer.
 * @exception  IllegalArgumentException if {@code pipeSize <= 0}.
 * @since      1.6
 */
public PipedInputStream(int pipeSize) {
    initPipe(pipeSize);
}
```

`connect()` 函数实现在 `PipedOutputStream` 中。

```java
/**
 * Causes this piped input stream to be connected
 * to the piped  output stream <code>src</code>.
 * If this object is already connected to some
 * other piped output  stream, an <code>IOException</code>
 * is thrown.
 * <p>
 * If <code>src</code> is an
 * unconnected piped output stream and <code>snk</code>
 * is an unconnected piped input stream, they
 * may be connected by either the call:
 *
 * <pre><code>snk.connect(src)</code> </pre>
 * <p>
 * or the call:
 *
 * <pre><code>src.connect(snk)</code> </pre>
 * <p>
 * The two calls have the same effect.
 *
 * @param      src   The piped output stream to connect to.
 * @exception  IOException  if an I/O error occurs.
 */
public void connect(PipedOutputStream src) throws IOException {
    src.connect(this);
}
```

## Receive

从管道中获取字节，并放入 buffer 中等待读取。首先需要确认对象是否满足可以 receive：

* 是否与一个 `PipedOutputStream` 连接
* 管道是否已经被关闭
* 读取管道的线程是否还存活

```java
private void checkStateForReceive() throws IOException {
    if (!connected) {
        throw new IOException("Pipe not connected");
    } else if (closedByWriter || closedByReader) {
        throw new IOException("Pipe closed");
    } else if (readSide != null && !readSide.isAlive()) {
        throw new IOException("Read end dead");
    }
}
```

在确认 buffer 满足接收字节的条件后，还要判断 buffer 是否有空间接收数据。如果没有，就陷入等待中，直到有空间：

```java
private void awaitSpace() throws IOException {
    while (in == out) {
        checkStateForReceive();

        /* full: kick any waiting readers */
        notifyAll();
        try {
            wait(1000);
        } catch (InterruptedException ex) {
            throw new java.io.InterruptedIOException();
        }
    }
}
```

所以以下函数应该是由写者线程调用的。写者线程发现 buffer 已满后，就暂停写入。等待读者线程将数据读出，buffer 有空间后，再继续写入。Buffer 数组会被循环使用。

```java
/**
 * Receives a byte of data.  This method will block if no input is
 * available.
 * @param b the byte being received
 * @exception IOException If the pipe is <a href="#BROKEN"> <code>broken</code></a>,
 *          {@link #connect(java.io.PipedOutputStream) unconnected},
 *          closed, or if an I/O error occurs.
 * @since     JDK1.1
 */
protected synchronized void receive(int b) throws IOException {
    checkStateForReceive();
    writeSide = Thread.currentThread();
    if (in == out)
        awaitSpace();
    if (in < 0) {
        in = 0;
        out = 0;
    }
    buffer[in++] = (byte)(b & 0xFF);
    if (in >= buffer.length) {
        in = 0;
    }
}

/**
 * Receives data into an array of bytes.  This method will
 * block until some input is available.
 * @param b the buffer into which the data is received
 * @param off the start offset of the data
 * @param len the maximum number of bytes received
 * @exception IOException If the pipe is <a href="#BROKEN"> broken</a>,
 *           {@link #connect(java.io.PipedOutputStream) unconnected},
 *           closed,or if an I/O error occurs.
 */
synchronized void receive(byte b[], int off, int len)  throws IOException {
    checkStateForReceive();
    writeSide = Thread.currentThread();
    int bytesToTransfer = len;
    while (bytesToTransfer > 0) {
        if (in == out)
            awaitSpace();
        int nextTransferAmount = 0;
        if (out < in) {
            nextTransferAmount = buffer.length - in;
        } else if (in < out) {
            if (in == -1) {
                in = out = 0;
                nextTransferAmount = buffer.length - in;
            } else {
                nextTransferAmount = out - in;
            }
        }
        if (nextTransferAmount > bytesToTransfer)
            nextTransferAmount = bytesToTransfer;
        assert(nextTransferAmount > 0);
        System.arraycopy(b, off, buffer, in, nextTransferAmount);
        bytesToTransfer -= nextTransferAmount;
        off += nextTransferAmount;
        in += nextTransferAmount;
        if (in >= buffer.length) {
            in = 0;
        }
    }
}
```

## Avaliable

查看 buffer 中还有多少字节可以被读取。通过计算 `in` 和 `out` 之间的距离可以获得。

```java
/**
 * Returns the number of bytes that can be read from this input
 * stream without blocking.
 *
 * @return the number of bytes that can be read from this input stream
 *         without blocking, or {@code 0} if this input stream has been
 *         closed by invoking its {@link #close()} method, or if the pipe
 *         is {@link #connect(java.io.PipedOutputStream) unconnected}, or
 *          <a href="#BROKEN"> <code>broken</code></a>.
 *
 * @exception  IOException  if an I/O error occurs.
 * @since   JDK1.0.2
 */
public synchronized int available() throws IOException {
    if(in < 0)
        return 0;
    else if(in == out)
        return buffer.length;
    else if (in > out)
        return in - out;
    else
        return in + buffer.length - out;
}
```

## Read

读取 buffer 中的字节。首先需要确认可以被读取的条件：

* 与一个 `PipedOutputStream` 连接
* 管道没有被关闭
* 管道写端线程依旧存活

然后等待直到 buffer 中有字节能够被读取。

```java
/**
 * Reads the next byte of data from this piped input stream. The
 * value byte is returned as an <code>int</code> in the range
 * <code>0</code> to <code>255</code>.
 * This method blocks until input data is available, the end of the
 * stream is detected, or an exception is thrown.
 *
 * @return     the next byte of data, or <code>-1</code> if the end of the
 *             stream is reached.
 * @exception  IOException  if the pipe is
 *           {@link #connect(java.io.PipedOutputStream) unconnected},
 *           <a href="#BROKEN"> <code>broken</code></a>, closed,
 *           or if an I/O error occurs.
 */
public synchronized int read()  throws IOException {
    if (!connected) {
        throw new IOException("Pipe not connected");
    } else if (closedByReader) {
        throw new IOException("Pipe closed");
    } else if (writeSide != null && !writeSide.isAlive()
                && !closedByWriter && (in < 0)) {
        throw new IOException("Write end dead");
    }

    readSide = Thread.currentThread();
    int trials = 2;
    while (in < 0) {
        if (closedByWriter) {
            /* closed by writer, return EOF */
            return -1;
        }
        if ((writeSide != null) && (!writeSide.isAlive()) && (--trials < 0)) {
            throw new IOException("Pipe broken");
        }
        /* might be a writer waiting */
        notifyAll();
        try {
            wait(1000);
        } catch (InterruptedException ex) {
            throw new java.io.InterruptedIOException();
        }
    }
    int ret = buffer[out++] & 0xFF;
    if (out >= buffer.length) {
        out = 0;
    }
    if (in == out) {
        /* now empty */
        in = -1;
    }

    return ret;
}

/**
 * Reads up to <code>len</code> bytes of data from this piped input
 * stream into an array of bytes. Less than <code>len</code> bytes
 * will be read if the end of the data stream is reached or if
 * <code>len</code> exceeds the pipe's buffer size.
 * If <code>len </code> is zero, then no bytes are read and 0 is returned;
 * otherwise, the method blocks until at least 1 byte of input is
 * available, end of the stream has been detected, or an exception is
 * thrown.
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
 * @exception  IOException if the pipe is <a href="#BROKEN"> <code>broken</code></a>,
 *           {@link #connect(java.io.PipedOutputStream) unconnected},
 *           closed, or if an I/O error occurs.
 */
public synchronized int read(byte b[], int off, int len)  throws IOException {
    if (b == null) {
        throw new NullPointerException();
    } else if (off < 0 || len < 0 || len > b.length - off) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return 0;
    }

    /* possibly wait on the first character */
    int c = read();
    if (c < 0) {
        return -1;
    }
    b[off] = (byte) c;
    int rlen = 1;
    while ((in >= 0) && (len > 1)) {

        int available;

        if (in > out) {
            available = Math.min((buffer.length - out), (in - out));
        } else {
            available = buffer.length - out;
        }

        // A byte is read beforehand outside the loop
        if (available > (len - 1)) {
            available = len - 1;
        }
        System.arraycopy(buffer, out, b, off + rlen, available);
        out += available;
        rlen += available;
        len -= available;

        if (out >= buffer.length) {
            out = 0;
        }
        if (in == out) {
            /* now empty */
            in = -1;
        }
    }
    return rlen;
}
```

## Close

将一个标志位设置为 `true` 即可。

```java
/**
 * Closes this piped input stream and releases any system resources
 * associated with the stream.
 *
 * @exception  IOException  if an I/O error occurs.
 */
public void close()  throws IOException {
    closedByReader = true;
    synchronized (this) {
        in = -1;
    }
}
```

---

