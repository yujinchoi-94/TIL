# Stacks

## Definition

Stack : Abstract data type with the following operations

- Push(Key) : adds key to collection
- Key Top() : returns most recently-added key
- Key Pop() : removes and returns most recently-added key
- Boolean Empty() : are there any elements?

Stack is useful when you need to keep track of what has happened in particular order.

### Balanced Brackets

- Input : A string str consisting of '(', ')', '[', ']' characters.
- Output : Return whether or not the string's parentheses and square brackets are balanced.
  - The inner most one is at the top of the stack
  - Tht outer most one is at the bottom of the stack

```
Stack stack // It represents parens and square brackets that are still open
for char in str:
	if char in '(', ')', '[', ']':
		stack.Push(char)
	else:
		if stack.Empty():
			return False
		top = stack.Pop()
		if ((top = '(' and char != ')') or (top = '[' and char != ']')):
			return False
return stack.Empty()
```

If `stack.Empty()` returns `False`, it means unbalanced.

## Array Implementation

`Push()` will return an error if there is no more space in stack.

### Disadvantages

1. We have a maximum size, based on we initially allocated.
2. We have potentially wasted space.

## Linked List Implementation

As long as you have available memory, you can keep adding,

### Disadvantage

We've got the overhead of storing a pointer. On the other hand, there's no wasted space in terms of allocated space that isn't actually being used.

## Summary

- Stacks can be implemented with either an array or a linked list.
- Each stack operation is O(1): Push, Pop, Top, Empty
- Stacks are ocassionalu known as LIFO queues.
