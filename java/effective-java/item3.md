# Item3 Enforce the singleton property with a private constructor or an enum type

- A *singleton* is simply a class that is instantiated exactly once.

- Singletons typically represent either a stateless object or a system component that is intrinsically unique.

- Making a class a singleton can make it difficult to test its clients because it's impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type.

- There are two common ways to implement singletons.

  - Public field approach and Static factory approach.
  - They are based on <u>keeping the constructor private</u> and <u>exporting a public static member to provide access to the sole instance</u>.

- **Public field approach**

  - <u>The member is a final field.</u>

  ```java
  // Singleton with public final field
  public class Elvis {
      public static final Elvis INSTANCE = new Elvis();
      private Elvis() { ... }
      public void leaveTheBuilding() { ... }
  }
  ```

  - The private constructor is called only once, to initialize the public static final field `Elvis.INSTANCE`. 
  - The lack of a public or protected constructor guarantees that only one `Elvis` instance will exist
  - Nothing that a client does can change this, with one caveat: a privileged client can invoke the private constructor reflectively with the aid of the `AccessibleObject.setAccessible` method.
    - If you need to defend against this attack, modify the constructor to make it throw an exception if it's asked to create a second instance.
  - Advantages
    - The API makes it clear that the class is a singleton:
      The public statc field is final, so it will always contain the same object reference.
    - It's simpler

- **Static factory appraoch**

  - <u>The public member is a static factory method.</u>

  ```java
  // Singleton with public final field
  public class Elvis {
      private static final Elvis INSTANCE = new Elvis();
      private Elvis() { ... }
      public static Elvis getInstance() { return INSTANCE; }
      public void leaveTheBuilding() { ... }
  }
  ```

  - All calls to `Elvis.getInstance` return the same object reference, and no other `Elvis` instance will ever be created (with the same caveat mentioned earlier).
  - Advantages
    - It gives you the flexibility to change your mind about whether the class is a singleton without changing its API. The factory method returns the sole instance, but it could be modified to return, say, a separate instance for each thread that invokes it.
    - You can write a generic singleton factory if your application requires it.
    - A method reference can be used as a supplier, for example `Elvis::instance` is a `Supplier<Elvis> ` Unless one of these advantages is relevant, the public field approach is preferable.

- To make a singleton class that uses either of these approaches serializable, it is not sufficient merely to add `implements Serializable` to its declaration.

  - To maintain the singleton guarantee, declare all instance fields `transient` and provide a `readResolve` method.

    - Otherwise, each time a serialized instance is deserialized, a new instance will be created.

    - To prevent this from happening, add this `readResolve` method to the Elvise class

      ```java
      // readResolve method to preserve singleton property
      private Object readResolve() {
          // Return the one true Elvis and let the garbage collector
          // take care of the Elvic impersonator.
          return INSTANCE;
      }
      ```

- A third way to implement a singleton is to declare a <u>single-element enum</u>:

  ```java
  // Ebum singleton - the preferred approach
  public enum Elvis {
      INSTANCE;
      public void leaveTheBuilding() { ... }
  }
  ```

  - This approach is similar to the public field approach, but it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee againt multiple instantiation, even in the face of sophisticated serialization or reflection attacks.
    - This approach is often the best way to implement a singleton. Note that you can't you this approach if your singleton must extend a superclass other than Enum(though you can declare an enum to implement interfaces).
