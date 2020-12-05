# Class - java.io.FileOutputStream

Created by : Mr Dk.

2020 / 12 / 05 13:09

Nanjing, Jiangsu, China

---

## Definition

这个输出流用于向文件对象或文件描述符对象中输出字节数据，如果想要输出字符，则考虑使用 `FileWriter`。一个文件是否可用取决于底层 OS 的实现 (有些 OS 只允许一个输出流向文件中输出)。

```java
/**
 * A file output stream is an output stream for writing data to a
 * <code>File</code> or to a <code>FileDescriptor</code>. Whether or not
 * a file is available or may be created depends upon the underlying
 * platform.  Some platforms, in particular, allow a file to be opened
 * for writing by only one <tt>FileOutputStream</tt> (or other
 * file-writing object) at a time.  In such situations the constructors in
 * this class will fail if the file involved is already open.
 *
 * <p><code>FileOutputStream</code> is meant for writing streams of raw bytes
 * such as image data. For writing streams of characters, consider using
 * <code>FileWriter</code>.
 *
 * @author  Arthur van Hoff
 * @see     java.io.File
 * @see     java.io.FileDescriptor
 * @see     java.io.FileInputStream
 * @see     java.nio.file.Files#newOutputStream
 * @since   JDK1.0
 */
public
class FileOutputStream extends OutputStream
{

}
```

## Fields

内部维护的文件信息，包括文件描述符、文件路径等。另外，还有一个是否以 `append` 模式打开文件的标志。

```java
/**
 * The system dependent file descriptor.
 */
private final FileDescriptor fd;

/**
 * True if the file is opened for append.
 */
private final boolean append;

/**
 * The associated channel, initialized lazily.
 */
private FileChannel channel;

/**
 * The path of the referenced file
 * (null if the stream is created with a file descriptor)
 */
private final String path;
```

## Constructor

构造函数支持三种类型的参数：

* 文件名字符串
* `File` 对象
* 已经打开的文件描述符对象

对于前两种形式，构造函数不仅需要把信息保存在类中，还需要调用 JVM 的 native `open()` 函数打开文件。

```java
/**
 * Creates a file output stream to write to the file with the
 * specified name. A new <code>FileDescriptor</code> object is
 * created to represent this file connection.
 * <p>
 * First, if there is a security manager, its <code>checkWrite</code>
 * method is called with <code>name</code> as its argument.
 * <p>
 * If the file exists but is a directory rather than a regular file, does
 * not exist but cannot be created, or cannot be opened for any other
 * reason then a <code>FileNotFoundException</code> is thrown.
 *
 * @param      name   the system-dependent filename
 * @exception  FileNotFoundException  if the file exists but is a directory
 *                   rather than a regular file, does not exist but cannot
 *                   be created, or cannot be opened for any other reason
 * @exception  SecurityException  if a security manager exists and its
 *               <code>checkWrite</code> method denies write access
 *               to the file.
 * @see        java.lang.SecurityManager#checkWrite(java.lang.String)
 */
public FileOutputStream(String name) throws FileNotFoundException {
    this(name != null ? new File(name) : null, false);
}

/**
 * Creates a file output stream to write to the file with the specified
 * name.  If the second argument is <code>true</code>, then
 * bytes will be written to the end of the file rather than the beginning.
 * A new <code>FileDescriptor</code> object is created to represent this
 * file connection.
 * <p>
 * First, if there is a security manager, its <code>checkWrite</code>
 * method is called with <code>name</code> as its argument.
 * <p>
 * If the file exists but is a directory rather than a regular file, does
 * not exist but cannot be created, or cannot be opened for any other
 * reason then a <code>FileNotFoundException</code> is thrown.
 *
 * @param     name        the system-dependent file name
 * @param     append      if <code>true</code>, then bytes will be written
 *                   to the end of the file rather than the beginning
 * @exception  FileNotFoundException  if the file exists but is a directory
 *                   rather than a regular file, does not exist but cannot
 *                   be created, or cannot be opened for any other reason.
 * @exception  SecurityException  if a security manager exists and its
 *               <code>checkWrite</code> method denies write access
 *               to the file.
 * @see        java.lang.SecurityManager#checkWrite(java.lang.String)
 * @since     JDK1.1
 */
public FileOutputStream(String name, boolean append)
    throws FileNotFoundException
{
    this(name != null ? new File(name) : null, append);
}

/**
 * Creates a file output stream to write to the file represented by
 * the specified <code>File</code> object. A new
 * <code>FileDescriptor</code> object is created to represent this
 * file connection.
 * <p>
 * First, if there is a security manager, its <code>checkWrite</code>
 * method is called with the path represented by the <code>file</code>
 * argument as its argument.
 * <p>
 * If the file exists but is a directory rather than a regular file, does
 * not exist but cannot be created, or cannot be opened for any other
 * reason then a <code>FileNotFoundException</code> is thrown.
 *
 * @param      file               the file to be opened for writing.
 * @exception  FileNotFoundException  if the file exists but is a directory
 *                   rather than a regular file, does not exist but cannot
 *                   be created, or cannot be opened for any other reason
 * @exception  SecurityException  if a security manager exists and its
 *               <code>checkWrite</code> method denies write access
 *               to the file.
 * @see        java.io.File#getPath()
 * @see        java.lang.SecurityException
 * @see        java.lang.SecurityManager#checkWrite(java.lang.String)
 */
public FileOutputStream(File file) throws FileNotFoundException {
    this(file, false);
}

/**
 * Creates a file output stream to write to the file represented by
 * the specified <code>File</code> object. If the second argument is
 * <code>true</code>, then bytes will be written to the end of the file
 * rather than the beginning. A new <code>FileDescriptor</code> object is
 * created to represent this file connection.
 * <p>
 * First, if there is a security manager, its <code>checkWrite</code>
 * method is called with the path represented by the <code>file</code>
 * argument as its argument.
 * <p>
 * If the file exists but is a directory rather than a regular file, does
 * not exist but cannot be created, or cannot be opened for any other
 * reason then a <code>FileNotFoundException</code> is thrown.
 *
 * @param      file               the file to be opened for writing.
 * @param     append      if <code>true</code>, then bytes will be written
 *                   to the end of the file rather than the beginning
 * @exception  FileNotFoundException  if the file exists but is a directory
 *                   rather than a regular file, does not exist but cannot
 *                   be created, or cannot be opened for any other reason
 * @exception  SecurityException  if a security manager exists and its
 *               <code>checkWrite</code> method denies write access
 *               to the file.
 * @see        java.io.File#getPath()
 * @see        java.lang.SecurityException
 * @see        java.lang.SecurityManager#checkWrite(java.lang.String)
 * @since 1.4
 */
public FileOutputStream(File file, boolean append)
    throws FileNotFoundException
{
    String name = (file != null ? file.getPath() : null);
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkWrite(name);
    }
    if (name == null) {
        throw new NullPointerException();
    }
    if (file.isInvalid()) {
        throw new FileNotFoundException("Invalid file path");
    }
    this.fd = new FileDescriptor();
    fd.attach(this);
    this.append = append;
    this.path = name;

    open(name, append);
}

/**
 * Creates a file output stream to write to the specified file
 * descriptor, which represents an existing connection to an actual
 * file in the file system.
 * <p>
 * First, if there is a security manager, its <code>checkWrite</code>
 * method is called with the file descriptor <code>fdObj</code>
 * argument as its argument.
 * <p>
 * If <code>fdObj</code> is null then a <code>NullPointerException</code>
 * is thrown.
 * <p>
 * This constructor does not throw an exception if <code>fdObj</code>
 * is {@link java.io.FileDescriptor#valid() invalid}.
 * However, if the methods are invoked on the resulting stream to attempt
 * I/O on the stream, an <code>IOException</code> is thrown.
 *
 * @param      fdObj   the file descriptor to be opened for writing
 * @exception  SecurityException  if a security manager exists and its
 *               <code>checkWrite</code> method denies
 *               write access to the file descriptor
 * @see        java.lang.SecurityManager#checkWrite(java.io.FileDescriptor)
 */
public FileOutputStream(FileDescriptor fdObj) {
    SecurityManager security = System.getSecurityManager();
    if (fdObj == null) {
        throw new NullPointerException();
    }
    if (security != null) {
        security.checkWrite(fdObj);
    }
    this.fd = fdObj;
    this.append = false;
    this.path = null;

    fd.attach(this);
}
```

