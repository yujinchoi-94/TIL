## Item 1: Consider static factory methods instead of constructors

- public constructor : the traditional way for a class to allow a client to obtain an instance
- static factory method : a static method that returns an instance of the clsass.

Note that a static factory method is not the same as the Factory Method pattern from Design Patterns. The static factory method described in this item has no direct equivalent in Design Patterns.

A class can provide its clients with static factory methos instead of, or in addition to, public constructors. Providing a static factory method instead of a public constructor has both advantages and disadvantages.

### Advantages

1. Unlike constructors, they have names.

   - If the parameters to a constructor do not, in and of themselves, describe the object being returned, a static factory with a well-chosen name is easier to use and the resulting client code easier to read.
   - A class can have only a single constructor with a given signature. Programmers have been known to get around this restriction by providing two construcotrs whose parameters differ.
     - This is a really bad idea. The user of such an API will never be able to remember which constructor is which and will end up calling the wrong on by mistake.
     - People reading code that uses these constructors will not know what the code does without referring to the class documentation.
   - Since static factory methods have names, they don't share the restriction dicussed before. In case where a class seems to require multiple constructors with the same signature, replace the constructors with static factory methods and carefully chosen names to highlight their differences.

2. Unlike constructors, they are not required to create a new object each time they're invoked.

   - This technique is similar to the *Flyweight* pattern. It can greatly improve performance if equivalent objects are requested often, especially if they are expensive to create.
   - The ablility of static factory methods to return the same object from repeated invocatinos allow classes to maintain strict control over what instances exist at anytime. Classes that to do this are said to be instance-controlled. 
     - There are severals reasons to write Instance-controlled classes.
       - Instance control allows a class to guarantee that it is a singleton or noninstantiable.
       - Also, it allows an immutable value class to make the guarantee that no two equal instances exist: `a.equals(b`) if and only if `a==b`

3. Unlike constructors, they can return object of any subtype of their return type.

   - One application of this flexibility is that an API can return objects without making their class public.

4. The returned object can vary from call to call as a function of the input parameters.

   - Any subtype of the declared return type is permissible. The class of the returned object can also vary from release to release.

   - The EnumSet class has no public constructors, only static factories. In the OpenJDK implementation, they return an instance of one of two subclasses.

   - The existence of these two implementatino classes is invisible to clients. Clients neither know nor care about the class of the object they get back from the factory; they care only that it is some subclass of EnumSet.

     ```java
         /**
          * Creates an empty enum set with the specified element type.
          *
          * @param <E> The class of the elements in the set
          * @param elementType the class object of the element type for this enum
          *     set
          * @return An empty enum set of the specified type.
          * @throws NullPointerException if <tt>elementType</tt> is null
          */
         public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
             Enum<?>[] universe = getUniverse(elementType);
             if (universe == null)
                 throw new ClassCastException(elementType + " not an enum");
     
             if (universe.length <= 64)
                 return new RegularEnumSet<>(elementType, universe);
             else
                 return new JumboEnumSet<>(elementType, universe);
         }
     ```

5. The class of the returned object need not exist when the class containing the method is written.

   - Such flexible static factory methods form the basis of *service provider frameworks.* (Ex: JDBC)

### Disadvantages

1. Classes without public or protected constructor cannot be subclassed.
2. They are hard for programmers to find.
