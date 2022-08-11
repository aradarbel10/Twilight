# Twilight Lang
Twilight is a purely functional programming language based on a construct called *models*, which can encode many useful concepts in PL:

Language Construct | Theory | Model | Operation | Axiom | Entailment | Introduction | Hierarchy | Access
:----|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
Model Theory | signature $T$ | $\mathcal{M}$ | operations | equations | $\mathcal{M}\vDash T$ |  | sub-$T$ |
Algebraic Effects | effect signature | handler |  |  |  | `with _ handle _` |  | open just in scope |
First-Class Modules | signature | module | member functions | dependent member functions |  | `import _` | "sub-module" | `M.x` or `open M` |
Dependent Records | `interface` | `struct` | members |  | `implements` | only local scope | subtyping/inheritance/`<:` | `M.x` |
Type Classes | (type) `class` | `instance` | functions | laws | implied from `instance` | global scope |  | always open |
**Twilight** | `M = [x : T]` | `E = [x = t]` | `x` | `[x : A, y : B(x)]` | `M : E` | `use M; c` | row polymorphism | qualification primitives

For a thorough overview of Twilight and reasons behind its designed see `/docs/motivation.md`.