# Class - java.io.BufferedReader

Created by : Mr Dk.

2020 / 10 / 11 15:41

Nanjing, Jiangsu, China

---

## Definition

与 `java.io.BufferedInputStream` 功能类似，将输入来源中的数据存放在一个 buffer 中。当 buffer 中的数据被读完时，再从输入来源中继续读取。区别在于 buffer 的数据类型不同，一个是 `byte[]`，一个是 `char[]`。这个类的用于主要在于提升性能，推荐作为一些慢速 Reader 的外层封装，比如可以封装在 FileReader 外面：

```java
BufferedReader in = new BufferedReader(new FileReader("foo.in"));
```

这样，调用 `read()` 时，BufferedReader 会一次读取大量数据到 buffer 中，性能上会比 FileReader 每次读取一小部分要更好 (多次 I/O)。Buffer 的大小用户可以自行设置。

```java
/**
 * Reads text from a character-input stream, buffering characters so as to
 * provide for the efficient reading of characters, arrays, and lines.
 *
 * <p> The buffer size may be specified, or the default size may be used.  The
 * default is large enough for most purposes.
 *
 * <p> In general, each read request made of a Reader causes a corresponding
 * read request to be made of the underlying character or byte stream.  It is
 * therefore advisable to wrap a BufferedReader around any Reader whose read()
 * operations may be costly, such as FileReaders and InputStreamReaders.  For
 * example,
 *
 * <pre>
 * BufferedReader in
 *   = new BufferedReader(new FileReader("foo.in"));
 * </pre>
 *
 * will buffer the input from the specified file.  Without buffering, each
 * invocation of read() or readLine() could cause bytes to be read from the
 * file, converted into characters, and then returned, which can be very
 * inefficient.
 *
 * <p> Programs that use DataInputStreams for textual input can be localized by
 * replacing each DataInputStream with an appropriate BufferedReader.
 *
 * @see FileReader
 * @see InputStreamReader
 * @see java.nio.file.Files#newBufferedReader
 *
 * @author      Mark Reinhold
 * @since       JDK1.1
 */

public class BufferedReader extends Reader {

}
```

其中，与上述特性相关的内部成员变量：

* `in` 负责维护输入来源 (底层是 InputStream)
* `cb` 数组就是 buffer

```java
private Reader in;
private char cb[];
```

以下成员变量用于处理不同操作系统平台对于换行符处理的差异：

```java
/** If the next character is a line feed, skip it */
private boolean skipLF = false;
```

## Constructor

构造函数的目的显而易见：

* 将输入来源维护在对象内
* 初始化 buffer 数组至指定长度 (如果用户未指定，则使用默认长度 8192)

```java
private static int defaultCharBufferSize = 8192;

/**
 * Creates a buffering character-input stream that uses an input buffer of
 * the specified size.
 *
 * @param  in   A Reader
 * @param  sz   Input-buffer size
 *
 * @exception  IllegalArgumentException  If {@code sz <= 0}
 */
public BufferedReader(Reader in, int sz) {
    super(in);
    if (sz <= 0)
        throw new IllegalArgumentException("Buffer size <= 0");
    this.in = in;
    cb = new char[sz];
    nextChar = nChars = 0;
}

/**
 * Creates a buffering character-input stream that uses a default-sized
 * input buffer.
 *
 * @param  in   A Reader
 */
public BufferedReader(Reader in) {
    this(in, defaultCharBufferSize);
}
```

## Fill

这个函数用于填充 buffer，当 buffer 中的字符已经不够读取时被调用。填充 buffer 的方法显而易见：从输入来源中调用 `read()` 即可。当然，需要根据当前对象是否启用了 mark 功能，以及 buffer 中的剩余空间，来决定从输入来源中读取的字节数。

其中，`dst` 变量指示了从输入流中读取字符后，存放到 buffer 中的位置；`nextChar` 指示下一个从当前 Reader 中被读取出的字符，`nChars` 指代 buffer 中最后一个有效字节的位置。如果当前 Reader 没有启用 mark，那么 buffer 中原有的内容已经全部被读取完毕，新读入的字符可以直接从 buffer 的头部开始存放，因此 `dst` 是 `0`。否则，至少需要保留从 `markedChar` 开始的字符。

