# String Pool

### Java String Pool

- The special memory region where Strings are stored by the JVM

- A pool of strings, initially empty, is maintained privately by the class `String`.

## String Interning

### Interning

- When we create a String variable and assign value to it, the JVM searches the pool for a String of equal value.
- If found, the Java compiler will simply return a reference to its memory address, without allocating additional memory.
- If not found, it'll be added to the pool(interned) and its reference will be returned.

```java
String constantString1 = "test";
String constantString2 = "test";
assertThat(constantString1).isSameAs(constantString2);
```

## Strings Allocated Using the Constructor

When we create a String via the new operator, the Java compiler will create a new object and store it in the heap space reserved for the JVM.

Every String created like this will point to a different memory region with its own address.

```java
String constantString = "test";
String newString = new String("test");
assertThat(constantString).isNotSameAs(newString);
```

## String Literal vs String Object

- When we create a String object using the new() operator, it always creates a new object in heap memory.
- On the other hand, if we create an object using String literal, it may return an existing object from the String pool, if it already exsits. Otherwise, it will create a new String object and put in the string pool for future re-use.

```java
String test1 = new String("test");
String test2 = new String("test");
assertThat(test1).isNotSameAs(test2);
```

## Manual Interning

- We can manually intern a String in the Java String Pool by calling the intern() method on the object we want to intern.
- Manually interning the String will store its reference in the pool, and the JVM will return this reference when needed.

```java
String literalString = "test";
String newString = new String("test");

assertThat(literalString).isNotSameAs(newString);

String internedString = newString.intern();

assertThat(internedString).isSameAs(literalString);
assertThat(internedString).isNotSameAs(newString);
```



### references

https://docs.oracle.com/javase/9/docs/api/java/lang/String.html

https://www.baeldung.com/java-string-pool
