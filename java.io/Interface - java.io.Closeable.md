# Interface - java.io.Closeable

Created by : Mr Dk.

2020 / 07 / 06 20:29

Nanjing, Jiangsu, China

---

## Definition

这个接口只定义了一个函数，用于释放对象持有的资源，比如打开的文件。

```java
/**
 * A {@code Closeable} is a source or destination of data that can be closed.
 * The close method is invoked to release resources that the object is
 * holding (such as open files).
 *
 * @since 1.5
 */
public interface Closeable extends AutoCloseable {

}
```

---

以下函数用于所有相关的系统资源。如果一个流已经被关闭，那么重复调用这个函数没有作用。

对于可能在关闭过程中会出错的场合，需要格外注意。在抛出 `IOException` 之前，强烈推荐回收底层资源，并在内部标记该对象为已关闭。

```java
/**
 * Closes this stream and releases any system resources associated
 * with it. If the stream is already closed then invoking this
 * method has no effect.
 *
 * <p> As noted in {@link AutoCloseable#close()}, cases where the
 * close may fail require careful attention. It is strongly advised
 * to relinquish the underlying resources and to internally
 * <em>mark</em> the {@code Closeable} as closed, prior to throwing
 * the {@code IOException}.
 *
 * @throws IOException if an I/O error occurs
 */
public void close() throws IOException;
```

---