如果说需要保留的字符已经超出了 `readAheadLimit`，那么 mark 就相当于作废了，也就是说 buffer 已经无法做到在保留原有 mark 字符的同时读取新的字符。如果没有超出 `readAheadLimit`，那么再判断这个值有没有超出 buffer 的长度：

* 如果没有超出，那么将 mark 位置开始的所有字符搬运到 buffer 开头，相当于丢弃 mark 位置之前的所有字符
* 如果超出，那么重新分配 buffer 数组，使其长度符合 `readAheadLimit`，然后再将 mark 位置开始的所有字符搬运到 buffer 开头

把原有的字符搬到 buffer 开头后，之后空出的位置就可以用于存放调用 `read()` 从输入流中获得的字符。

```java
private static final int INVALIDATED = -2;
private static final int UNMARKED = -1;
private int markedChar = UNMARKED;
private int readAheadLimit = 0; /* Valid only when markedChar > 0 */

/**
 * Fills the input buffer, taking the mark into account if it is valid.
 */
private void fill() throws IOException {
    int dst;
    if (markedChar <= UNMARKED) {
        /* No mark */
        dst = 0;
    } else {
        /* Marked */
        int delta = nextChar - markedChar;
        if (delta >= readAheadLimit) {
            /* Gone past read-ahead limit: Invalidate mark */
            markedChar = INVALIDATED;
            readAheadLimit = 0;
            dst = 0;
        } else {
            if (readAheadLimit <= cb.length) {
                /* Shuffle in the current buffer */
                System.arraycopy(cb, markedChar, cb, 0, delta);
                markedChar = 0;
                dst = delta;
            } else {
                /* Reallocate buffer to accommodate read-ahead limit */
                char ncb[] = new char[readAheadLimit];
                System.arraycopy(cb, markedChar, ncb, 0, delta);
                cb = ncb;
                markedChar = 0;
                dst = delta;
            }
            nextChar = nChars = delta;
        }
    }

    int n;
    do {
        n = in.read(cb, dst, cb.length - dst);
    } while (n == 0);
    if (n > 0) {
        nChars = dst + n;
        nextChar = dst;
    }
}
```

## Read

以下函数从当前 Reader 中读取一个字符。在确保输入流不为空的前提下，返回 buffer 中的下一个字符。如果下一个字符已经超出了 buffer 的维护范围，那么调用一次 `fill()` 填充 buffer：

```java
/**
 * Reads a single character.
 *
 * @return The character read, as an integer in the range
 *         0 to 65535 (<tt>0x00-0xffff</tt>), or -1 if the
 *         end of the stream has been reached
 * @exception  IOException  If an I/O error occurs
 */
public int read() throws IOException {
    synchronized (lock) {
        ensureOpen();
        for (;;) {
            if (nextChar >= nChars) {
                fill();
                if (nextChar >= nChars)
                    return -1;
            }
            if (skipLF) {
                skipLF = false;
                if (cb[nextChar] == '\n') {
                    nextChar++;
                    continue;
                }
            }
            return cb[nextChar++];
        }
    }
}
```

以下函数读取一段字符到指定数组内，是内部函数。如果读取的长度超出了 buffer 的长度且没有 mark/reset，那么直接调用底层 Reader 的 `read()`。如果剩余要读的字符不够了，则调用一次 `fill()` 填充 buffer。然后将 buffer 中的指定个数字符复制到目标数组中，并返回实际复制的字符数：

```java
/**
 * Reads characters into a portion of an array, reading from the underlying
 * stream if necessary.
 */
private int read1(char[] cbuf, int off, int len) throws IOException {
    if (nextChar >= nChars) {
        /* If the requested length is at least as large as the buffer, and
            if there is no mark/reset activity, and if line feeds are not
            being skipped, do not bother to copy the characters into the
            local buffer.  In this way buffered streams will cascade
            harmlessly. */
        if (len >= cb.length && markedChar <= UNMARKED && !skipLF) {
            return in.read(cbuf, off, len);
        }
        fill();
    }
    if (nextChar >= nChars) return -1;
    if (skipLF) {
        skipLF = false;
        if (cb[nextChar] == '\n') {
            nextChar++;
            if (nextChar >= nChars)
                fill();
            if (nextChar >= nChars)
                return -1;
        }
    }
    int n = Math.min(len, nChars - nextChar);
    System.arraycopy(cb, nextChar, cbuf, off, n);
    nextChar += n;
    return n;
}
```

