# Item 5 : Prefer dependency injection to hardwiring resources

- Many classes depend on one or more undelying resources. 

- For a example, a spell checker depends on a dictionary.

  - Implemented as static utility classes

    ```java
    public class SpellChecker {
      private static final Lexicon dictionary = ...;
      private SpellChecker() {} // Noninstantiable
      public static List<String> suggestions(String typo){...}
    }
    ```

  - Implemented as singletons

    ```java
    public class SpellChecker {
      private final Lexicon dictionary = ...;
      private SpellChecker(...) {}
      public static INSTANCE = new SpellChecker(...);
      
      public boolean isValid(String word) {...}
      public List<String> suggestions(String type) {...}
    }
    ```

- Neither of these approaches is satisfactory, because they assume that there is only one dictionary worth using.

- You could try to have `SpellChecker` support multiple dictionaries by making the `dictionary` field non-final and adding a method to change the dictionary in an existing spell checker, but this would be awkward, error-prone, and unworkable in a consurrent setting.

  - Static utility classes and singletons are inappropriate for classes whose behavior is parameterized by an underlying resource.

- What is required is the ablility to support multiple instances of the class, each of whish uses the resource desired by the client. A simple pattern that satisfies this requirement is to <u>pass the resource into the constructor when creating a new instance</u>.

  - This is one form of <u>dependency injection</u>: the dictionary is a dependency of the spell checker and is injected into the spell checker when it created.

- This pattern works with an arbitrary number of resources and arbitrary dependency graphs. It preserves immutability, so multiple clients can share dependent objects. 

- Dependency injection is equally applicable to constructors, static factories, and builders.

- A useful variant of the pattern is to pass a resource factory to the constructor.

- Although dependency injection greatly improves flexibility and testabliity, it can clutter up large projects, which typically contain thousands of dependencies. This clutter can be all but eliminated by using a dependency injection framework.

- Note that APIs designed for manual dependency injection are trivially adapted for use by these frameworks.

- In summary,

  - Do not use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class.
  - Do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor(or static factory or builder).
  - The dependency injection will greatlu enhance the flexibllity, reusability, and testability of a class.
