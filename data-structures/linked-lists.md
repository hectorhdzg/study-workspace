# Linked Lists

## Types

- **Singly Linked List** — Each node points to the next.
- **Doubly Linked List** — Each node points to both next and previous.
- **Circular Linked List** — Tail points back to head.

## Complexity

| Operation       | Singly LL | Notes                       |
|----------------|-----------|------------------------------|
| Access by index| O(n)      | Must traverse                |
| Search         | O(n)      |                              |
| Insert at head | O(1)      |                              |
| Insert at tail | O(n) / O(1)| O(1) with tail pointer      |
| Delete at head | O(1)      |                              |
| Delete at tail | O(n)      |                              |

---

## Implementation

A linked list is built from individual **nodes**, each holding a value and a pointer to the next node. Unlike arrays, linked lists don't store elements contiguously in memory — each node can live anywhere on the heap. This makes insertion and deletion at known positions O(1) (no shifting), but random access O(n) since you must traverse from the head.

A common helper pattern is building a list from an array for testing, and converting back for verification.

```javascript
class ListNode {
  constructor(val, next = null) {
    this.val = val;
    this.next = next;
  }
}

// Helper: build list from array
function buildList(arr) {
  if (!arr.length) return null;
  const head = new ListNode(arr[0]);
  let curr = head;
  for (let i = 1; i < arr.length; i++) {
    curr.next = new ListNode(arr[i]);
    curr = curr.next;
  }
  return head;
}

// Helper: list to array
function listToArray(head) {
  const result = [];
  while (head) {
    result.push(head.val);
    head = head.next;
  }
  return result;
}
```

---

## Common Operations

### Reverse a Linked List

Reversing a linked list is one of the most fundamental operations. Walk through the list, and at each node, redirect its `next` pointer to the previous node. You need three pointers: `prev` (starts as null), `curr` (starts at head), and a temporary `next` to avoid losing the rest of the list. After the loop, `prev` is the new head.

**Time:** O(n) | **Space:** O(1)

```javascript
function reverseList(head) {
  let prev = null, curr = head;

  while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }

  return prev;
}
```

```python
def reverse_list(head):
    prev, curr = None, head
    while curr:
        curr.next, prev, curr = prev, curr, curr.next
    return prev
```

```csharp
public ListNode ReverseList(ListNode head)
{
    ListNode prev = null, curr = head;
    while (curr != null)
    {
        var next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

### Detect Cycle (Floyd's Tortoise & Hare)

Floyd's algorithm uses two pointers moving at different speeds: **slow** moves one step, **fast** moves two steps. If there's a cycle, the fast pointer will eventually lap the slow pointer and they'll meet inside the cycle. If fast reaches null, there's no cycle.

This is O(n) time and O(1) space — much better than using a hash set to track visited nodes.

```javascript
function hasCycle(head) {
  let slow = head, fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }

  return false;
}
```

```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False
```

```csharp
public bool HasCycle(ListNode head)
{
    ListNode slow = head, fast = head;
    while (fast?.next != null)
    {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

### Find Middle Node

Use the same slow/fast pointer technique: when the fast pointer reaches the end, the slow pointer is at the middle. This avoids needing to know the list length ahead of time.

```javascript
function middleNode(head) {
  let slow = head, fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }

  return slow;
}
```

### Merge Two Sorted Lists

Given two sorted linked lists, merge them into one sorted list by comparing the heads and always picking the smaller value. A **dummy node** simplifies the logic — it acts as a temporary head so you don't need special-case logic for the first element. When one list is exhausted, append the rest of the other.

**Time:** O(n + m) | **Space:** O(1) (reuse existing nodes)

```javascript
function mergeTwoLists(l1, l2) {
  const dummy = new ListNode(0);
  let curr = dummy;

  while (l1 && l2) {
    if (l1.val <= l2.val) {
      curr.next = l1;
      l1 = l1.next;
    } else {
      curr.next = l2;
      l2 = l2.next;
    }
    curr = curr.next;
  }

  curr.next = l1 || l2;
  return dummy.next;
}
```

```python
def merge_two_lists(l1, l2):
    dummy = curr = ListNode(0)
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next, l1 = l1, l1.next
        else:
            curr.next, l2 = l2, l2.next
        curr = curr.next
    curr.next = l1 or l2
    return dummy.next
```

```csharp
public ListNode MergeTwoLists(ListNode l1, ListNode l2)
{
    var dummy = new ListNode(0);
    var curr = dummy;
    while (l1 != null && l2 != null)
    {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = l1 ?? l2;
    return dummy.next;
}
```

### Remove Nth Node from End

To remove the nth node from the end in a single pass, use a **gap technique**: advance a `fast` pointer n+1 steps ahead, then move both `fast` and `slow` together. When `fast` reaches null, `slow` is right before the target node. A dummy node handles the edge case of removing the head.

```javascript
function removeNthFromEnd(head, n) {
  const dummy = new ListNode(0, head);
  let fast = dummy, slow = dummy;

  // Move fast n+1 steps ahead
  for (let i = 0; i <= n; i++) fast = fast.next;

  while (fast) {
    slow = slow.next;
    fast = fast.next;
  }

  slow.next = slow.next.next;
  return dummy.next;
}
```

---

## Practice Problems

- [LeetCode 206 — Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)
- [LeetCode 141 — Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/)
- [LeetCode 21 — Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)
- [LeetCode 19 — Remove Nth Node From End](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)
- [LeetCode 143 — Reorder List](https://leetcode.com/problems/reorder-list/)
- [LeetCode 23 — Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)
- [LeetCode 148 — Sort List](https://leetcode.com/problems/sort-list/)