以下函数是暴露给用户的外部函数，基本功能与上述内部函数一致，但是加入了同步操作，在保证输入流开启的前提下，循环调用上述内部函数，尽可能读取到用户指定的字符长度并返回用户。

```java
/**
 * Reads characters into a portion of an array.
 *
 * <p> This method implements the general contract of the corresponding
 * <code>{@link Reader#read(char[], int, int) read}</code> method of the
 * <code>{@link Reader}</code> class.  As an additional convenience, it
 * attempts to read as many characters as possible by repeatedly invoking
 * the <code>read</code> method of the underlying stream.  This iterated
 * <code>read</code> continues until one of the following conditions becomes
 * true: <ul>
 *
 *   <li> The specified number of characters have been read,
 *
 *   <li> The <code>read</code> method of the underlying stream returns
 *   <code>-1</code>, indicating end-of-file, or
 *
 *   <li> The <code>ready</code> method of the underlying stream
 *   returns <code>false</code>, indicating that further input requests
 *   would block.
 *
 * </ul> If the first <code>read</code> on the underlying stream returns
 * <code>-1</code> to indicate end-of-file then this method returns
 * <code>-1</code>.  Otherwise this method returns the number of characters
 * actually read.
 *
 * <p> Subclasses of this class are encouraged, but not required, to
 * attempt to read as many characters as possible in the same fashion.
 *
 * <p> Ordinarily this method takes characters from this stream's character
 * buffer, filling it from the underlying stream as necessary.  If,
 * however, the buffer is empty, the mark is not valid, and the requested
 * length is at least as large as the buffer, then this method will read
 * characters directly from the underlying stream into the given array.
 * Thus redundant <code>BufferedReader</code>s will not copy data
 * unnecessarily.
 *
 * @param      cbuf  Destination buffer
 * @param      off   Offset at which to start storing characters
 * @param      len   Maximum number of characters to read
 *
 * @return     The number of characters read, or -1 if the end of the
 *             stream has been reached
 *
 * @exception  IOException  If an I/O error occurs
 */
public int read(char cbuf[], int off, int len) throws IOException {
    synchronized (lock) {
        ensureOpen();
        if ((off < 0) || (off > cbuf.length) || (len < 0) ||
            ((off + len) > cbuf.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        }

        int n = read1(cbuf, off, len);
        if (n <= 0) return n;
        while ((n < len) && in.ready()) {
            int n1 = read1(cbuf, off + n, len - n);
            if (n1 <= 0) break;
            n += n1;
        }
        return n;
    }
}
```

