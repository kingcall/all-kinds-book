## 1. Overview

In this quick article, we're going to see what is the footprint of a *boolean* value in the JVM in different circumstances.

First, we'll inspect the JVM to see the object sizes. Then, we'll understand the rationale behind those sizes.

## 2. Setup

To inspect the memory layout of objects in the JVM, we're going to use the Java Object Layout ([JOL](https://openjdk.java.net/projects/code-tools/jol/)) extensively. Therefore, we need to add the[ *jol-core*](https://search.maven.org/artifact/org.openjdk.jol/jol-core) dependency:

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

## 3. Object Sizes

If we ask JOL to print the VM details in terms of Object Sizes:

```java
System.out.println(VM.current().details());
```

When the [compressed references](https://www.baeldung.com/jvm-compressed-oops) are enabled (the default behavior), we'll see the output:

```bash
# Running 64-bit HotSpot VM.
# Using compressed oop with 3-bit shift.
# Using compressed klass with 3-bit shift.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

In the first few lines, we can see some general information about the VM. After that, we learn about object sizes:

- Java references consume 4 bytes, *boolean*s/*byte*s are 1 byte, *char*s/*short*s are 2 bytes, *int*s/*float*s are 4 bytes, and finally, *long*s/*double*s are 8 bytes
- These types consume the same amount of memory even when we use them as array elements

**So, in the presence of compressed references, each \*boolean\* value takes 1 byte. Similarly, each \*boolean\* in a \*boolean[]\* consumes 1 byte.** However, alignment paddings and object headers can increase the space consumed by *boolean* and *boolean[]* as we'll see later.

### 3.1. No Compressed References

Even if we disable the compressed references via *-XX:-UseCompressedOops*, **the boolean size won't change at all**:

```bash
# Field sizes by type: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

On the other hand, Java references are taking twice the memory.

**So despite what we might expect at first, \*booleans\* are consuming 1 byte instead of just 1 bit.**

### 3.2. Word Tearing

In most architecture, there is no way to access a single bit atomically. Even if we wanted to do so, we probably would end up writing to adjacent bits while updating another one.

**One of the design goals of the JVM is to prevent this phenomenon, known as [word tearing](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.6)**. That is, in the JVM, every field and array element should be distinct; updates to one field or element must not interact with reads or updates of any other field or element.

**To recap, addressability issues and word tearing are the main reasons why \*boolean\*s are more than just one single bit.**

## 4. Ordinary Object Pointers (OOPs)

Now that we know *boolean*s are 1 byte, let's consider this simple class:

```java
class BooleanWrapper {
    private boolean value;
}
```

If we inspect the memory layout of this class using JOL:

```java
System.out.println(ClassLayout.parseClass(BooleanWrapper.class).toPrintable());
```

Then JOL will print the memory layout:

```bash
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     1   boolean BooleanWrapper.value                      N/A
     13     3           (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 3 bytes external = 3 bytes total
```

The *BooleanWrapper* layout consists of:

- 12 bytes for the header, including two [*mark*](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/markOop.hpp) words and one [*klass*](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/klass.hpp) word. The HotSpot JVM uses the *mark* word to store the GC metadata, identity hashcode and locking information. Also, it uses the *klass* word to store class metadata such as runtime type checks
- 1 byte for the actual *boolean* value
- **3 bytes of padding for alignment purposes**

**By default, object references should be aligned by 8 bytes.** Therefore, the JVM adds 3 bytes to 13 bytes of header and *boolean* to make it 16 bytes.

Therefore, *boolean* fields may consume more memory because of their field alignment.

### 4.1. Custom Alignment

If we change the alignment value to 32 via *-XX:ObjectAlignmentInBytes=32,* then the same class layout changes to:

```bash
OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     1   boolean BooleanWrapper.value                      N/A
     13    19           (loss due to the next object alignment)
Instance size: 32 bytes
Space losses: 0 bytes internal + 19 bytes external = 19 bytes total
```

As shown above, the JVM adds 19 bytes of padding to make the object size a multiple of 32.

## 5. Array OOPs

Let's see how the JVM lays out a *boolean* array in memory:

```java
boolean[] value = new boolean[3];
System.out.println(ClassLayout.parseInstance(value).toPrintable());
```

This will print the instance layout as following:

```bash
OFFSET  SIZE      TYPE DESCRIPTION                              
      0     4           (object header)  # mark word
      4     4           (object header)  # mark word
      8     4           (object header)  # klass word
     12     4           (object header)  # array length
     16     3   boolean [Z.<elements>    # [Z means boolean array                        
     19     5           (loss due to the next object alignment)
```

In addition to two *mark* words and one *klass* word, **[array pointers](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/arrayOop.hpp) contain an extra 4 bytes to store their lengths.** 

Since our array has three elements, the size of the array elements is 3 bytes. However, **these 3 bytes will be padded by 5 field alignment bytes to ensure proper alignment.**

Although each *boolean* element in an array is just 1 byte, the whole array consumes much more memory. In other words, **we should consider the header and padding overhead while computing the array size.**

## 6. Conclusion

In this quick tutorial, we saw that *boolean* fields are consuming 1 byte. Also, we learned that we should consider the header and padding overheads in object sizes.

For a more detailed discussion, it's highly recommended to check out the [oops section of the JVM](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/) source code. Also, [Aleksey Shipilëv](https://shipilev.net/) has a much more [in-depth article](https://shipilev.net/jvm/objects-inside-out/) in this area.

As usual, all the examples are available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2).