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

### Detect Cycle (Floyd's Tortoise & Hare)

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

### Find Middle Node

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

### Remove Nth Node from End

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
