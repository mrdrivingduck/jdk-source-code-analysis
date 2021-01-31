# Class - java.lang.Integer

Created by : Mr Dk.

2020 / 05 / 29 11:15

Nanjing, Jiangsu, China

---

## Definition

```java
/**
 * The {@code Integer} class wraps a value of the primitive type
 * {@code int} in an object. An object of type {@code Integer}
 * contains a single field whose type is {@code int}.
 *
 * <p>In addition, this class provides several methods for converting
 * an {@code int} to a {@code String} and a {@code String} to an
 * {@code int}, as well as other constants and methods useful when
 * dealing with an {@code int}.
 *
 * <p>Implementation note: The implementations of the "bit twiddling"
 * methods (such as {@link #highestOneBit(int) highestOneBit} and
 * {@link #numberOfTrailingZeros(int) numberOfTrailingZeros}) are
 * based on material from Henry S. Warren, Jr.'s <i>Hacker's
 * Delight</i>, (Addison Wesley, 2002).
 *
 * @author  Lee Boynton
 * @author  Arthur van Hoff
 * @author  Josh Bloch
 * @author  Joseph D. Darcy
 * @since JDK1.0
 */
public final class Integer extends Number implements Comparable<Integer> {

}
```

`Integer` 类将一个原始类型的 `int` 封装在对象中。类中的函数主要分为两部分：

* 静态工具函数，负责 `int` 和 `String` 之间的互相转换
* 对象函数，负责有效处理 `Integer` 中封装的 `int` 值

---

内部 `int` 数值的范围为 `-2^31 ~ 2^31-1`。

```java
/**
 * A constant holding the minimum value an {@code int} can
 * have, -2<sup>31</sup>.
 */
@Native public static final int   MIN_VALUE = 0x80000000;

/**
 * A constant holding the maximum value an {@code int} can
 * have, 2<sup>31</sup>-1.
 */
@Native public static final int   MAX_VALUE = 0x7fffffff;
```

---

`Integer` 类中提供的一个重要功能，就是 `int` 与 `String` 类之间的转换。这个功能提供的函数基本是 `static`，与 `Integer` 本身的状态无关，仅提供代码功能。这些函数接收的外部参数不是 `int` 就是 `String`。其中，字符串所对应的 **进制** 可以通过参数来指定。

首先缓存了字符串中可能出现的所有字符。如果输入的进制 `radix` 为 N，那么前 N 个字符将被用于表示这个字符串。比如对于二进制，就用前两个字符 `0` `1`。进制数如果没有特别指定，默认为十进制；指定的范围为 `2` 到 `36`。

```java
/**
 * All possible chars for representing a number as a String
 */
final static char[] digits = {
    '0' , '1' , '2' , '3' , '4' , '5' ,
    '6' , '7' , '8' , '9' , 'a' , 'b' ,
    'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
    'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
    'o' , 'p' , 'q' , 'r' , 's' , 't' ,
    'u' , 'v' , 'w' , 'x' , 'y' , 'z'
};
```

以下函数将一个输入的 `int` 按输入的进制 `radix` 转换为对应的字符串。如果对应一个负数，那么先将其转换为相反数进行转换，最终在字符串最前面将被添上 `-`。整体过程就是经典的 *除以进制数取余数*，倒序构造字符串的每一个字符。如果确定转换十进制字符串，那么调用另一个专为十进制设计的快速版本。

