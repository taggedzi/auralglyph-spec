# Contributing to AuralGlyph

Thank you for your interest in helping shape the AuralGlyph specification!

This project uses a robust but lightweight contribution process designed to
protect openness, legal safety, and long-term viability of the standard.

## 1. Contribution License

All contributions (text, diagrams, examples, discussions, code) are licensed
under the same license as the AuralGlyph specification: **CC-BY-4.0**.

## 2. Developer Certificate of Origin (DCO)

All commits must include a Signed-off-by line:

    Signed-off-by: Your Name <email>

This certifies that:

- You wrote the contribution OR have the right to submit it
- You are not violating employer or contractual IP restrictions
- You cannot revoke your contributions later

You may add this automatically:

    git commit -s

## 3. DCO 1.1 (Full Text)

Developer Certificate of Origin 1.1
-----------------------------------

By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I have the
right to submit it under the open source license indicated in the file; or

(b) The contribution is based upon previous work that, to the best of my
knowledge, is covered under an appropriate open source license and I have the
right under that license to submit that work with modifications, whether
created in whole or in part by me, under the same license; or

(c) The contribution was provided directly to me by some other person who
certified (a), (b), or (c), and I have not modified it.

(d) I understand and agree that this project and the contribution are public
and that a record of the contribution (including all personal information
submitted with it) is maintained indefinitely and may be redistributed under
the project's open license.

## 4. Contribution Workflow

1. Fork the repository
2. Create a feature branch
3. Commit changes using `git commit -s`
4. Open a pull request

Thank you again for contributing to AuralGlyph!

---

### AuralGlyph Markdown Math & Notation Standard

**Status:** Normative for all specification documents

#### 1. Rendering Target

All mathematical notation in AuralGlyph specifications MUST render correctly on **GitHub Markdown** using GitHub’s built-in MathJax support.

---

#### 2. Inline Mathematics

Inline mathematical expressions MUST use the following form:

```md
$`expression`$
```

Examples:

- $`x[n]`$
- $`φ_b(f)`$
- $`t ∈ T`$

Other inline math delimiters (`$...$`, `\(...\)`) MUST NOT be used.

---

#### 3. Block Mathematics

Multiline or displayed mathematics MUST be written using **fenced math blocks**:

````md
```math
expression
```
````

Rules:
- Math fences MUST be **left-aligned**
- Math fences MUST NOT be nested inside lists, blockquotes, or tables
- `$$ ... $$`, `\[` `\]`, and similar delimiters MUST NOT be used

---

#### 4. Symbol Style
Where possible, **UTF-8 mathematical symbols** SHOULD be used instead of LaTeX macros.

Preferred examples:
- `φ` instead of `\phi`
- `→` instead of `\rightarrow`
- `∈` instead of `\in`
- `≥`, `≤` instead of `\ge`, `\le`
- `ℝ`, `ℕ` instead of `\mathbb{R}`, `\mathbb{N}`

---

#### 5. LaTeX Macros
LaTeX macros MAY be used **only when necessary** for structure, including:
- Fractions: `\frac{a}{b}`
- Subscripts / superscripts: `x_{b}`, `x^{n}`
- Hats / accents: `\hat{x}`
- Grouping: `{}`, `()`, `[]`

Decorative or stylistic macros (`\mathcal`, `\dots`, `\mathrm`, etc.) MUST NOT be used.

---

#### 6. Readability Over Formalism
The specification prioritizes:
1. Correct GitHub rendering
2. Human readability
3. Copy-and-paste robustness

Perfect typographic fidelity is explicitly **not** a goal.
