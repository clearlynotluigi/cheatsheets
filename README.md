# Bert's Dev Toolkit 😎🛠️

> A personal collection of cheatsheets, technical notes, and useful code snippets for the developer's everyday life.

This repository is a living document to the most practical references for the tools we use daily. Whether it's a quick `kubectl` command, a powerful `podman` workflow, or a handy shell script, you'll find it here.

---
### 📚 What's Inside?
---

* **🚀 Cheatsheets:** Quick, practical references for essential CLI tools like `kubectl`, `podman`, `minikube`, and more. Designed for clarity and immediate use.
* **📝 Tech Notes:** In-depth explanations of concepts, workflows, and solutions to common problems we've encountered.
* **💻 Code Snippets:** Reusable scripts and command chains that solve everyday problems and automate repetitive tasks.

---
### 🎨 Styling Conventions
---
*To maintain a beautiful, consistent, and easy-to-read format across all documents, we follow these styling guidelines. This section is also a reference for any AI agent that might contribute in the future.*

#### Use Emojis for Visual Cues
*Emojis in headers and section titles provide a quick visual reference to the content's theme.*
```
# This is a main title 🚀
### 📦 This is a section about packaging
```

#### Use Separators for Logical Breaks
*Horizontal rules (`---`) should be used to create clear separations between major sections of a document.*

#### Use Blockquotes for Tips and Notes
*Blockquotes (`>`) are perfect for highlighting pro-tips, warnings, or important asides without distracting from the main content.*
> ☢️ **Destructive Action:** This is an example of a warning blockquote.

#### Avoid Tables for Code
*To ensure code snippets are easy to copy, **we avoid using markdown tables to display commands and their descriptions**.*
> Tables often trap code in a way that makes copy-pasting difficult and error-prone. Instead, we prefer a linear, card-like layout.

#### Prefer a "Card-Like" Layout
*Each command or concept should be presented as a self-contained block, making it easy to read and copy.*
```markdown
#### `command-name [options]`
*A brief, italicized description of what the command does.*
> An optional blockquote for tips, best practices, or important context.
```shell
# A clear, copy-paste-friendly code block
command-name --with-an-example
```

#### When to Use Tables
*Tables are still great for presenting non-code information where direct copying is not a primary concern.*
> Good use cases include comparing features, listing configuration options, or showing supported values.
| Feature | Supported | Notes |
| :--- | :---: | :--- |
| Feature A | Yes | Has some caveats. |
| Feature B | No | Planned for future. |