```java
/**
 * Returns a string representation of the first argument in the
 * radix specified by the second argument.
 *
 * <p>If the radix is smaller than {@code Character.MIN_RADIX}
 * or larger than {@code Character.MAX_RADIX}, then the radix
 * {@code 10} is used instead.
 *
 * <p>If the first argument is negative, the first element of the
 * result is the ASCII minus character {@code '-'}
 * ({@code '\u005Cu002D'}). If the first argument is not
 * negative, no sign character appears in the result.
 *
 * <p>The remaining characters of the result represent the magnitude
 * of the first argument. If the magnitude is zero, it is
 * represented by a single zero character {@code '0'}
 * ({@code '\u005Cu0030'}); otherwise, the first character of
 * the representation of the magnitude will not be the zero
 * character.  The following ASCII characters are used as digits:
 *
 * <blockquote>
 *   {@code 0123456789abcdefghijklmnopqrstuvwxyz}
 * </blockquote>
 *
 * These are {@code '\u005Cu0030'} through
 * {@code '\u005Cu0039'} and {@code '\u005Cu0061'} through
 * {@code '\u005Cu007A'}. If {@code radix} is
 * <var>N</var>, then the first <var>N</var> of these characters
 * are used as radix-<var>N</var> digits in the order shown. Thus,
 * the digits for hexadecimal (radix 16) are
 * {@code 0123456789abcdef}. If uppercase letters are
 * desired, the {@link java.lang.String#toUpperCase()} method may
 * be called on the result:
 *
 * <blockquote>
 *  {@code Integer.toString(n, 16).toUpperCase()}
 * </blockquote>
 *
 * @param   i       an integer to be converted to a string.
 * @param   radix   the radix to use in the string representation.
 * @return  a string representation of the argument in the specified radix.
 * @see     java.lang.Character#MAX_RADIX
 * @see     java.lang.Character#MIN_RADIX
 */
public static String toString(int i, int radix) {
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
        radix = 10;

    /* Use the faster version */
    if (radix == 10) {
        return toString(i);
    }

    char buf[] = new char[33];
    boolean negative = (i < 0);
    int charPos = 32;

    if (!negative) {
        i = -i;
    }

    while (i <= -radix) {
        buf[charPos--] = digits[-(i % radix)];
        i = i / radix;
    }
    buf[charPos] = digits[-i];

    if (negative) {
        buf[--charPos] = '-';
    }

    return new String(buf, charPos, (33 - charPos));
}
```

另外，还可以按进制将 `int` 转换为无符号的字符串。在 `Integer` 类中，只支持二进制、八进制、十六进制。其它进制的支持位于 `Long` 类中：

