# Writing Math Formulas in Markdown (.md)

Markdown supports LaTeX-style math syntax when rendered with engines like MathJax or KaTeX.

---

## 🧮 Inline Math

Use single dollar signs `$...$`:

```md
This is an inline formula: $E = mc^2$
```

**Rendered (if supported):**

This is an inline formula: $E = mc^2$

---

## 🧾 Block Math

Use double dollar signs `$$...$$` on separate lines:

```md
$$
\int_a^b f(x)\,dx = F(b) - F(a)
$$
```

**Rendered (if supported):**

$$
\int_a^b f(x)\,dx = F(b) - F(a)
$$

---

## 🧰 Common Examples

```md
- Pythagorean Theorem: $a^2 + b^2 = c^2$
- Fraction: $\frac{1}{2}$
- Sum: $\sum_{i=1}^{n} i$
- Matrix:

$$
\begin{bmatrix}
1 & 2 \\
3 & 4
\end{bmatrix}
$$
```

---

## ⚠️ Notes

- Use **double backslashes (`\\`)** for line breaks inside formulas.
- Use `\quad`, `\;`, `\,` for spacing.
- Not all Markdown platforms support math by default.
  - ✅ Good: Jupyter, Obsidian, HackMD, VSCode (with preview plugin)
  - ⚠️ Limited: GitHub, basic Markdown renderers
