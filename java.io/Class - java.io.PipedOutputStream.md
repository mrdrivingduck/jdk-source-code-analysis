# Class - java.io.PipedOutputStream

Created by : Mr Dk.

2021 / 02 / 03 16:12

Ningbo, Zhejiang, China

---

## Definition

OS 管道的写入端，需要与一个 `PipedInputStream` 连接。如果一个正在从管道中读取数据的线程不再存活，那么管道就被称为 *破裂 (broken)*。

```java
/**
 * A piped output stream can be connected to a piped input stream
 * to create a communications pipe. The piped output stream is the
 * sending end of the pipe. Typically, data is written to a
 * <code>PipedOutputStream</code> object by one thread and data is
 * read from the connected <code>PipedInputStream</code> by some
 * other thread. Attempting to use both objects from a single thread
 * is not recommended as it may deadlock the thread.
 * The pipe is said to be <a name=BROKEN> <i>broken</i> </a> if a
 * thread that was reading data bytes from the connected piped input
 * stream is no longer alive.
 *
 * @author  James Gosling
 * @see     java.io.PipedInputStream
 * @since   JDK1.0
 */
public
class PipedOutputStream extends OutputStream {

}
```

## Connect

类内部维护一个 `PipedInputStream` 对象，用于与当前对象连接。连接完毕后，设置 `PipedInputStream` 对象中的缓冲区位置和标志位。之后该对象就不能和其它对象连接了。

```java
/* REMIND: identification of the read and write sides needs to be
    more sophisticated.  Either using thread groups (but what about
    pipes within a thread?) or using finalization (but it may be a
    long time until the next GC). */
private PipedInputStream sink;

/**
 * Connects this piped output stream to a receiver. If this object
 * is already connected to some other piped input stream, an
 * <code>IOException</code> is thrown.
 * <p>
 * If <code>snk</code> is an unconnected piped input stream and
 * <code>src</code> is an unconnected piped output stream, they may
 * be connected by either the call:
 * <blockquote><pre>
 * src.connect(snk)</pre></blockquote>
 * or the call:
 * <blockquote><pre>
 * snk.connect(src)</pre></blockquote>
 * The two calls have the same effect.
 *
 * @param      snk   the piped input stream to connect to.
 * @exception  IOException  if an I/O error occurs.
 */
public synchronized void connect(PipedInputStream snk) throws IOException {
    if (snk == null) {
        throw new NullPointerException();
    } else if (sink != null || snk.connected) {
        throw new IOException("Already connected");
    }
    sink = snk;
    snk.in = -1;
    snk.out = 0;
    snk.connected = true;
}
```

## Constructor

构造一个空的对象，或用一个已有的 `PipedInputStream` 对象直接完成连接。

```java
/**
 * Creates a piped output stream connected to the specified piped
 * input stream. Data bytes written to this stream will then be
 * available as input from <code>snk</code>.
 *
 * @param      snk   The piped input stream to connect to.
 * @exception  IOException  if an I/O error occurs.
 */
public PipedOutputStream(PipedInputStream snk)  throws IOException {
    connect(snk);
}

/**
 * Creates a piped output stream that is not yet connected to a
 * piped input stream. It must be connected to a piped input stream,
 * either by the receiver or the sender, before being used.
 *
 * @see     java.io.PipedInputStream#connect(java.io.PipedOutputStream)
 * @see     java.io.PipedOutputStream#connect(java.io.PipedInputStream)
 */
public PipedOutputStream() {
}
```

## Write

调用 `PipedInputStream` 中的 `receive()` 函数，向管道中写入数据。如果管道缓冲区已经被写满，那么当前线程将会进入等待状态，直到缓冲区有空间。

```java
/**
 * Writes the specified <code>byte</code> to the piped output stream.
 * <p>
 * Implements the <code>write</code> method of <code>OutputStream</code>.
 *
 * @param      b   the <code>byte</code> to be written.
 * @exception IOException if the pipe is <a href=#BROKEN> broken</a>,
 *          {@link #connect(java.io.PipedInputStream) unconnected},
 *          closed, or if an I/O error occurs.
 */
public void write(int b)  throws IOException {
    if (sink == null) {
        throw new IOException("Pipe not connected");
    }
    sink.receive(b);
}

/**
 * Writes <code>len</code> bytes from the specified byte array
 * starting at offset <code>off</code> to this piped output stream.
 * This method blocks until all the bytes are written to the output
 * stream.
 *
 * @param      b     the data.
 * @param      off   the start offset in the data.
 * @param      len   the number of bytes to write.
 * @exception IOException if the pipe is <a href=#BROKEN> broken</a>,
 *          {@link #connect(java.io.PipedInputStream) unconnected},
 *          closed, or if an I/O error occurs.
 */
public void write(byte b[], int off, int len) throws IOException {
    if (sink == null) {
        throw new IOException("Pipe not connected");
    } else if (b == null) {
        throw new NullPointerException();
    } else if ((off < 0) || (off > b.length) || (len < 0) ||
                ((off + len) > b.length) || ((off + len) < 0)) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return;
    }
    sink.receive(b, off, len);
}
```

## Flush

通知所有等待 `PipedInputStream` 的对象读取管道中的字节。

```java
/**
 * Flushes this output stream and forces any buffered output bytes
 * to be written out.
 * This will notify any readers that bytes are waiting in the pipe.
 *
 * @exception IOException if an I/O error occurs.
 */
public synchronized void flush() throws IOException {
    if (sink != null) {
        synchronized (sink) {
            sink.notifyAll();
        }
    }
}
```

## Close

调用 `PipedInputStream` 的 `receivedLast()` 函数，通知所有等待管道数据的线程读取数据。并设置关闭标志位。

```java
/**
 * Closes this piped output stream and releases any system resources
 * associated with this stream. This stream may no longer be used for
 * writing bytes.
 *
 * @exception  IOException  if an I/O error occurs.
 */
public void close()  throws IOException {
    if (sink != null) {
        sink.receivedLast();
    }
}
```

```java
/**
 * Notifies all waiting threads that the last byte of data has been
 * received.
 */
synchronized void receivedLast() {
    closedByWriter = true;
    notifyAll();
}
```

---

