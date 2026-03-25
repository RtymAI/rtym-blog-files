---
title: Heading 1
slug: heading-1
date: '2026-03-25T11:41:46.837Z'
updatedAt: '2026-03-25T11:41:46.837Z'
description: test
tags: []
cover: >-
  https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSCeFFQCHg4OkwtcgbTzrHwaMlEdvAYmnn9Wg&s
canonical: ''
seoTitle: Heading 1
seoDescription: test
seoKeywords: []
---

# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6

---

## Text Formatting

**Bold text**  
*Italic text*  
***Bold + Italic***  
~~Strikethrough~~  
`Inline code`

> Blockquote
>> Nested blockquote

---

## Lists

### Unordered List
- Item 1
- Item 2
  - Nested Item 2.1
  - Nested Item 2.2

### Ordered List
1. First
2. Second
   1. Sub item
   2. Sub item

### Task List (GFM)
- [x] Completed task
- [ ] Pending task

---

## Links & Images

[OpenAI](https://openai.com)

![Sample Image](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSCeFFQCHg4OkwtcgbTzrHwaMlEdvAYmnn9Wg&s)

---

## Code Blocks

### JavaScript
```js
function greet(name) {
  return `Hello, ${name}!`;
}
console.log(greet("World"));
```

### JSON
```json
{
  "name": "Test",
  "version": "1.0.0",
  "features": ["markdown", "preview"]
}
```

### Bash
```bash
npm install
npm run dev
```

---

## Tables (GFM)

| Name     | Age | Role        |
|----------|-----|------------|
| Alice    | 25  | Developer  |
| Bob      | 30  | Designer   |

---

## Horizontal Rule

---

## Inline HTML

<div style="color: red;">
  This is raw HTML inside Markdown
</div>

---

## Escaping Characters

\*This is not italic\*  
\# This is not a heading  

---

## Footnotes

Here is a footnote reference[^1].

[^1]: This is the footnote content.

---

## Definition List (may not work in all parsers)

Term 1
: Definition 1

Term 2
: Definition 2

---

## Emoji (depends on parser)

:smile: :rocket: :fire:

---

## Math (if supported)

Inline math: $E = mc^2$

Block math:
$$
\int_a^b f(x)\,dx
$$

---

## Collapsible Section (HTML)

<details>
  <summary>Click to expand</summary>

  Hidden content here!

</details>

---

## Mixed Content

1. **Bold item**
2. *Italic item*
3. `Code item`
4. [Link item](https://example.com)

---

## End of Test File 🎉