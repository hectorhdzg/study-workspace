# Study Workspace

A workspace to study tech-related topics for programming interviews, covering algorithms, data structures, and system design.

## Contents

- [Algorithms](./algorithms/README.md) — Sorting, searching, dynamic programming, graphs, and more
- [Data Structures](./data-structures/README.md) — Arrays, linked lists, trees, graphs, hash tables, and more
- [System Design](./system-design/README.md) — Scalability, databases, caching, messaging, and real-world designs
- [Generative AI](./gen-ai/README.md) — LLMs, RAG, prompt engineering, vector databases, agents, and safety
- [Interview Tips](./interview-tips/README.md) — Problem-solving strategies, communication, and preparation advice

## How to Use This Workspace

1. Pick a topic from the list above.
2. Read the concept explanation and complexity analysis.
3. Study the provided code examples (JavaScript/TypeScript).
4. Practice the suggested problems.

## Viewing as a Website

This workspace includes a [Docsify](https://docsify.js.org/) setup that renders all the markdown files as a clean, searchable website with sidebar navigation.

### Preview Locally

```bash
npx docsify-cli serve .
```

Then open **http://localhost:3000** in your browser.

### Deploy to GitHub Pages

1. Push this repo to GitHub.
2. Go to **Settings → Pages**.
3. Set source to **Deploy from a branch** → **main** → **/ (root)**.
4. Your site will be live at `https://<username>.github.io/study-workspace/`.

> Use the 🌙 button in the bottom-right corner to toggle dark mode.

## Big-O Complexity Quick Reference

| Complexity | Name        | Example                       |
|-----------|-------------|-------------------------------|
| O(1)      | Constant    | Array access by index         |
| O(log n)  | Logarithmic | Binary search                 |
| O(n)      | Linear      | Linear search                 |
| O(n log n)| Linearithmic| Merge sort, heap sort         |
| O(n²)     | Quadratic   | Bubble sort, insertion sort   |
| O(2ⁿ)     | Exponential | Recursive Fibonacci           |
| O(n!)     | Factorial   | Permutation generation        |