```java
/**
 * Returns a string representation of the first argument as an
 * unsigned integer value in the radix specified by the second
 * argument.
 *
 * <p>If the radix is smaller than {@code Character.MIN_RADIX}
 * or larger than {@code Character.MAX_RADIX}, then the radix
 * {@code 10} is used instead.
 *
 * <p>Note that since the first argument is treated as an unsigned
 * value, no leading sign character is printed.
 *
 * <p>If the magnitude is zero, it is represented by a single zero
 * character {@code '0'} ({@code '\u005Cu0030'}); otherwise,
 * the first character of the representation of the magnitude will
 * not be the zero character.
 *
 * <p>The behavior of radixes and the characters used as digits
 * are the same as {@link #toString(int, int) toString}.
 *
 * @param   i       an integer to be converted to an unsigned string.
 * @param   radix   the radix to use in the string representation.
 * @return  an unsigned string representation of the argument in the specified radix.
 * @see     #toString(int, int)
 * @since 1.8
 */
public static String toUnsignedString(int i, int radix) {
    return Long.toUnsignedString(toUnsignedLong(i), radix);
}

/**
 * Returns a string representation of the integer argument as an
 * unsigned integer in base&nbsp;16.
 *
 * <p>The unsigned integer value is the argument plus 2<sup>32</sup>
 * if the argument is negative; otherwise, it is equal to the
 * argument.  This value is converted to a string of ASCII digits
 * in hexadecimal (base&nbsp;16) with no extra leading
 * {@code 0}s.
 *
 * <p>The value of the argument can be recovered from the returned
 * string {@code s} by calling {@link
 * Integer#parseUnsignedInt(String, int)
 * Integer.parseUnsignedInt(s, 16)}.
 *
 * <p>If the unsigned magnitude is zero, it is represented by a
 * single zero character {@code '0'} ({@code '\u005Cu0030'});
 * otherwise, the first character of the representation of the
 * unsigned magnitude will not be the zero character. The
 * following characters are used as hexadecimal digits:
 *
 * <blockquote>
 *  {@code 0123456789abcdef}
 * </blockquote>
 *
 * These are the characters {@code '\u005Cu0030'} through
 * {@code '\u005Cu0039'} and {@code '\u005Cu0061'} through
 * {@code '\u005Cu0066'}. If uppercase letters are
 * desired, the {@link java.lang.String#toUpperCase()} method may
 * be called on the result:
 *
 * <blockquote>
 *  {@code Integer.toHexString(n).toUpperCase()}
 * </blockquote>
 *
 * @param   i   an integer to be converted to a string.
 * @return  the string representation of the unsigned integer value
 *          represented by the argument in hexadecimal (base&nbsp;16).
 * @see #parseUnsignedInt(String, int)
 * @see #toUnsignedString(int, int)
 * @since   JDK1.0.2
 */
public static String toHexString(int i) {
    return toUnsignedString0(i, 4);
}

/**
 * Returns a string representation of the integer argument as an
 * unsigned integer in base&nbsp;8.
 *
 * <p>The unsigned integer value is the argument plus 2<sup>32</sup>
 * if the argument is negative; otherwise, it is equal to the
 * argument.  This value is converted to a string of ASCII digits
 * in octal (base&nbsp;8) with no extra leading {@code 0}s.
 *
 * <p>The value of the argument can be recovered from the returned
 * string {@code s} by calling {@link
 * Integer#parseUnsignedInt(String, int)
 * Integer.parseUnsignedInt(s, 8)}.
 *
 * <p>If the unsigned magnitude is zero, it is represented by a
 * single zero character {@code '0'} ({@code '\u005Cu0030'});
 * otherwise, the first character of the representation of the
 * unsigned magnitude will not be the zero character. The
 * following characters are used as octal digits:
 *
 * <blockquote>
 * {@code 01234567}
 * </blockquote>
 *
 * These are the characters {@code '\u005Cu0030'} through
 * {@code '\u005Cu0037'}.
 *
 * @param   i   an integer to be converted to a string.
 * @return  the string representation of the unsigned integer value
 *          represented by the argument in octal (base&nbsp;8).
 * @see #parseUnsignedInt(String, int)
 * @see #toUnsignedString(int, int)
 * @since   JDK1.0.2
 */
public static String toOctalString(int i) {
    return toUnsignedString0(i, 3);
}

/**
 * Returns a string representation of the integer argument as an
 * unsigned integer in base&nbsp;2.
 *
 * <p>The unsigned integer value is the argument plus 2<sup>32</sup>
 * if the argument is negative; otherwise it is equal to the
 * argument.  This value is converted to a string of ASCII digits
 * in binary (base&nbsp;2) with no extra leading {@code 0}s.
 *
 * <p>The value of the argument can be recovered from the returned
 * string {@code s} by calling {@link
 * Integer#parseUnsignedInt(String, int)
 * Integer.parseUnsignedInt(s, 2)}.
 *
 * <p>If the unsigned magnitude is zero, it is represented by a
 * single zero character {@code '0'} ({@code '\u005Cu0030'});
 * otherwise, the first character of the representation of the
 * unsigned magnitude will not be the zero character. The
 * characters {@code '0'} ({@code '\u005Cu0030'}) and {@code
 * '1'} ({@code '\u005Cu0031'}) are used as binary digits.
 *
 * @param   i   an integer to be converted to a string.
 * @return  the string representation of the unsigned integer value
 *          represented by the argument in binary (base&nbsp;2).
 * @see #parseUnsignedInt(String, int)
 * @see #toUnsignedString(int, int)
 * @since   JDK1.0.2
 */
public static String toBinaryString(int i) {
    return toUnsignedString0(i, 1);
}
```

从实现中可以看到，先计算了转换后字符串的长度，然后开辟 buffer，以倒序将缓存中的字符复制到 buffer 中，同时对数值进行移位和掩码：