以下函数返回一行字符串。该函数将 `\n` (Line Feed, LF) (Unix)、`\r` (Carriage Return, CR, 显示为 `^M`) (OSX)、`\r\n` (CRLF) (Windows) 中的任意一个视为行分隔符。[区别](https://www.cnblogs.com/xiaotiannet/p/3510586.html)。函数参数中的 `boolean` 变量决定了在返回的字符串中是否忽略最后的换行符，默认保留换行符。

内部实现会建立一个默认的 StringBuffer，默认一行的长度为 `defaultExpectedLineLength` (80)，然后在一个循环中不断读取字符，并 append 到 StringBuffer 后，直到识别出换行符为止。

```java
private static int defaultExpectedLineLength = 80;

/**
 * Reads a line of text.  A line is considered to be terminated by any one
 * of a line feed ('\n'), a carriage return ('\r'), or a carriage return
 * followed immediately by a linefeed.
 *
 * @param      ignoreLF  If true, the next '\n' will be skipped
 *
 * @return     A String containing the contents of the line, not including
 *             any line-termination characters, or null if the end of the
 *             stream has been reached
 *
 * @see        java.io.LineNumberReader#readLine()
 *
 * @exception  IOException  If an I/O error occurs
 */
String readLine(boolean ignoreLF) throws IOException {
    StringBuffer s = null;
    int startChar;

    synchronized (lock) {
        ensureOpen();
        boolean omitLF = ignoreLF || skipLF;

    bufferLoop:
        for (;;) {

            if (nextChar >= nChars)
                fill();
            if (nextChar >= nChars) { /* EOF */
                if (s != null && s.length() > 0)
                    return s.toString();
                else
                    return null;
            }
            boolean eol = false;
            char c = 0;
            int i;

            /* Skip a leftover '\n', if necessary */
            if (omitLF && (cb[nextChar] == '\n'))
                nextChar++;
            skipLF = false;
            omitLF = false;

        charLoop:
            for (i = nextChar; i < nChars; i++) {
                c = cb[i];
                if ((c == '\n') || (c == '\r')) {
                    eol = true;
                    break charLoop;
                }
            }

            startChar = nextChar;
            nextChar = i;

            if (eol) {
                String str;
                if (s == null) {
                    str = new String(cb, startChar, i - startChar);
                } else {
                    s.append(cb, startChar, i - startChar);
                    str = s.toString();
                }
                nextChar++;
                if (c == '\r') {
                    skipLF = true;
                }
                return str;
            }

            if (s == null)
                s = new StringBuffer(defaultExpectedLineLength);
            s.append(cb, startChar, i - startChar);
        }
    }
}

/**
 * Reads a line of text.  A line is considered to be terminated by any one
 * of a line feed ('\n'), a carriage return ('\r'), or a carriage return
 * followed immediately by a linefeed.
 *
 * @return     A String containing the contents of the line, not including
 *             any line-termination characters, or null if the end of the
 *             stream has been reached
 *
 * @exception  IOException  If an I/O error occurs
 *
 * @see java.nio.file.Files#readAllLines
 */
public String readLine() throws IOException {
    return readLine(false);
}
```

以下函数返回一个可以被迭代器遍历的流，其中的内容就是每一行字符串：

```java
/**
 * Returns a {@code Stream}, the elements of which are lines read from
 * this {@code BufferedReader}.  The {@link Stream} is lazily populated,
 * i.e., read only occurs during the
 * <a href="../util/stream/package-summary.html#StreamOps">terminal
 * stream operation</a>.
 *
 * <p> The reader must not be operated on during the execution of the
 * terminal stream operation. Otherwise, the result of the terminal stream
 * operation is undefined.
 *
 * <p> After execution of the terminal stream operation there are no
 * guarantees that the reader will be at a specific position from which to
 * read the next character or line.
 *
 * <p> If an {@link IOException} is thrown when accessing the underlying
 * {@code BufferedReader}, it is wrapped in an {@link
 * UncheckedIOException} which will be thrown from the {@code Stream}
 * method that caused the read to take place. This method will return a
 * Stream if invoked on a BufferedReader that is closed. Any operation on
 * that stream that requires reading from the BufferedReader after it is
 * closed, will cause an UncheckedIOException to be thrown.
 *
 * @return a {@code Stream<String>} providing the lines of text
 *         described by this {@code BufferedReader}
 *
 * @since 1.8
 */
public Stream<String> lines() {
    Iterator<String> iter = new Iterator<String>() {
        String nextLine = null;

        @Override
        public boolean hasNext() {
            if (nextLine != null) {
                return true;
            } else {
                try {
                    nextLine = readLine();
                    return (nextLine != null);
                } catch (IOException e) {
                    throw new UncheckedIOException(e);
                }
            }
        }

        @Override
        public String next() {
            if (nextLine != null || hasNext()) {
                String line = nextLine;
                nextLine = null;
                return line;
            } else {
                throw new NoSuchElementException();
            }
        }
    };
    return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
            iter, Spliterator.ORDERED | Spliterator.NONNULL), false);
}
```

## Skip

跳过一些字节。当 buffer 中的有效字节用尽时，调用一次 `fill()`。

```java
/**
 * Skips characters.
 *
 * @param  n  The number of characters to skip
 *
 * @return    The number of characters actually skipped
 *
 * @exception  IllegalArgumentException  If <code>n</code> is negative.
 * @exception  IOException  If an I/O error occurs
 */
public long skip(long n) throws IOException {
    if (n < 0L) {
        throw new IllegalArgumentException("skip value is negative");
    }
    synchronized (lock) {
        ensureOpen();
        long r = n;
        while (r > 0) {
            if (nextChar >= nChars)
                fill();
            if (nextChar >= nChars) /* EOF */
                break;
            if (skipLF) {
                skipLF = false;
                if (cb[nextChar] == '\n') {
                    nextChar++;
                }
            }
            long d = nChars - nextChar;
            if (r <= d) {
                nextChar += r;
                r = 0;
                break;
            }
            else {
                r -= d;
                nextChar = nChars;
            }
        }
        return n - r;
    }
}
```

## Ready

当 buffer 中还可以有字符被读取或底层的输入流准备就绪 (调用 `read()` 不会阻塞) 时，当前函数返回 `true`，表示当前 Reader 准备就绪。默认 LF 会被跳过，不会被当作新一行的字符，保证只有当下一行字符准备就绪时，当前函数才返回 `true`。

```java
/**
 * Tells whether this stream is ready to be read.  A buffered character
 * stream is ready if the buffer is not empty, or if the underlying
 * character stream is ready.
 *
 * @exception  IOException  If an I/O error occurs
 */
public boolean ready() throws IOException {
    synchronized (lock) {
        ensureOpen();

        /*
         * If newline needs to be skipped and the next char to be read
         * is a newline character, then just skip it right away.
         */
        if (skipLF) {
            /* Note that in.ready() will return true if and only if the next
             * read on the stream will not block.
             */
            if (nextChar >= nChars && in.ready()) {
                fill();
            }
            if (nextChar < nChars) {
                if (cb[nextChar] == '\n')
                    nextChar++;
                skipLF = false;
            }
        }
        return (nextChar < nChars) || in.ready();
    }
}
```

## Mark / Reset

这个类支持 mark / reset 操作。在调用 `mark()` 时，需要指定 `readAheadLimit`。当 mark 的位置距离当前 read 的位置超过这个阈值时，reset 将无法回到 mark 所在位置而失败 (IOException)。如果这个值设置为超过 buffer 长度时，buffer 将会被扩展到不小于这个长度。

```java
/**
 * Tells whether this stream supports the mark() operation, which it does.
 */
public boolean markSupported() {
    return true;
}

/**
 * Marks the present position in the stream.  Subsequent calls to reset()
 * will attempt to reposition the stream to this point.
 *
 * @param readAheadLimit   Limit on the number of characters that may be
 *                         read while still preserving the mark. An attempt
 *                         to reset the stream after reading characters
 *                         up to this limit or beyond may fail.
 *                         A limit value larger than the size of the input
 *                         buffer will cause a new buffer to be allocated
 *                         whose size is no smaller than limit.
 *                         Therefore large values should be used with care.
 *
 * @exception  IllegalArgumentException  If {@code readAheadLimit < 0}
 * @exception  IOException  If an I/O error occurs
 */
public void mark(int readAheadLimit) throws IOException {
    if (readAheadLimit < 0) {
        throw new IllegalArgumentException("Read-ahead limit < 0");
    }
    synchronized (lock) {
        ensureOpen();
        this.readAheadLimit = readAheadLimit;
        markedChar = nextChar;
        markedSkipLF = skipLF;
    }
}

/**
 * Resets the stream to the most recent mark.
 *
 * @exception  IOException  If the stream has never been marked,
 *                          or if the mark has been invalidated
 */
public void reset() throws IOException {
    synchronized (lock) {
        ensureOpen();
        if (markedChar < 0)
            throw new IOException((markedChar == INVALIDATED)
                                    ? "Mark invalid"
                                    : "Stream not marked");
        nextChar = markedChar;
        skipLF = markedSkipLF;
    }
}
```

## Close

关闭底层的 Reader：

```java
public void close() throws IOException {
    synchronized (lock) {
        if (in == null)
            return;
        try {
            in.close();
        } finally {
            in = null;
            cb = null;
        }
    }
}
```

---

