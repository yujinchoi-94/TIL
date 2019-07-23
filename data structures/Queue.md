# Queues

## Definition

Queue: Abstract data type with the following operations:

- Enqueue(Key) : adds key to collection
- Key Dequeue() : removes and returns least recently-added key
- Boolean Empty() : are there any elements?

FIFO : First-In, First-Out

Queues are very useful for things like servers. Where you have got a bunch of operations coming in and you want to service the one that has been waiting the longest.

## Implementation with Linked List

- Enqueue : use `List.PushBack`
- Dequeue: use `List.TopFront` and `List.PopFront`
- Empty : use `Lust.Empty`

#### What would happen if we used List.PushFront to implement Enqueue and List.TopBack and List.PopBack to implement Dequeue?

- The queue would work correctly if the list were singly linked with a tail pointer.
- The queue would work correctly if the list were doubly linked with a tail pointer.
- The queue would work correctly.
  - Both singly- and doubly- linked lists support(correctly ) PushFront, TopBack, and PopBack. It's only the of the operations that are dependent on the list type.
- The queue would work correctly, but Dequeue would be O(n) time if the list were singly-linked with a tail pointer.
  - Dequeue would be O(n) time since PopBack is O(n) for a singly linked list.

## Implementation with Array (Circular array)

- The front of the array is the beginning of the queue.
- read index: Where the next `Dequeue()` operation should happen.
- write index : Where the `Enqueue()` operation should happen.
- Empty() : If read is equal to write, it's empty.
- We have a buffer of at least one element that can't be written to, to make sure read and write are seperate and distinct if the queue is not empty.

## Summary

- Queues can be implemented with either a linked list (with tail pointer) or an array.
- Each queue operation is O(1) : Enqueue, Dequeue, Empty.



- One distinction between the array and the linked list implementation, is that in the array implmentation, we have a size that the queue can grow to. It's bounded.
  - If you don't know a priori how long the queue you need is going to be an array is a bad choice. And any amount that is unused is wasted space.
  - In the linked list implementation, the queues can arbitrarily large as long as there's available memory. The downside is, every element you have to pay for another pointer.
