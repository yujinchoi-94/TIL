# Constructors are not inherited

```java
package com.yujinchoi.springstudy;

public abstract class Car {
    // member
    private CarEngine carEngine;
    // member
    private Wheel wheel;

    // not member
    public Car() {
    }

    // not member
    public Car(CarEngine carEngine, Wheel wheel) {
        this.carEngine = carEngine;
        this.wheel = wheel;
    }

    ...
}

```

```java
// It gets an error
Truck truck = new Truck(carEngine(), wheel());
```

- A subclass inherits all the *members* (fields, methods, and nested classes) from its superclass. Constructors are not members, so they are not inherited by subclasses, but the constructor of the superclass can be invoked from the subclass.

  - You can write a subclass constructor that invokes the constructor of the superclass, either implicitly or by using the keyword `super`.

    ```java
    package com.yujinchoi.springstudy;
    
    public class Truck extends Car {
        // Constructors are not inherited since they're not members.
        public Truck(CarEngine carEngine, Wheel wheel) {
            // Invoke the constructor of the superclass
            super(carEngine, wheel);
        }
    }
    ```

https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html



- For more details
  - https://www.geeksforgeeks.org/constructors-not-inherited-java/
