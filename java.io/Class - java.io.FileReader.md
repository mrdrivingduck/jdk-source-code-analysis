# Class - java.io.FileReader

Created by : Mr Dk.

2020 / 10 / 02 11:31

Ningbo, Zhejiang, China

---

## Definition

这个类完全是 `FileInputStream` 类的复制。在 `FileInputStream` 类中，将一个打开的文件描述符抽象为一个输入。在这里，直接构造了 `FileInputStream` 对象作为 `Reader` 类的输入参数，且不指定另一个字符编码参数，即 `null`。

其余的所有函数全部继承自 `InputStreamReader`，包括几个不同形式的 char 版本的 `read()`，以及 `ready()` 和 `close()`。

```java
/**
 * Convenience class for reading character files.  The constructors of this
 * class assume that the default character encoding and the default byte-buffer
 * size are appropriate.  To specify these values yourself, construct an
 * InputStreamReader on a FileInputStream.
 *
 * <p><code>FileReader</code> is meant for reading streams of characters.
 * For reading streams of raw bytes, consider using a
 * <code>FileInputStream</code>.
 *
 * @see InputStreamReader
 * @see FileInputStream
 *
 * @author      Mark Reinhold
 * @since       JDK1.1
 */
public class FileReader extends InputStreamReader {

}
```

## Constructors

几个不同版本的构造函数，对应 `FileInputStream` 的不同版本的构造函数。最终目的都是初始化 `FileInputStream` 中的文件描述符并打开它。

```java
/**
 * Creates a new <tt>FileReader</tt>, given the name of the
 * file to read from.
 *
 * @param fileName the name of the file to read from
 * @exception  FileNotFoundException  if the named file does not exist,
 *                   is a directory rather than a regular file,
 *                   or for some other reason cannot be opened for
 *                   reading.
 */
public FileReader(String fileName) throws FileNotFoundException {
    super(new FileInputStream(fileName));
}

/**
 * Creates a new <tt>FileReader</tt>, given the <tt>File</tt>
 * to read from.
 *
 * @param file the <tt>File</tt> to read from
 * @exception  FileNotFoundException  if the file does not exist,
 *                   is a directory rather than a regular file,
 *                   or for some other reason cannot be opened for
 *                   reading.
 */
public FileReader(File file) throws FileNotFoundException {
    super(new FileInputStream(file));
}

/**
 * Creates a new <tt>FileReader</tt>, given the
 * <tt>FileDescriptor</tt> to read from.
 *
 * @param fd the FileDescriptor to read from
 */
public FileReader(FileDescriptor fd) {
    super(new FileInputStream(fd));
}
```

---

