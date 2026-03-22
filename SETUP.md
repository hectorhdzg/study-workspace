# Setup & Deployment

## Prerequisites

- [Git](https://git-scm.com/)
- [Node.js](https://nodejs.org/) (for local preview only — no build step required)

## Local Preview

```bash
# Clone the repo
git clone https://github.com/hectorhdzg/study-workspace.git
cd study-workspace

# Start the Docsify dev server
npx docsify-cli serve .
```

Open **http://localhost:3000** in your browser. Changes to any `.md` file are reflected instantly on refresh.

## Project Structure

```
study-workspace/
├── index.html          ← Docsify app (theme, plugins, code tabs)
├── _sidebar.md         ← Sidebar navigation
├── .nojekyll           ← Tells GitHub Pages to skip Jekyll
├── README.md           ← Landing page content
├── algorithms/         ← Sorting, searching, DP, graphs, etc.
├── data-structures/    ← Arrays, trees, heaps, hash tables, etc.
├── system-design/      ← Databases, caching, load balancing, etc.
├── gen-ai/             ← LLMs, RAG, prompt engineering, agents, etc.
├── observability/      ← Logging, metrics, tracing, OpenTelemetry, etc.
└── interview-tips/     ← Problem-solving framework, behavioral, study plan
```

## How It Works

This site uses [Docsify](https://docsify.js.org/) — a lightweight documentation site generator that renders Markdown files on the fly. There is **no build step**.

- `index.html` loads Docsify and all plugins (search, syntax highlighting, code tabs)
- `_sidebar.md` defines the sidebar navigation
- All `.md` files are fetched and rendered at runtime in the browser
- Consecutive code blocks in different languages are auto-grouped into tabs

## Adding Content

1. Create or edit any `.md` file in the appropriate folder.
2. If adding a new page, add a link to it in `_sidebar.md`.
3. Commit and push — the site updates automatically.

### Code Samples Convention

To get the auto-tabbed code blocks, place JavaScript, Python, and C# blocks one right after another with no text between them:

````markdown
```javascript
function hello() { console.log("Hello"); }
```

```python
def hello():
    print("Hello")
```

```csharp
public void Hello() => Console.WriteLine("Hello");
```
````

They will automatically render as **JavaScript | Python | C#** tabs. The reader's preferred language is remembered across pages.

## Deploy to GitHub Pages

1. Push this repo to GitHub.
2. Go to **Settings → Pages**.
3. Set **Source** to **Deploy from a branch**.
4. Select **main** branch, **/ (root)** folder.
5. Click **Save**.

The site deploys on every push to `main`. No CI/CD configuration needed.

**Live URL:** https://hectorhdzg.github.io/study-workspace/

## Features

- **Search** — Full-text search across all notes (top-left)
- **Code tabs** — Switch between JavaScript, Python, and C# with one click
- **Dark mode** — Toggle with the 🌙 button (bottom-right), preference saved
- **Sidebar navigation** — All topics organized by category
- **Mobile responsive** — Works on phone/tablet
- **Zero build** — Edit Markdown, push, done