```java
/**
 * Convert the integer to an unsigned number.
 */
private static String toUnsignedString0(int val, int shift) {
    // assert shift > 0 && shift <=5 : "Illegal shift value";
    int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
    int chars = Math.max(((mag + (shift - 1)) / shift), 1);
    char[] buf = new char[chars];

    formatUnsignedInt(val, shift, buf, 0, chars);

    // Use special constructor which takes over "buf".
    return new String(buf, true);
}

/**
 * Format a long (treated as unsigned) into a character buffer.
 * @param val the unsigned int to format
 * @param shift the log2 of the base to format in (4 for hex, 3 for octal, 1 for binary)
 * @param buf the character buffer to write to
 * @param offset the offset in the destination buffer to start at
 * @param len the number of characters to write
 * @return the lowest character  location used
 */
static int formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
    int charPos = len;
    int radix = 1 << shift;
    int mask = radix - 1;
    do {
        buf[offset + --charPos] = Integer.digits[val & mask];
        val >>>= shift;
    } while (val != 0 && charPos > 0);

    return charPos;
}
```

---

将字符串按指定进制转换为 `int`。首先根据字符串的首字符来判断符号，之后一律按正数处理。然后就是根据进制数将数不断扩大，以一个最大值作为限制。最终，根据之前判断的符号决定返回正数还是负数。

```java
/**
 * Parses the string argument as a signed integer in the radix
 * specified by the second argument. The characters in the string
 * must all be digits of the specified radix (as determined by
 * whether {@link java.lang.Character#digit(char, int)} returns a
 * nonnegative value), except that the first character may be an
 * ASCII minus sign {@code '-'} ({@code '\u005Cu002D'}) to
 * indicate a negative value or an ASCII plus sign {@code '+'}
 * ({@code '\u005Cu002B'}) to indicate a positive value. The
 * resulting integer value is returned.
 *
 * <p>An exception of type {@code NumberFormatException} is
 * thrown if any of the following situations occurs:
 * <ul>
 * <li>The first argument is {@code null} or is a string of
 * length zero.
 *
 * <li>The radix is either smaller than
 * {@link java.lang.Character#MIN_RADIX} or
 * larger than {@link java.lang.Character#MAX_RADIX}.
 *
 * <li>Any character of the string is not a digit of the specified
 * radix, except that the first character may be a minus sign
 * {@code '-'} ({@code '\u005Cu002D'}) or plus sign
 * {@code '+'} ({@code '\u005Cu002B'}) provided that the
 * string is longer than length 1.
 *
 * <li>The value represented by the string is not a value of type
 * {@code int}.
 * </ul>
 *
 * <p>Examples:
 * <blockquote><pre>
 * parseInt("0", 10) returns 0
 * parseInt("473", 10) returns 473
 * parseInt("+42", 10) returns 42
 * parseInt("-0", 10) returns 0
 * parseInt("-FF", 16) returns -255
 * parseInt("1100110", 2) returns 102
 * parseInt("2147483647", 10) returns 2147483647
 * parseInt("-2147483648", 10) returns -2147483648
 * parseInt("2147483648", 10) throws a NumberFormatException
 * parseInt("99", 8) throws a NumberFormatException
 * parseInt("Kona", 10) throws a NumberFormatException
 * parseInt("Kona", 27) returns 411787
 * </pre></blockquote>
 *
 * @param      s   the {@code String} containing the integer
 *                  representation to be parsed
 * @param      radix   the radix to be used while parsing {@code s}.
 * @return     the integer represented by the string argument in the
 *             specified radix.
 * @exception  NumberFormatException if the {@code String}
 *             does not contain a parsable {@code int}.
 */
public static int parseInt(String s, int radix)
            throws NumberFormatException
{
    /*
        * WARNING: This method may be invoked early during VM initialization
        * before IntegerCache is initialized. Care must be taken to not use
        * the valueOf method.
        */

    if (s == null) {
        throw new NumberFormatException("null");
    }

    if (radix < Character.MIN_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " less than Character.MIN_RADIX");
    }

    if (radix > Character.MAX_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " greater than Character.MAX_RADIX");
    }

    int result = 0;
    boolean negative = false;
    int i = 0, len = s.length();
    int limit = -Integer.MAX_VALUE;
    int multmin;
    int digit;

    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // Possible leading "+" or "-"
            if (firstChar == '-') {
                negative = true;
                limit = Integer.MIN_VALUE;
            } else if (firstChar != '+')
                throw NumberFormatException.forInputString(s);

            if (len == 1) // Cannot have lone "+" or "-"
                throw NumberFormatException.forInputString(s);
            i++;
        }
        multmin = limit / radix;
        while (i < len) {
            // Accumulating negatively avoids surprises near MAX_VALUE
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {
                throw NumberFormatException.forInputString(s);
            }
            result *= radix;
            if (result < limit + digit) {
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;
        }
    } else {
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;
}
```

