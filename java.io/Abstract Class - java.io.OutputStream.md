# Abstract Class - java.io.OutputStream

Created by : Mr Dk.

2020 / 12 / 05 0:22

Nanjing, Jiangsu, China

---

## Definition

所有输出流的抽象类定义。输出流负责接收一些输出字节，并把它们送到相应的汇聚位置。输出流实现了 `Closeable` 和 `Flushable` 接口，因此需要实现 `close()` 和 `flush()` 函数。

```java
/**
 * This abstract class is the superclass of all classes representing
 * an output stream of bytes. An output stream accepts output bytes
 * and sends them to some sink.
 * <p>
 * Applications that need to define a subclass of
 * <code>OutputStream</code> must always provide at least a method
 * that writes one byte of output.
 *
 * @author  Arthur van Hoff
 * @see     java.io.BufferedOutputStream
 * @see     java.io.ByteArrayOutputStream
 * @see     java.io.DataOutputStream
 * @see     java.io.FilterOutputStream
 * @see     java.io.InputStream
 * @see     java.io.OutputStream#write(int)
 * @since   JDK1.0
 */
public abstract class OutputStream implements Closeable, Flushable {
```

---

## Write

定义了一个抽象函数 `write()`，能够将单个字节写入到输出中。这个函数必须被继承的类实现。其中，`int` 参数的低 8 位才是被写到输出中的字节：

```java
/**
 * Writes the specified byte to this output stream. The general
 * contract for <code>write</code> is that one byte is written
 * to the output stream. The byte to be written is the eight
 * low-order bits of the argument <code>b</code>. The 24
 * high-order bits of <code>b</code> are ignored.
 * <p>
 * Subclasses of <code>OutputStream</code> must provide an
 * implementation for this method.
 *
 * @param      b   the <code>byte</code>.
 * @exception  IOException  if an I/O error occurs. In particular,
 *             an <code>IOException</code> may be thrown if the
 *             output stream has been closed.
 */
public abstract void write(int b) throws IOException;
```

基于上述的抽象函数，还实现了将字节批量写入输出的变种。以下函数实现将字节数组指定位置开始指定个数的字节写入输出。首先检查开始写入的字节位置 `off` 是否合法，还要判断 `off` 加上指定的字节长度 `len` 后是否超出字节数组的长度。最终，通过循环调用上述最原始版本的 `write()` 实现批量输出。

```java
/**
 * Writes <code>len</code> bytes from the specified byte array
 * starting at offset <code>off</code> to this output stream.
 * The general contract for <code>write(b, off, len)</code> is that
 * some of the bytes in the array <code>b</code> are written to the
 * output stream in order; element <code>b[off]</code> is the first
 * byte written and <code>b[off+len-1]</code> is the last byte written
 * by this operation.
 * <p>
 * The <code>write</code> method of <code>OutputStream</code> calls
 * the write method of one argument on each of the bytes to be
 * written out. Subclasses are encouraged to override this method and
 * provide a more efficient implementation.
 * <p>
 * If <code>b</code> is <code>null</code>, a
 * <code>NullPointerException</code> is thrown.
 * <p>
 * If <code>off</code> is negative, or <code>len</code> is negative, or
 * <code>off+len</code> is greater than the length of the array
 * <code>b</code>, then an <tt>IndexOutOfBoundsException</tt> is thrown.
 *
 * @param      b     the data.
 * @param      off   the start offset in the data.
 * @param      len   the number of bytes to write.
 * @exception  IOException  if an I/O error occurs. In particular,
 *             an <code>IOException</code> is thrown if the output
 *             stream is closed.
 */
public void write(byte b[], int off, int len) throws IOException {
    if (b == null) {
        throw new NullPointerException();
    } else if ((off < 0) || (off > b.length) || (len < 0) ||
                ((off + len) > b.length) || ((off + len) < 0)) {
        throw new IndexOutOfBoundsException();
    } else if (len == 0) {
        return;
    }
    for (int i = 0 ; i < len ; i++) {
        write(b[off + i]);
    }
}
```

以下函数是将整个字节数组写出，借用了上一个 `write()` 的实现。

```java
/**
 * Writes <code>b.length</code> bytes from the specified byte array
 * to this output stream. The general contract for <code>write(b)</code>
 * is that it should have exactly the same effect as the call
 * <code>write(b, 0, b.length)</code>.
 *
 * @param      b   the data.
 * @exception  IOException  if an I/O error occurs.
 * @see        java.io.OutputStream#write(byte[], int, int)
 */
public void write(byte b[]) throws IOException {
    write(b, 0, b.length);
}
```

## Flush

这个函数应当是输出流中的核心，也是难点。

函数本身的功能是，将输出流内部维护的 buffer 中的全部数据写入目的地。假设输出流的具体实现依赖于底层 OS，比如文件输出流，那么这个函数只保证将 buffer 数据全部交给 OS，但是并不保证数据能直接写入磁盘，具体什么时候写入磁盘由 OS 自身决定 - **这里有可能会丢失数据**。也就是说，这个函数只负责将 **输出流对象** 中维护的缓冲数据 (假设有一个字节数组) 调用 OS 的 write 系统调用写入操作系统高速缓冲区，但是并不调用 fsync 系统调用保证 OS 高速缓冲区中的数据写入物理磁盘。

```java
/**
 * Flushes this output stream and forces any buffered output bytes
 * to be written out. The general contract of <code>flush</code> is
 * that calling it is an indication that, if any bytes previously
 * written have been buffered by the implementation of the output
 * stream, such bytes should immediately be written to their
 * intended destination.
 * <p>
 * If the intended destination of this stream is an abstraction provided by
 * the underlying operating system, for example a file, then flushing the
 * stream guarantees only that bytes previously written to the stream are
 * passed to the operating system for writing; it does not guarantee that
 * they are actually written to a physical device such as a disk drive.
 * <p>
 * The <code>flush</code> method of <code>OutputStream</code> does nothing.
 *
 * @exception  IOException  if an I/O error occurs.
 */
public void flush() throws IOException {
}
```

## Close

关闭输出流并释放占用的系统资源。一个被关闭的流将不能进行输出操作，也不能被重新打开。

```java
/**
 * Closes this output stream and releases any system resources
 * associated with this stream. The general contract of <code>close</code>
 * is that it closes the output stream. A closed stream cannot perform
 * output operations and cannot be reopened.
 * <p>
 * The <code>close</code> method of <code>OutputStream</code> does nothing.
 *
 * @exception  IOException  if an I/O error occurs.
 */
public void close() throws IOException {
}
```

---

