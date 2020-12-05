# Class - java.io.FilterOutputStream

Created by : Mr Dk.

2020 / 12 / 05 10:04

Nanjing, Jiangsu, China

---

## Definition

该类是所有 *需要对输出信息进行过滤的输出流* 的父类 (过滤指在数据被送入底层输出流之前，可能会对数据进行一些处理)。类自身维护了一个输出流，并重写了对这个输出流的所有操作。该类的子类需要自行重写一部分的操作，可能还需要添加一些新的操作和域。

```java
/**
 * This class is the superclass of all classes that filter output
 * streams. These streams sit on top of an already existing output
 * stream (the <i>underlying</i> output stream) which it uses as its
 * basic sink of data, but possibly transforming the data along the
 * way or providing additional functionality.
 * <p>
 * The class <code>FilterOutputStream</code> itself simply overrides
 * all methods of <code>OutputStream</code> with versions that pass
 * all requests to the underlying output stream. Subclasses of
 * <code>FilterOutputStream</code> may further override some of these
 * methods as well as provide additional methods and fields.
 *
 * @author  Jonathan Payne
 * @since   JDK1.0
 */
public
class FilterOutputStream extends OutputStream {

}
```

## Field

维护一个底层的输出流。

```java
/**
 * The underlying output stream to be filtered.
 */
protected OutputStream out;
```

## Write

```java
/**
 * Writes the specified <code>byte</code> to this output stream.
 * <p>
 * The <code>write</code> method of <code>FilterOutputStream</code>
 * calls the <code>write</code> method of its underlying output stream,
 * that is, it performs <tt>out.write(b)</tt>.
 * <p>
 * Implements the abstract <tt>write</tt> method of <tt>OutputStream</tt>.
 *
 * @param      b   the <code>byte</code>.
 * @exception  IOException  if an I/O error occurs.
 */
public void write(int b) throws IOException {
    out.write(b);
}

/**
 * Writes <code>b.length</code> bytes to this output stream.
 * <p>
 * The <code>write</code> method of <code>FilterOutputStream</code>
 * calls its <code>write</code> method of three arguments with the
 * arguments <code>b</code>, <code>0</code>, and
 * <code>b.length</code>.
 * <p>
 * Note that this method does not call the one-argument
 * <code>write</code> method of its underlying stream with the single
 * argument <code>b</code>.
 *
 * @param      b   the data to be written.
 * @exception  IOException  if an I/O error occurs.
 * @see        java.io.FilterOutputStream#write(byte[], int, int)
 */
public void write(byte b[]) throws IOException {
    write(b, 0, b.length);
}

/**
 * Writes <code>len</code> bytes from the specified
 * <code>byte</code> array starting at offset <code>off</code> to
 * this output stream.
 * <p>
 * The <code>write</code> method of <code>FilterOutputStream</code>
 * calls the <code>write</code> method of one argument on each
 * <code>byte</code> to output.
 * <p>
 * Note that this method does not call the <code>write</code> method
 * of its underlying input stream with the same arguments. Subclasses
 * of <code>FilterOutputStream</code> should provide a more efficient
 * implementation of this method.
 *
 * @param      b     the data.
 * @param      off   the start offset in the data.
 * @param      len   the number of bytes to write.
 * @exception  IOException  if an I/O error occurs.
 * @see        java.io.FilterOutputStream#write(int)
 */
public void write(byte b[], int off, int len) throws IOException {
    if ((off | len | (b.length - (len + off)) | (off + len)) < 0)
        throw new IndexOutOfBoundsException();

    for (int i = 0 ; i < len ; i++) {
        write(b[off + i]);
    }
}
```

## Flush

```java
/**
 * Flushes this output stream and forces any buffered output bytes
 * to be written out to the stream.
 * <p>
 * The <code>flush</code> method of <code>FilterOutputStream</code>
 * calls the <code>flush</code> method of its underlying output stream.
 *
 * @exception  IOException  if an I/O error occurs.
 * @see        java.io.FilterOutputStream#out
 */
public void flush() throws IOException {
    out.flush();
}
```

## Close

```java
/**
 * Closes this output stream and releases any system resources
 * associated with the stream.
 * <p>
 * The <code>close</code> method of <code>FilterOutputStream</code>
 * calls its <code>flush</code> method, and then calls the
 * <code>close</code> method of its underlying output stream.
 *
 * @exception  IOException  if an I/O error occurs.
 * @see        java.io.FilterOutputStream#flush()
 * @see        java.io.FilterOutputStream#out
 */
@SuppressWarnings("try")
public void close() throws IOException {
    try (OutputStream ostream = out) {
        flush();
    }
}
```

---

