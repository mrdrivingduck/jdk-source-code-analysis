# Abstract Class - java.util.concurrent.atomic.AtomicIntegerFieldUpdater

Created by : Mr Dk.

2020 / 06 / 08 19:00

Nanjing, Jiangsu, China

---

## Definition

该类通过 **反射** 的方式，原子地更新某个指定类中的指定 `volatile int` 变量。这样可以使得一个类中的某个字段能够具备被原子操作的能力。

在使用时，给出一个类，以及类中某个 `volatile int` field 的名称字符串，该类中的函数可以通过反射找到指定的 field，然后调用类中的函数来原子地更新这个 field。

```java
/**
 * A reflection-based utility that enables atomic updates to
 * designated {@code volatile int} fields of designated classes.
 * This class is designed for use in atomic data structures in which
 * several fields of the same node are independently subject to atomic
 * updates.
 *
 * <p>Note that the guarantees of the {@code compareAndSet}
 * method in this class are weaker than in other atomic classes.
 * Because this class cannot ensure that all uses of the field
 * are appropriate for purposes of atomic access, it can
 * guarantee atomicity only with respect to other invocations of
 * {@code compareAndSet} and {@code set} on the same updater.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <T> The type of the object holding the updatable field
 */
public abstract class AtomicIntegerFieldUpdater<T> {

}
```

---

获得该类的实例。可以看到，需要传入的参数有目标类的 `Class` 对象，以及目标 field 的名称：

```java
/**
 * Creates and returns an updater for objects with the given field.
 * The Class argument is needed to check that reflective types and
 * generic types match.
 *
 * @param tclass the class of the objects holding the field
 * @param fieldName the name of the field to be updated
 * @param <U> the type of instances of tclass
 * @return the updater
 * @throws IllegalArgumentException if the field is not a
 * volatile integer type
 * @throws RuntimeException with a nested reflection-based
 * exception if the class does not hold field or is the wrong type,
 * or the field is inaccessible to the caller according to Java language
 * access control
 */
@CallerSensitive
public static <U> AtomicIntegerFieldUpdater<U> newUpdater(Class<U> tclass,
                                                            String fieldName) {
    return new AtomicIntegerFieldUpdaterImpl<U>
        (tclass, fieldName, Reflection.getCallerClass());
}

/**
 * Protected do-nothing constructor for use by subclasses.
 */
protected AtomicIntegerFieldUpdater() {
}
```

---

该类的剩余部分定义了与 `AtomicInteger` 相同的抽象或非抽象函数。其中，抽象的函数由一个内部实现类 `AtomicIntegerFieldUpdaterImpl` 实现：

```java
/**
 * Standard hotspot implementation using intrinsics.
 */
private static final class AtomicIntegerFieldUpdaterImpl<T>
    extends AtomicIntegerFieldUpdater<T> {
    private static final sun.misc.Unsafe U = sun.misc.Unsafe.getUnsafe();
    private final long offset;
    /**
     * if field is protected, the subclass constructing updater, else
     * the same as tclass
     */
    private final Class<?> cclass;
    /** class holding the field */
    private final Class<T> tclass;

    // ...
}
```

其中，构造函数主要完成 **鉴权** 的工作，判断指定的类中要被更新的 field 是否符合可以被更新的条件：

* 首先，通过反射取得 field
* 获取 field 的修饰符并验证是否可以被更新
* 验证包名 (子类或相邻包中的类才能更新 `protected` 修饰的 field)
* 确认被更新的 field 是 `volatile int`

```java
AtomicIntegerFieldUpdaterImpl(final Class<T> tclass,
                                final String fieldName,
                                final Class<?> caller) {
    final Field field;
    final int modifiers;
    try {
        field = AccessController.doPrivileged(
            new PrivilegedExceptionAction<Field>() {
                public Field run() throws NoSuchFieldException {
                    return tclass.getDeclaredField(fieldName);
                }
            });
        modifiers = field.getModifiers();
        sun.reflect.misc.ReflectUtil.ensureMemberAccess(
            caller, tclass, null, modifiers);
        ClassLoader cl = tclass.getClassLoader();
        ClassLoader ccl = caller.getClassLoader();
        if ((ccl != null) && (ccl != cl) &&
            ((cl == null) || !isAncestor(cl, ccl))) {
            sun.reflect.misc.ReflectUtil.checkPackageAccess(tclass);
        }
    } catch (PrivilegedActionException pae) {
        throw new RuntimeException(pae.getException());
    } catch (Exception ex) {
        throw new RuntimeException(ex);
    }

    if (field.getType() != int.class)
        throw new IllegalArgumentException("Must be integer type");

    if (!Modifier.isVolatile(modifiers))
        throw new IllegalArgumentException("Must be volatile type");

    // Access to protected field members is restricted to receivers only
    // of the accessing class, or one of its subclasses, and the
    // accessing class must in turn be a subclass (or package sibling)
    // of the protected member's defining class.
    // If the updater refers to a protected field of a declaring class
    // outside the current package, the receiver argument will be
    // narrowed to the type of the accessing class.
    this.cclass = (Modifier.isProtected(modifiers) &&
                    tclass.isAssignableFrom(caller) &&
                    !isSamePackage(tclass, caller))
                    ? caller : tclass;
    this.tclass = tclass;
    this.offset = U.objectFieldOffset(field);
}
```

以下子函数用于获取父子类的继承关系，以及包名：

```java
/**
 * Returns true if the second classloader can be found in the first
 * classloader's delegation chain.
 * Equivalent to the inaccessible: first.isAncestor(second).
 */
private static boolean isAncestor(ClassLoader first, ClassLoader second) {
    ClassLoader acl = first;
    do {
        acl = acl.getParent();
        if (second == acl) {
            return true;
        }
    } while (acl != null);
    return false;
}

/**
 * Returns true if the two classes have the same class loader and
 * package qualifier
 */
private static boolean isSamePackage(Class<?> class1, Class<?> class2) {
    return class1.getClassLoader() == class2.getClassLoader()
            && Objects.equals(getPackageName(class1), getPackageName(class2));
}

private static String getPackageName(Class<?> cls) {
    String cn = cls.getName();
    int dot = cn.lastIndexOf('.');
    return (dot != -1) ? cn.substring(0, dot) : "";
}
```

---

剩余函数基本与 `AtomicInteger` 中的函数相同。唯一的区别是，在调用 `Unsafe` 类封装的函数前，需要对目标类的 `Class` 对象实例进行类型验证。以 CAS 函数为例：

```java
/**
 * Checks that target argument is instance of cclass.  On
 * failure, throws cause.
 */
private final void accessCheck(T obj) {
    if (!cclass.isInstance(obj))
        throwAccessCheckException(obj);
}

/**
 * Throws access exception if accessCheck failed due to
 * protected access, else ClassCastException.
 */
private final void throwAccessCheckException(T obj) {
    if (cclass == tclass)
        throw new ClassCastException();
    else
        throw new RuntimeException(
            new IllegalAccessException(
                "Class " +
                cclass.getName() +
                " can not access a protected member of class " +
                tclass.getName() +
                " using an instance of " +
                obj.getClass().getName()));
}

public final boolean compareAndSet(T obj, int expect, int update) {
    accessCheck(obj);
    return U.compareAndSwapInt(obj, offset, expect, update);
}
```

---