```java
/**
 * Opens a file, with the specified name, for overwriting or appending.
 * @param name name of file to be opened
 * @param append whether the file is to be opened in append mode
 */
private native void open0(String name, boolean append)
    throws FileNotFoundException;

// wrap native call to allow instrumentation
/**
 * Opens a file, with the specified name, for overwriting or appending.
 * @param name name of file to be opened
 * @param append whether the file is to be opened in append mode
 */
private void open(String name, boolean append)
    throws FileNotFoundException {
    open0(name, append);
}
```

## Write

向文件中的写入操作基本都由 JVM native 函数支持，包含写一个字节和写多个字节的函数。

```java
/**
 * Writes the specified byte to this file output stream.
 *
 * @param   b   the byte to be written.
 * @param   append   {@code true} if the write operation first
 *     advances the position to the end of file
 */
private native void write(int b, boolean append) throws IOException;

/**
 * Writes a sub array as a sequence of bytes.
 * @param b the data to be written
 * @param off the start offset in the data
 * @param len the number of bytes that are written
 * @param append {@code true} to first advance the position to the
 *     end of file
 * @exception IOException If an I/O error has occurred.
 */
private native void writeBytes(byte b[], int off, int len, boolean append)
    throws IOException;
```

该类对这些函数做了封装：

```java
/**
 * Writes the specified byte to this file output stream. Implements
 * the <code>write</code> method of <code>OutputStream</code>.
 *
 * @param      b   the byte to be written.
 * @exception  IOException  if an I/O error occurs.
 */
public void write(int b) throws IOException {
    write(b, append);
}

/**
 * Writes <code>b.length</code> bytes from the specified byte array
 * to this file output stream.
 *
 * @param      b   the data.
 * @exception  IOException  if an I/O error occurs.
 */
public void write(byte b[]) throws IOException {
    writeBytes(b, 0, b.length, append);
}

/**
 * Writes <code>len</code> bytes from the specified byte array
 * starting at offset <code>off</code> to this file output stream.
 *
 * @param      b     the data.
 * @param      off   the start offset in the data.
 * @param      len   the number of bytes to write.
 * @exception  IOException  if an I/O error occurs.
 */
public void write(byte b[], int off, int len) throws IOException {
    writeBytes(b, off, len, append);
}
```

---

