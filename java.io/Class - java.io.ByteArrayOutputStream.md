# Class - java.io.ByteArrayOutputStream

Created by : Mr Dk.

2020 / 12 / 05 10:57

Nanjing, Jiangsu, China

---

## Definition

该类实现了一个基于字节数组的输出流，对于流的写操作会不断写入这个字节数组，字节数组的长度也会自动增长。类还提供了获得这个字节数组的函数。

```java
/**
 * This class implements an output stream in which the data is
 * written into a byte array. The buffer automatically grows as data
 * is written to it.
 * The data can be retrieved using <code>toByteArray()</code> and
 * <code>toString()</code>.
 * <p>
 * Closing a <tt>ByteArrayOutputStream</tt> has no effect. The methods in
 * this class can be called after the stream has been closed without
 * generating an <tt>IOException</tt>.
 *
 * @author  Arthur van Hoff
 * @since   JDK1.0
 */

public class ByteArrayOutputStream extends OutputStream {

}
```

## Fields

类内部维护一个字节数组，以及字节数组的有效长度。字节数组的长度会根据需要自动扩容。

```java
/**
 * The buffer where data is stored.
 */
protected byte buf[];

/**
 * The number of valid bytes in the buffer.
 */
protected int count;
```

## Constructor

字节数组的默认容量为 32，也支持自定义容量。

```java
/**
 * Creates a new byte array output stream. The buffer capacity is
 * initially 32 bytes, though its size increases if necessary.
 */
public ByteArrayOutputStream() {
    this(32);
}

/**
 * Creates a new byte array output stream, with a buffer capacity of
 * the specified size, in bytes.
 *
 * @param   size   the initial size.
 * @exception  IllegalArgumentException if size is negative.
 */
public ByteArrayOutputStream(int size) {
    if (size < 0) {
        throw new IllegalArgumentException("Negative initial size: "
                                            + size);
    }
    buf = new byte[size];
}
```

## Capacity

与字节数组容量相关的函数操作。在确保字节数组的容量不低于指定值的前提下，每次将数组的大小翻倍，并处理达到数组长度最大值的情况。

```java
/**
 * Increases the capacity if necessary to ensure that it can hold
 * at least the number of elements specified by the minimum
 * capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 * @throws OutOfMemoryError if {@code minCapacity < 0}.  This is
 * interpreted as a request for the unsatisfiably large capacity
 * {@code (long) Integer.MAX_VALUE + (minCapacity - Integer.MAX_VALUE)}.
 */
private void ensureCapacity(int minCapacity) {
    // overflow-conscious code
    if (minCapacity - buf.length > 0)
        grow(minCapacity);
}

/**
 * The maximum size of array to allocate.
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = buf.length;
    int newCapacity = oldCapacity << 1;
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    buf = Arrays.copyOf(buf, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

## Write

写入操作。首先根据要写入的数据长度确保字节数组的容量足够，然后将数据复制到字节数组中。

```java
/**
 * Writes the specified byte to this byte array output stream.
 *
 * @param   b   the byte to be written.
 */
public synchronized void write(int b) {
    ensureCapacity(count + 1);
    buf[count] = (byte) b;
    count += 1;
}

/**
 * Writes <code>len</code> bytes from the specified byte array
 * starting at offset <code>off</code> to this byte array output stream.
 *
 * @param   b     the data.
 * @param   off   the start offset in the data.
 * @param   len   the number of bytes to write.
 */
public synchronized void write(byte b[], int off, int len) {
    if ((off < 0) || (off > b.length) || (len < 0) ||
        ((off + len) - b.length > 0)) {
        throw new IndexOutOfBoundsException();
    }
    ensureCapacity(count + len);
    System.arraycopy(b, off, buf, count, len);
    count += len;
}
```

另外这里提供了一个 `writeTo()` 函数，可以通过参数传入一个输出流，将当前流字节数组中的所有数据写入这个输出流中。

```java
/**
 * Writes the complete contents of this byte array output stream to
 * the specified output stream argument, as if by calling the output
 * stream's write method using <code>out.write(buf, 0, count)</code>.
 *
 * @param      out   the output stream to which to write the data.
 * @exception  IOException  if an I/O error occurs.
 */
public synchronized void writeTo(OutputStream out) throws IOException {
    out.write(buf, 0, count);
}
```

## Reset

将字节数组的有效长度置为 0，相当于清空内部缓存的所有字节。

```java
/**
 * Resets the <code>count</code> field of this byte array output
 * stream to zero, so that all currently accumulated output in the
 * output stream is discarded. The output stream can be used again,
 * reusing the already allocated buffer space.
 *
 * @see     java.io.ByteArrayInputStream#count
 */
public synchronized void reset() {
    count = 0;
}
```

```java
/**
 * Returns the current size of the buffer.
 *
 * @return  the value of the <code>count</code> field, which is the number
 *          of valid bytes in this output stream.
 * @see     java.io.ByteArrayOutputStream#count
 */
public synchronized int size() {
    return count;
}
```

## Retrieve

该类提供函数将内部的字节数组取出来。取出的形式可以是字节数组，也可以是编码后的字符串。

```java
/**
 * Creates a newly allocated byte array. Its size is the current
 * size of this output stream and the valid contents of the buffer
 * have been copied into it.
 *
 * @return  the current contents of this output stream, as a byte array.
 * @see     java.io.ByteArrayOutputStream#size()
 */
public synchronized byte toByteArray()[] {
    return Arrays.copyOf(buf, count);
}

/**
 * Converts the buffer's contents into a string decoding bytes using the
 * platform's default character set. The length of the new <tt>String</tt>
 * is a function of the character set, and hence may not be equal to the
 * size of the buffer.
 *
 * <p> This method always replaces malformed-input and unmappable-character
 * sequences with the default replacement string for the platform's
 * default character set. The {@linkplain java.nio.charset.CharsetDecoder}
 * class should be used when more control over the decoding process is
 * required.
 *
 * @return String decoded from the buffer's contents.
 * @since  JDK1.1
 */
public synchronized String toString() {
    return new String(buf, 0, count);
}

/**
 * Converts the buffer's contents into a string by decoding the bytes using
 * the named {@link java.nio.charset.Charset charset}. The length of the new
 * <tt>String</tt> is a function of the charset, and hence may not be equal
 * to the length of the byte array.
 *
 * <p> This method always replaces malformed-input and unmappable-character
 * sequences with this charset's default replacement string. The {@link
 * java.nio.charset.CharsetDecoder} class should be used when more control
 * over the decoding process is required.
 *
 * @param      charsetName  the name of a supported
 *             {@link java.nio.charset.Charset charset}
 * @return     String decoded from the buffer's contents.
 * @exception  UnsupportedEncodingException
 *             If the named charset is not supported
 * @since      JDK1.1
 */
public synchronized String toString(String charsetName)
    throws UnsupportedEncodingException
{
    return new String(buf, 0, count, charsetName);
}
```

## Close

这个类的关闭操作是无效的，在 `close()` 被调用后，其它函数依旧可以被调用，并且不会抛出异常。

```java
/**
 * Closing a <tt>ByteArrayOutputStream</tt> has no effect. The methods in
 * this class can be called after the stream has been closed without
 * generating an <tt>IOException</tt>.
 */
public void close() throws IOException {
}
```

---

