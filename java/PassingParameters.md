## Data types in Java

There are two types of data types in Java

1. Primitive Type
   - Stores the actual value
2. Reference Type
   - Stores the memory address of a object

## Pass-by-Value

The caller and the callee method operate on two different variables which are copies of each other. Any changes to one variable don’t modify the other.

It means that while calling a method, **parameters passed to the callee method will be clones of original parameters.** Any modification done in callee method will have no effect on the original parameters in caller method.

## Pass-by-Reference

The caller and the callee operate on the same object.

It means that when a variable is pass-by-reference, **the unique identifier of the object is sent to the method.** Any changes to the parameter’s instance members will result in that change being made to the original value.

# Parameter Passing in Java

Arguments in Java are always passed-by-value. During method invocation, a copy of each argument, whether its a value or reference, is created in stack memory which is then passed to the method.

## Passing Primitive Types

Primitive variables are directly stored in stack memory.

Whenever any variable of primitive data type is passed as an argument, the actual parameters are copied to formal arguments and these formal arguments accumulate their own space in stack memory.

```java
public class Primitive {
    @Test
    public void primitive() {
        int a = 1;
        modify(a);
        assertEquals(1, a);
    }

    private void modify(int a) {
        a = 100;
    }
}
```

## Passing Object References

In Java, all objects are dynamically stored in Heap space under the hood. These objects are referred from references called reference variables.

A Java object, in contrast to Primitives, is stored in two stages.

1. The reference variables are stored in stack memory 
2. and the object that they’re referring to, are stored in a Heap memory.

**Whenever an object is passed as an argument, an exact copy of the reference variable is created which points to the same location of the object in heap memory as the original reference variable.**

**As a result of this, whenever we make any change in the same object in the method, that change is reflected in the original object.** However, if we allocate a new object to the passed reference variable, then it won’t be reflected in the original object.

```java
public class NonPrimitive {

    @Test
    public void non_primitive() {
        Bar bar = new Bar(10);
        modify(bar);
        assertEquals(100, bar.i);
    }

    @Test
    public void non_primitive2() {
        Bar bar = new Bar(10);
        modify2(bar);
        assertNotEquals(100, bar.i);
    }

    private void modify(Bar bar) {
        bar.i = 100;
    }

    private void modify2(Bar bar) {
        bar = new Bar(10);
        bar.i = 100;
    }

    private class Bar {
        int i;
        public Bar(int i) {
            this.i = i;
        }
    }
}
```



### References

https://www.baeldung.com/java-pass-by-value-or-pass-by-reference
