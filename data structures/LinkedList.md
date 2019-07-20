# Linked Lists

## Lists

Store a list of ints as an array

#### Disadvantages

1. Insert item at beginning or middle
   - Time propotional to length of array
2. Arrays have a fixed length

## Linked Lists

- A recursive data type
- Made up of nodes. Each node has an item and a reference to next node in list

```java
package com.yujinchoi.algo.list;

public class ListNode {
    int item;
    ListNode next;
}
```

```java
package com.yujinchoi.algo.list;

public class Main {
    public static void main(String[] args) {
        ListNode listNode1 = new ListNode();
        ListNode listNode2 = new ListNode();
        ListNode listNode3 = new ListNode();

        listNode1.item = 7;
        listNode2.item = 0;
        listNode3.item = 6;

        listNode1.next = listNode2;
        listNode2.next = listNode3;
        listNode3.next = null;
    }
}
```

#### Node Operations

```java
    public ListNode(int item, ListNode next) {
        this.item = item;
        this.next = next;
    }

    public ListNode(int item) {
        this(item, null);
    }
```

```java
ListNode listNode = new ListNode(7, new ListNode(0, new ListNode(6)));
```

#### Advantages over array lists

- Inserting item int middle of linked takes constant time if you have reference to previous node.
- Moreover, lists can keep growing until memory runs out.

##### Insert a new item after "this"

```java
    public void insertAfter(int item) {
        this.next = new ListNode(item, next);
        // The left hand side of next is an old value since it is executed before the assignment.
        // and then, the right hand side of next gets new value.
    }
```

### Disadvantages

- Finding the n-th item of a linked list takes time propotional to n -> length of the list. (Constant time on array list)

```java
    public ListNode nth(int position) {
        if (position == 1) {
            return this;
        } else if (position < 0 || next == null) {
            return null;
        } else {
            return next.nth(position-1);
        }
    }
```

#### List Objects

Reference any object by declaring a reference of type object.

```java
package com.yujinchoi.algo.list;

public class SListNode {
    public Object item;
    public SListNode next;
}
```

#### A List Class

Two problems with SListNodes.

1. Insert a new item of beginning of the list

2. How do you represent an empty list?

   - Run-time error if you call a method on a null object.

   ```java
   x = null; // Empty list. You'll get a run-time error when you call a method on this object.
   ```

##### Solutions

- Separate SList class maintains head of list.

  ```java
  public class SList {
      private ListNode head; //first item in the list
      private int size; // another advantage. Keep track the size
      public SList() {
          head = null;
          size = 0;
      }
      
      public void insertFront(Object item) {
          head = new SListNode(item, head);
          size++;
      }
  }
  ```


### The SList ADT

- Another advantage of SList class:
  - SList ADT enforces two invariants:
    1. "size" is always correct.
    2. List is never circularly linked.
  - Both goal accomplished because only SList methods can change the lists
  - SList ensures this:
    - The fields of SList (head and size) are private.
    - No method of SList returns an SListNode (They can return an item, not a node)

## Doubly Linked Lists

- Inserting/deleting at front of list is easy

```java
public void deleteFront() {
    if (head != null) {
        head = head.next;
        size--;
    }
}
```

- Inserting/deleting at end of list takes a long time

```java
class DListNode {
    Object item;
    DListNode next;
    DListNode prev;
}
class DList {
    private DListNode head;
    private DListNode tail;
}
```

- Insert & delete items at both ends in constant running time.
- Removes the tail node (at least two items in DList)

```java
tail.prev.next = null;
tail = tali.prev;
```

#### sentinel

- A special node that does not represent an item.
- DList v.2 : circulary linked

```java
class DList {
    private DListNode head; // sentinel node
    private int size;
}
```

#### DList invariants with sentinel

1. For any DList d, `d.head != null` (Since there is always sentinel)
2. For any DListNode x, `x.next != null` (It might be a sentinel)
3. For any DListNode x, `x.prev != null`
4. For any DListNode x,  if `x.next == y`, then `y.prev == x`
5. For any DListNode x,  if `x.prev == y`, then `y.next == x`
6. A DList's size variables is the number of DListNodes. NOT COUNTING THE SENTINEL.

**Sentinel is an implementation detial. It should be hidden by ADT**

- Empty DList : sentinel's prev & next field points to itself.