---

接下来是对象相关的部分。这里的核心是如何将一个 `int` 维护在 `Integer` 对象中，并保证操作性能。如下所示，一个 `int` 类型的数值就这样被维护在对象中。这个值可以通过构造函数直接指定，也可以通过传入一个字符串进行转换。

```java
/**
 * The value of the {@code Integer}.
 *
 * @serial
 */
private final int value;

/**
 * Constructs a newly allocated {@code Integer} object that
 * represents the specified {@code int} value.
 *
 * @param   value   the value to be represented by the
 *                  {@code Integer} object.
 */
public Integer(int value) {
    this.value = value;
}

/**
 * Constructs a newly allocated {@code Integer} object that
 * represents the {@code int} value indicated by the
 * {@code String} parameter. The string is converted to an
 * {@code int} value in exactly the manner used by the
 * {@code parseInt} method for radix 10.
 *
 * @param      s   the {@code String} to be converted to an
 *                 {@code Integer}.
 * @exception  NumberFormatException  if the {@code String} does not
 *               contain a parsable integer.
 * @see        java.lang.Integer#parseInt(java.lang.String, int)
 */
public Integer(String s) throws NumberFormatException {
    this.value = parseInt(s, 10);
}
```

---

性能相关。在 `Integer` 类第一次被加载时，会将一个 (JLS 规定) 范围内的 `int` 相对应的 `Integer` 对象提前实例化，缓存起来。在之后访问时，如果需要的 `Integer` 对象在这个缓存范围内，将不再实例化新的对象，而是直接使用缓存中的。默认的缓存范围是 `-128` 到 `127`，但这个范围可以在 JVM 初始化时通过参数进行调整。

所以这里就成了关于 `equals()` `==` 的面试题的常见考点。因为有些 `Integer` 到最终指向的相同的内存地址，有些指向的是不同的内存地址。

```java
/**
 * Cache to support the object identity semantics of autoboxing for values between
 * -128 and 127 (inclusive) as required by JLS.
 *
 * The cache is initialized on first usage.  The size of the cache
 * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
 * During VM initialization, java.lang.Integer.IntegerCache.high property
 * may be set and saved in the private system properties in the
 * sun.misc.VM class.
 */

private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

获取 `Integer` 对象时，如果已在缓存中，就不实例化新的对象：

```java
/**
 * Returns an {@code Integer} instance representing the specified
 * {@code int} value.  If a new {@code Integer} instance is not
 * required, this method should generally be used in preference to
 * the constructor {@link #Integer(int)}, as this method is likely
 * to yield significantly better space and time performance by
 * caching frequently requested values.
 *
 * This method will always cache values in the range -128 to 127,
 * inclusive, and may cache other values outside of this range.
 *
 * @param  i an {@code int} value.
 * @return an {@code Integer} instance representing {@code i}.
 * @since  1.5
 */
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

---

以下是 `java.lang.Number` 抽象类中需要实现的函数，主要是将 `int` 强制转换为其它数据类型：

