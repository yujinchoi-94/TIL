# item2 Consider a builder when faced with many construcor parameters

Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters. What sort of constructors or static factories should you write for such a class?

- Telescoping constructor pattern
  - Provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.
  - Disadvantages
    - This pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.
- JavaBeans pattern
  - Call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.
  - It is easy, if a bit wordy, to create instances, and easy to read the resulting code.
  - Disadvantages
    - Because construction is split across multiple calls, a JavaBean may be in an inconsistent state partway througin it construction.
    - The JavaBeans pattern precludes the possibility of making a class immutable and requires added effort on the part of the programmer to ensure thread safety.
    - It is possible to reduce these disadvantages by manually "freezing" the object when its construction is complete and not allowing it to be used until frozen, but this variant is unwieldly and rarely in pratice. Moreover, it can cause errors at runtime because the compiler cannot ensure that the programmer calls the freeze method on an object before using it.
- Builder pattern
  - It combines the safety of the telescoping constructor pattern with the readability of the JavaBeans pattern.
    1. The client calls a constructor (or static factory) with **all of the required parameters and gets a builder object**.
    2. Then the client calls **setter-like methods on the builder object** to set each optional parameter of interest.
    3. Finally, **the client calls a parameterless `build` method to generate the object**, which is typically immutable.
  - The builder is typically a static member class of the class it builds.
  - The builder's setter methods return the builder itself so that invocations can be chained, resulting in a fluent API.
  - The Builder pattern simulates named optional parameters as found in Python and Scala.
  - To detect invalid parameters as soon as possible, check parameter validity in the builder's constructor and methods. Check invariants involving multiple parameters in the constricotr invoked by the `build` method. To ensure these invariants against attack, do the checks on object fields after copying parameters from the builder. If a check fails, throw an IllegalArgumentException whose detail message indicates which parameters are invalid.(?)
  - The Builder pattern is well suited to class hierarchies. Use a parallel hierarchy of builders, each nested in the corresponding class.
  - covariant return typing : A subclass method is declared to return a subtype of the return type declared in the super-class. It allows clients to use these builders without the need for casting
  - Minor advantage
    - It can have multiple varargs parameters because each parameter is specified in its own method.
      - Alternatively, builders can aggregate the parameters passed into multiple calls to.  method into a. ingle field.
  - The Builder pattern is quite flexible. A single builder can be used repeatedly to build multiple objects. The parameters of the builder can be tweaked between invocations of the build method to vary the objects that are created.
  - Disadvantages
    - In order to create an object, you must first create its builder. It could be a problem in performance-critical situations.
    - The pattern is more verbose than the telescoping constructor pattern, so it should be used only if there are enough parameters to make it worthwhile, say four or more.
    - If you start out with constructors or static factories and switch to a builder when the class evolves to the point where the numbers of parameters gets out of hand, the obsolete constructors or static factories will stick out like a sore thumb. Therefore, it's often better to start with a builder in the first place.
  - Summary
    - The Builder patter is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if many of the parameters are optional or of identical type.
