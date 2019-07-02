# Item4 : Enforce noninstantiability with a private constructor

- You can write a class that is just a grouping of static methods and static fields. Such classes have acquired a bad reputation but they do have valid uses.

- They can be used to group related methods on primitive values or arrays, in the manner of `java.lang.Math`, `java.util.Arrays`. They can also be used to group static methods, including factories, for objects that implement some interface, in the manner of `java.util.Collections`. Lastly, such classes can be used to group methods on a final class, since you can't put them in a subclass 

- Such utility classes were not designed to be instantiated: an instance would be nonsensical.

  - In the absence of explicit consturctors, however, the compiler provides a public, parameterless default constructor. To a user, this constructor is indistinguishable from any other.

- Attempting to enforce noninstantiablility by makind a class abstract does not work.

  - The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance.

- A default constructor is generated only if a class contains no explicit constructors, so a class can be made noninstantiable by including a private constructor.

  ```java
  public class Utils {
    private Utils() {
      throw new AssertionError();
    }
  }
  ```

  - Because the explicit constructor is private, it is inaccessible outside the class.  The `AssertionError` isn't strictly required, but it provides insurance in the case the constructor is accidentally invoked from within the class.

- As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.

  

<details>
  <summary>Utility classes should not have public constructors (from sonar qube coding rules)</summary>

  

  Utility classes, which are collections of static members, are not meant to be instantiated. Even abstract utility classes, which can be extended, should not have public constructors.

Java adds an implicit public constructor to every class which does not define at least one explicitly. Hence, at least one non-public constructor should be defined.

Noncompliant Code Example

```java
class StringUtils { // Noncompliant

  public static String concatenate(String s1, String s2) {
    return s1 + s2;
  }

}
```

Compliant Solution

```java
class StringUtils { // Compliant

  private StringUtils() {
    throw new IllegalStateException("Utility class");
  }

  public static String concatenate(String s1, String s2) {
    return s1 + s2;
  }

}
```

Exceptions

When class contains public static void main(String[] args) method it is not considered as utility class and will be ignored by this rule.
</details>
