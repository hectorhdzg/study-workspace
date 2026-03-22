# Tries (Prefix Trees)

## What is a Trie?

A **Trie** (prefix tree) is a tree where each node represents a character. Used for efficient prefix-based string search and autocomplete.

## Complexity

| Operation   | Time   |
|------------|--------|
| Insert     | O(m)   |
| Search     | O(m)   |
| StartsWith | O(m)   |

where `m` = length of the word.

---

## Implementation

Each trie node has a map of children (one per possible character) and a boolean flag marking the end of a word. To **insert**, walk character by character, creating nodes as needed, and mark the final node as a word end. To **search**, walk the same path — if you reach the end and the flag is set, the word exists. **startsWith** is the same walk but doesn't require the end flag.

Tries trade space for speed: they use more memory than a hash set, but enable prefix queries that hash sets can't do efficiently.

```javascript
class TrieNode {
  constructor() {
    this.children = {};
    this.isEnd = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }

  insert(word) {
    let node = this.root;
    for (const c of word) {
      if (!node.children[c]) node.children[c] = new TrieNode();
      node = node.children[c];
    }
    node.isEnd = true;
  }

  search(word) {
    let node = this.root;
    for (const c of word) {
      if (!node.children[c]) return false;
      node = node.children[c];
    }
    return node.isEnd;
  }

  startsWith(prefix) {
    let node = this.root;
    for (const c of prefix) {
      if (!node.children[c]) return false;
      node = node.children[c];
    }
    return true;
  }
}

// Example
const trie = new Trie();
trie.insert("apple");
console.log(trie.search("apple"));   // true
console.log(trie.search("app"));     // false
console.log(trie.startsWith("app")); // true
trie.insert("app");
console.log(trie.search("app"));     // true
```

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]
        node.is_end = True

    def search(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                return False
            node = node.children[c]
        return node.is_end

    def starts_with(self, prefix):
        node = self.root
        for c in prefix:
            if c not in node.children:
                return False
            node = node.children[c]
        return True

trie = Trie()
trie.insert("apple")
print(trie.search("apple"))      # True
print(trie.starts_with("app"))   # True
```

```csharp
public class Trie
{
    private class TrieNode
    {
        public Dictionary<char, TrieNode> Children = new();
        public bool IsEnd;
    }

    private readonly TrieNode _root = new();

    public void Insert(string word)
    {
        var node = _root;
        foreach (char c in word)
        {
            if (!node.Children.ContainsKey(c))
                node.Children[c] = new TrieNode();
            node = node.Children[c];
        }
        node.IsEnd = true;
    }

    public bool Search(string word)
    {
        var node = _root;
        foreach (char c in word)
        {
            if (!node.Children.ContainsKey(c)) return false;
            node = node.Children[c];
        }
        return node.IsEnd;
    }

    public bool StartsWith(string prefix)
    {
        var node = _root;
        foreach (char c in prefix)
        {
            if (!node.Children.ContainsKey(c)) return false;
            node = node.Children[c];
        }
        return true;
    }
}
```

---

## Word Search with Trie

This combines a trie with **backtracking DFS** on a 2D grid. First, insert all target words into a trie. Then for each cell on the board, run DFS following the trie structure — the trie prunes the search by only exploring paths that match known word prefixes. Without the trie, you'd need a separate DFS for every word; with it, you search for all words simultaneously.

**Time:** O(m · n · 4^L) where m×n is the board size and L is max word length | **Space:** O(total characters in word list)

```javascript
// Find all words from a list that appear in a 2D board
function findWords(board, words) {
  const trie = new Trie();
  for (const word of words) trie.insert(word);

  const result = new Set();
  const rows = board.length, cols = board[0].length;

  function dfs(node, row, col, path) {
    if (node.isEnd) result.add(path);

    if (row < 0 || row >= rows || col < 0 || col >= cols) return;
    const c = board[row][col];
    if (c === '#' || !node.children[c]) return;

    board[row][col] = '#'; // mark visited
    const next = node.children[c];
    dfs(next, row + 1, col, path + c);
    dfs(next, row - 1, col, path + c);
    dfs(next, row, col + 1, path + c);
    dfs(next, row, col - 1, path + c);
    board[row][col] = c; // restore
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      dfs(trie.root, r, c, '');
    }
  }

  return [...result];
}
```

---

## Practice Problems

- [LeetCode 208 — Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree/)
- [LeetCode 211 — Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/)
- [LeetCode 212 — Word Search II](https://leetcode.com/problems/word-search-ii/)
- [LeetCode 648 — Replace Words](https://leetcode.com/problems/replace-words/)
