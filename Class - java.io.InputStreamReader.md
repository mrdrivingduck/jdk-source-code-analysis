# Class - java.io.InputStreamReader

Created by : Mr Dk.

2020 / 10 / 02 11:11

Ningbo, Zhejiang, China

---

## Definition

这个类是 **字节流** (byte) 转为 **字符流** (char) 的桥梁。除了要接收一个字节流 (InputStream) 以外，还要接收一个字符编码的参数。然后按照编码指定的方式将字节解析为字符。

```java
/**
 * An InputStreamReader is a bridge from byte streams to character streams: It
 * reads bytes and decodes them into characters using a specified {@link
 * java.nio.charset.Charset charset}.  The charset that it uses
 * may be specified by name or may be given explicitly, or the platform's
 * default charset may be accepted.
 *
 * <p> Each invocation of one of an InputStreamReader's read() methods may
 * cause one or more bytes to be read from the underlying byte-input stream.
 * To enable the efficient conversion of bytes to characters, more bytes may
 * be read ahead from the underlying stream than are necessary to satisfy the
 * current read operation.
 *
 * <p> For top efficiency, consider wrapping an InputStreamReader within a
 * BufferedReader.  For example:
 *
 * <pre>
 * BufferedReader in
 *   = new BufferedReader(new InputStreamReader(System.in));
 * </pre>
 *
 * @see BufferedReader
 * @see InputStream
 * @see java.nio.charset.Charset
 *
 * @author      Mark Reinhold
 * @since       JDK1.1
 */

public class InputStreamReader extends Reader {

}
```

## Member & Constructor

该类每部维护一个字节流的解码器。该类的构造函数被调用时，将字节流与编码集封装为这个对象。如果不支持参数指定的编码集，则抛出异常。支持多种形式的字符编码参数：

```java
private final StreamDecoder sd;

/**
 * Creates an InputStreamReader that uses the default charset.
 *
 * @param  in   An InputStream
 */
public InputStreamReader(InputStream in) {
    super(in);
    try {
        sd = StreamDecoder.forInputStreamReader(in, this, (String)null); // ## check lock object
    } catch (UnsupportedEncodingException e) {
        // The default encoding should always be available
        throw new Error(e);
    }
}

/**
 * Creates an InputStreamReader that uses the named charset.
 *
 * @param  in
 *         An InputStream
 *
 * @param  charsetName
 *         The name of a supported
 *         {@link java.nio.charset.Charset charset}
 *
 * @exception  UnsupportedEncodingException
 *             If the named charset is not supported
 */
public InputStreamReader(InputStream in, String charsetName)
    throws UnsupportedEncodingException
{
    super(in);
    if (charsetName == null)
        throw new NullPointerException("charsetName");
    sd = StreamDecoder.forInputStreamReader(in, this, charsetName);
}

/**
 * Creates an InputStreamReader that uses the given charset.
 *
 * @param  in       An InputStream
 * @param  cs       A charset
 *
 * @since 1.4
 * @spec JSR-51
 */
public InputStreamReader(InputStream in, Charset cs) {
    super(in);
    if (cs == null)
        throw new NullPointerException("charset");
    sd = StreamDecoder.forInputStreamReader(in, this, cs);
}

/**
 * Creates an InputStreamReader that uses the given charset decoder.
 *
 * @param  in       An InputStream
 * @param  dec      A charset decoder
 *
 * @since 1.4
 * @spec JSR-51
 */
public InputStreamReader(InputStream in, CharsetDecoder dec) {
    super(in);
    if (dec == null)
        throw new NullPointerException("charset decoder");
    sd = StreamDecoder.forInputStreamReader(in, this, dec);
}
```

## Encoding

取得字符编码名称：

```java
/**
 * Returns the name of the character encoding being used by this stream.
 *
 * <p> If the encoding has an historical name then that name is returned;
 * otherwise the encoding's canonical name is returned.
 *
 * <p> If this instance was created with the {@link
 * #InputStreamReader(InputStream, String)} constructor then the returned
 * name, being unique for the encoding, may differ from the name passed to
 * the constructor. This method will return <code>null</code> if the
 * stream has been closed.
 * </p>
 * @return The historical name of this encoding, or
 *         <code>null</code> if the stream has been closed
 *
 * @see java.nio.charset.Charset
 *
 * @revised 1.4
 * @spec JSR-51
 */
public String getEncoding() {
    return sd.getEncoding();
}
```

## Read

读取字符。可以看到目标的参数已经由 `byte[]` 变为了 `char[]`。

```java
/**
 * Reads a single character.
 *
 * @return The character read, or -1 if the end of the stream has been
 *         reached
 *
 * @exception  IOException  If an I/O error occurs
 */
public int read() throws IOException {
    return sd.read();
}

/**
 * Reads characters into a portion of an array.
 *
 * @param      cbuf     Destination buffer
 * @param      offset   Offset at which to start storing characters
 * @param      length   Maximum number of characters to read
 *
 * @return     The number of characters read, or -1 if the end of the
 *             stream has been reached
 *
 * @exception  IOException  If an I/O error occurs
 */
public int read(char cbuf[], int offset, int length) throws IOException {
    return sd.read(cbuf, offset, length);
}
```

## Ready

```java
/**
 * Tells whether this stream is ready to be read.  An InputStreamReader is
 * ready if its input buffer is not empty, or if bytes are available to be
 * read from the underlying byte stream.
 *
 * @exception  IOException  If an I/O error occurs
 */
public boolean ready() throws IOException {
    return sd.ready();
}
```

## Close

```java
public void close() throws IOException {
    sd.close();
}
```

---