```java
/**
 * Returns the value of this {@code Integer} as a {@code byte}
 * after a narrowing primitive conversion.
 * @jls 5.1.3 Narrowing Primitive Conversions
 */
public byte byteValue() {
    return (byte)value;
}

/**
 * Returns the value of this {@code Integer} as a {@code short}
 * after a narrowing primitive conversion.
 * @jls 5.1.3 Narrowing Primitive Conversions
 */
public short shortValue() {
    return (short)value;
}

/**
 * Returns the value of this {@code Integer} as an
 * {@code int}.
 */
public int intValue() {
    return value;
}

/**
 * Returns the value of this {@code Integer} as a {@code long}
 * after a widening primitive conversion.
 * @jls 5.1.2 Widening Primitive Conversions
 * @see Integer#toUnsignedLong(int)
 */
public long longValue() {
    return (long)value;
}

/**
 * Returns the value of this {@code Integer} as a {@code float}
 * after a widening primitive conversion.
 * @jls 5.1.2 Widening Primitive Conversions
 */
public float floatValue() {
    return (float)value;
}

/**
 * Returns the value of this {@code Integer} as a {@code double}
 * after a widening primitive conversion.
 * @jls 5.1.2 Widening Primitive Conversions
 */
public double doubleValue() {
    return (double)value;
}
```

---

关于 hash code，直接返回对象内维护的 `int` 数值：

```java
/**
 * Returns a hash code for this {@code Integer}.
 *
 * @return  a hash code value for this object, equal to the
 *          primitive {@code int} value represented by this
 *          {@code Integer} object.
 */
@Override
public int hashCode() {
    return Integer.hashCode(value);
}

/**
 * Returns a hash code for a {@code int} value; compatible with
 * {@code Integer.hashCode()}.
 *
 * @param value the value to hash
 * @since 1.8
 *
 * @return a hash code value for a {@code int} value.
 */
public static int hashCode(int value) {
    return value;
}
```

---

`equals()` 函数的实现，直接比较对象内维护的 `int` 数值 (前提是比较对象也要是一个 `Integer`)：

```java
/**
 * Compares this object to the specified object.  The result is
 * {@code true} if and only if the argument is not
 * {@code null} and is an {@code Integer} object that
 * contains the same {@code int} value as this object.
 *
 * @param   obj   the object to compare with.
 * @return  {@code true} if the objects are the same;
 *          {@code false} otherwise.
 */
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

`compare()` 函数的实现。通过返回负数、0、整数来决定两个数的大小：

```java
/**
 * Compares two {@code Integer} objects numerically.
 *
 * @param   anotherInteger   the {@code Integer} to be compared.
 * @return  the value {@code 0} if this {@code Integer} is
 *          equal to the argument {@code Integer}; a value less than
 *          {@code 0} if this {@code Integer} is numerically less
 *          than the argument {@code Integer}; and a value greater
 *          than {@code 0} if this {@code Integer} is numerically
 *           greater than the argument {@code Integer} (signed
 *           comparison).
 * @since   1.2
 */
public int compareTo(Integer anotherInteger) {
    return compare(this.value, anotherInteger.value);
}

/**
 * Compares two {@code int} values numerically.
 * The value returned is identical to what would be returned by:
 * <pre>
 *    Integer.valueOf(x).compareTo(Integer.valueOf(y))
 * </pre>
 *
 * @param  x the first {@code int} to compare
 * @param  y the second {@code int} to compare
 * @return the value {@code 0} if {@code x == y};
 *         a value less than {@code 0} if {@code x < y}; and
 *         a value greater than {@code 0} if {@code x > y}
 * @since 1.7
 */
public static int compare(int x, int y) {
    return (x < y) ? -1 : ((x == y) ? 0 : 1);
}

/**
 * Compares two {@code int} values numerically treating the values
 * as unsigned.
 *
 * @param  x the first {@code int} to compare
 * @param  y the second {@code int} to compare
 * @return the value {@code 0} if {@code x == y}; a value less
 *         than {@code 0} if {@code x < y} as unsigned values; and
 *         a value greater than {@code 0} if {@code x > y} as
 *         unsigned values
 * @since 1.8
 */
public static int compareUnsigned(int x, int y) {
    return compare(x + MIN_VALUE, y + MIN_VALUE);
}
```

---

位操作。。。😥 要么暂时。。。

---

