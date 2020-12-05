# Class - java.io.BufferedOutputStream

Created by : Mr Dk.

2020 / 12 / 05 10:21

Nanjing, Jiangsu, China

---

## Definition

该类对用户的写操作进行了缓存，因此用户的每一次写操作 **不再必须** 引发对底层系统的写调用。

```java
/**
 * The class implements a buffered output stream. By setting up such
 * an output stream, an application can write bytes to the underlying
 * output stream without necessarily causing a call to the underlying
 * system for each byte written.
 *
 * @author  Arthur van Hoff
 * @since   JDK1.0
 */
public
class BufferedOutputStream extends FilterOutputStream {

}
```

## Fields

类内用一个字节数组 `buf` 作为缓存，从数组的 `0` 位置开始是有效的，`count` 记录了数组中的有效字节长度。

```java
/**
 * The internal buffer where data is stored.
 */
protected byte buf[];

/**
 * The number of valid bytes in the buffer. This value is always
 * in the range <tt>0</tt> through <tt>buf.length</tt>; elements
 * <tt>buf[0]</tt> through <tt>buf[count-1]</tt> contain valid
 * byte data.
 */
protected int count;
```

## Constructor

构造函数，支持默认的 buffer 大小，也支持自定义的。

```java
/**
 * Creates a new buffered output stream to write data to the
 * specified underlying output stream.
 *
 * @param   out   the underlying output stream.
 */
public BufferedOutputStream(OutputStream out) {
    this(out, 8192);
}

/**
 * Creates a new buffered output stream to write data to the
 * specified underlying output stream with the specified buffer
 * size.
 *
 * @param   out    the underlying output stream.
 * @param   size   the buffer size.
 * @exception IllegalArgumentException if size &lt;= 0.
 */
public BufferedOutputStream(OutputStream out, int size) {
    super(out);
    if (size <= 0) {
        throw new IllegalArgumentException("Buffer size <= 0");
    }
    buf = new byte[size];
}
```

## Flush

这里的 flush 包含两层含义：

1. 将类内 buffer 中缓存的数据 `write` 到底层输出流中，这种 flush 是自动触发的
2. 将类内 buffer 中缓存的数据 `write` 到底层输出流，并调用底层输出流的 `flush` 写入底层系统，由 `flush()` 函数手动触发

自动触发的 flush 一般由类的 `write()` 函数触发，因为缓存的 buffer 装不下了，所以需要向底层输出流 `write()` 一次：

```java
/** Flush the internal buffer */
private void flushBuffer() throws IOException {
    if (count > 0) {
        out.write(buf, 0, count);
        count = 0;
    }
}
```

```java
/**
 * Flushes this buffered output stream. This forces any buffered
 * output bytes to be written out to the underlying output stream.
 *
 * @exception  IOException  if an I/O error occurs.
 * @see        java.io.FilterOutputStream#out
 */
public synchronized void flush() throws IOException {
    flushBuffer();
    out.flush();
}
```

## Write

该类重写了父类的写操作，调用该类的 `write()` 后，通过参数传入的数据会被缓存到 `buf` 中。只有当 `buf` 装不下的时候，才会触发调用底层输出流的 `write()`，清空 `buf`。

```java
/**
 * Writes the specified byte to this buffered output stream.
 *
 * @param      b   the byte to be written.
 * @exception  IOException  if an I/O error occurs.
 */
public synchronized void write(int b) throws IOException {
    if (count >= buf.length) {
        flushBuffer();
    }
    buf[count++] = (byte)b;
}

/**
 * Writes <code>len</code> bytes from the specified byte array
 * starting at offset <code>off</code> to this buffered output stream.
 *
 * <p> Ordinarily this method stores bytes from the given array into this
 * stream's buffer, flushing the buffer to the underlying output stream as
 * needed.  If the requested length is at least as large as this stream's
 * buffer, however, then this method will flush the buffer and write the
 * bytes directly to the underlying output stream.  Thus redundant
 * <code>BufferedOutputStream</code>s will not copy data unnecessarily.
 *
 * @param      b     the data.
 * @param      off   the start offset in the data.
 * @param      len   the number of bytes to write.
 * @exception  IOException  if an I/O error occurs.
 */
public synchronized void write(byte b[], int off, int len) throws IOException {
    if (len >= buf.length) {
        /* If the request length exceeds the size of the output buffer,
           flush the output buffer and then write the data directly.
           In this way buffered streams will cascade harmlessly. */
        flushBuffer();
        out.write(b, off, len);
        return;
    }
    if (len > buf.length - count) {
        flushBuffer();
    }
    System.arraycopy(b, off, buf, count, len);
    count += len;
}
```

---

