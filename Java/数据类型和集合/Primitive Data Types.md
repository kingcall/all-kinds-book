### Primitive Data Types in Java

The Java programming language is **statically-typed** meaning all variables should first be declared before they can be used. So you need to state variable's type and name, as example

```
int test = 1;
```

Doing so tells your program that a field named "test" exists, holding numerical data and has an initial value of "1".

A variable's data type determines the values it may contain, plus the operations that may be performed on it. There are two categories of data types in Java-

- Primitive data types, for example- int, float.
- Non-primitive data types, for example- User defined types like classes and interfaces, String, [Array](https://www.netjstech.com/2017/02/array-in-java.html).

This post gives details about the primitive data types in Java.

### Primitive data type

A primitive type is predefined by the language and is named by a reserved keyword. Primitive values do not share state with other primitive values. Java programming language supports **eight primitive data types** which are as follows:

1. byte
2. short
3. int
4. long
5. float
6. double
7. boolean
8. char

These primitive data types in Java can be classified into 4 groups -

1. **Integer types** – byte, short, int, long
2. **Floating-point types** – float, double
3. **Characters** – char
4. **Booleans** - boolean

### Integer data types

\1. **byte**- The byte data type in Java is an **8-bit** signed two's complement integer. It has a **minimum value of -128** and a **maximum value of 127** (inclusive). The byte data type can be useful for saving memory in large arrays, where the memory savings actually matters.

Though you may be tempted to use it at the places where you know value won’t go beyond the byte’s range but the fact is using byte in an expression will result in an [automatic type promotion](https://www.netjstech.com/2015/04/java-automatic-numeric-type-promotion.html) of byte to int when expression is evaluated.

\2. **short**- The short data type is a **16-bit** signed two's complement integer. It has a **minimum value of -32,768** and a **maximum value of 32,767** (inclusive). As with byte, you can use a short to save memory in large arrays, in situations where the memory savings actually matters.

Also the same automatic type promotion from short to int will happen in case of short too.

\3. **int**- The int data type in Java is a **32-bit** signed two's complement integer, which has a **minimum value of -231** and a **maximum value of 231-1**, which means a range of **-2,147,483,648 to 2,147,483,647**.

Note from Java 8 there are some unsigned int methods provided to represent an unsigned 32-bit integer, which has a **minimum value of 0 and a maximum value of 232-1**.

An example of unsigned int method is **Integer.parseUnsignedInt**.

\4. **long**- The long data type in Java is a **64-bit** two's complement integer. The signed long has a **minimum value of -263** and a **maximum value of 263-1** which means a range of **-9,223,372,036,854,775,808 to 9,223,372,036,854,775,807**.

In most of the cases int would be sufficient but long will be useful in scenarios where an int type is not large enough to hold the value.

### Floating-point types

\1. **float**- float data type in Java belongs to the **floating-point type group** along with double. The float data type is a single-precision value requiring **32-bit (4 byte)** of storage. This data type should never be used for precise values where round-off errors may cause problems.

**As example** System.out.println(1-.56); will print 0.43999999999999995 instead of 0.44.

You have to use suffix F (or f) with floating data type numbers, not providing suffix ‘F’ means number would be considered double.

**As example**– If you declare a float variable as below:

```
float f = 1.1; 
```

You will get a compile-time error- *Type mismatch: cannot convert from double to float*

As there is no suffix F so number will be considered double. If you want float data type correct way to write the above statement is- **float f = 1.1F;**

\2. **double**- The double data type in Java is a double-precision value requiring **64-bit (8 byte)** of storage. For decimal values, this data type is generally the default choice. Just like float this data type should never be used for precise values.

- Refer [BigDecimal in Java](https://www.netjstech.com/2017/04/bigdecimal-in-java.html) to see why BigDecimal should be used for monetary calculations where precise values are required.

### Boolean data typen

\1. **boolean**- The boolean data type has only two possible values: **true and false**. The boolean data type is generally used as simple flag to track true/false conditions.

### Character data type

\1. **char**- char data type is used to store characters. In Java, the char data type is a single 16-bit Unicode character. The range of a char is **0 to 65,536**. It has a minimum value of **'\u0000' (or 0)** and a maximum value of **'\uffff' (or 65,535 inclusive)**.

You can either use the character enclosed in single quote to define a char or you can also provide the unicode of the character (hexadecimal form which has \u prefix).

```
char a = 'A';
```

### Default values for Java primitive types

You don't need to always assign a value when a field is declared. Fields that are declared but not initialized will be set to a default value by the compiler. Generally speaking, this default will be zero or null, depending on the data type.

The following table summarizes the default values for the primitive data types in Java.

| **Data Type**          | **Default Value (for fields)** |
| ---------------------- | ------------------------------ |
| byte                   | 0                              |
| short                  | 0                              |
| int                    | 0                              |
| long                   | 0L                             |
| float                  | 0.0f                           |
| double                 | 0.0d                           |
| char                   | '\u0000'                       |
| String (or any object) | null                           |
| boolean                | false                          |

In addition to the above mentioned eight primitive data types, the Java programming language also provides special support for character strings via the java.lang.String class. The [String](https://www.netjstech.com/2016/07/string-in-java.html) class is not technically a primitive data type, but considering the special support given to it by the language, you'll probably tend to think of it as such.

String object (non-primitive data type) can be defined in the same way as primitive data types.

```
String s = "this is a string"
```

That's all for this topic **Primitive Data Types in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!