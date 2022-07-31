# Twilight Lang
Experiment in advanced type inference implementation techniques and inventive language constructs.

## The 4-Way Equivalence
Language Construct | Theory | Model | Operation | Axiom | Entailment | Introduction | Hierarchy | Access
:----|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
Model Theory | signature $T$ | $\mathcal{M}$ | operations | equations | $\mathcal{M}\vDash T$ |  | sub-$T$ |
Algebraic Effects | effect signature | handler |  |  |  | `with _ handle _` |  | open just in scope |
First-Class Modules | signature | module | member functions | dependent member functions |  | `import _` | "sub-module" | `M.x` or `open M` |
Dependent Records | `interface` | `struct` | members |  | `implements` | only local scope | subtyping/inheritance/`<:` | `M.x` |
Type Classes | (type) `class` | `instance` | functions | laws | implied from `instance` | global scope |  | always open |


## Theories & Models
### As Type Classes
- [Implementing, and Understanding Type Classes](https://okmij.org/ftp/Computation/typeclass.html) online webpage, general survey w/ Haskell and OCaml examples
- [Implementing Type Classes](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.127.8206&rep=rep1&type=pdf) seems approachable

- [First-Class Type Classes](https://sozeau.gitlabpages.inria.fr/www/research/publications/First-Class_Type_Classes.pdf) practical formalization of first-class type classes expressed as linearly-dependent record types.


### As Effects
- [Type and Effect Systems](https://www.ccs.neu.edu/home/amal/course/7480-s12/effects-notes.pdf) gentle introduction

- [Algebraic Effects for Functional Programming](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/08/algeff-tr-2016-v3.pdf)
    - overview of common usecases
    - typing rules & inference
    - compilation via type-directed CPS transformations
- [Efficient Compilation of Algebraic Effect Handlers](https://dl.acm.org/doi/pdf/10.1145/3485479) describes mostly core-to-core optimization transformations

- [What's Algebraic About Algebraic Effects and Handlers?](https://youtu.be/atYp386EGo8) video lecture, theory-focused (formal semantics)
    - [Corresponding lecture notes](https://github.com/OPLSS/introduction-to-algebraic-effects-and-handlers)
    - backing literature
        - [An Introduction to Algebraic Effects and Handlers](https://www.eff-lang.org/handlers-tutorial.pdf)
        - [An Effect System for Algebraic Effects and Handlers](https://arxiv.org/pdf/1306.6316.pdf)
    - [Handlers of Algebraic Effects](https://homepages.inf.ed.ac.uk/gdp/publications/Effect_Handlers.pdf) related paper

#### Builtin Effects
- `pure` (neutral element in the semilattice of effects)
- `io.read`, `io.write`
- `chaos` (equivalent to `top` for types)
#### Builtin Handlers
- `stdio : a | io`
- [IO Inside](https://wiki.haskell.org/IO_inside#Running_with_the_RealWorld) how Haskell's IO monad is implemented.


#### Inference
- [A Generic Type-and-Effect System](http://web.cs.ucla.edu/~todd/research/tldi09.pdf) thorough formalization of effects as prigileges
- arrow types, latent effects
$$
\mathrm{T-Abs}\;\frac{
\Delta;\Gamma,x:\tau_1\vdash e:\tau_2!\phi
}{
\Delta;\Gamma\vdash λ x:\tau_1.e:e_1\xrightarrow{\phi}e_2!\mathtt{pure}
}
\quad\quad\quad\quad
\mathrm{T-App}\;\frac{
\Delta;\Gamma\vdash e_1:\tau_1\xrightarrow{\phi}\tau_2!\phi_1
\quad\quad
\Delta;\Gamma\vdash e_2:\tau_1!\phi_2
}{
\Delta;\Gamma\vdash e_1\;e_2:\tau_2!\phi\cup\phi_1\cup\phi_2
}
$$

### As Modules
- `[ l1 : T1 , ... , ln : Tn ]`
#### Width & Depth Subtyping
#### First-Class Modules
- [Modules Matter Most](http://macqueenfest.cs.uchicago.edu/slides/harper.pdf) slides, relates to typeclasses
#### Dependent Record Types
- [Dependent Record Types Revisited](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.156.3187&rep=rep1&type=pdf#:~:text=A%20dependent%20record%20type%20is,type%20V%20ect(2)) rather simple formalization and application. briefly talks about structural subtyping and relation to sigma-types.
- [Typed Operational Semantics for Dependent Record Types](https://www.cs.rhul.ac.uk/~zhaohui/TYPES09.pdf) more thorough formalization of the system described in the previous paper, including proofs of many important properties.


## Subtyping
### Primitive Hierarchy
- `Nat <: Int <: Real`
### User Defined Coercions
- homomorphisms between algebraic structures

## Algebra of Types
### (Dependent) Records -- (Sigma) Products
- `[a : T, b : S]` is the type of `[a = t, b = s]`
### Tagged Variants
- `{a : T, b : S}` is the type of `{a = t}` and `{b = s}`
### Untagged Variants (Coproducts)
- `{T, S}` is the type of `t : T` and `s : S`
- interaction with subtyping? still decidable?

## Inductive Types
- [Foundation of Inductive Types](https://www.labri.fr/perso/casteran/CoqArt/chapter14.pdf)
### In Terms of Records & Variants
### Inheritence
### Dependent Variations
### Parameters & Indices
### Proof by Induction as Coverage & Termination Checking

## Dependent Types
### Dependent Elimination (Pattern Matching)
- [Elaborating dependent (co)pattern matching](https://www.cse.chalmers.se/~abela/icfp18.pdf) co-authored by Abel. considers dep types.
- [Focusing on Pattern Matching](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.86.2805&rep=rep1&type=pdf) considers relation to curry-howard. very formal. has sections on coverage checking & compilation.
- [Dependently Typed Pattern Matching](http://ats-lang.sourceforge.net/PAPER/DTPM-jucs03.pdf) looks like it has more explanations, appeals to intuition. still also has some formal descriptions.
- [Pattern Matching with Dependent Types](https://wonks.github.io/type-theory-reading-group/papers/proc92-coquand.pdf) uses coverings, not a lot of detail. seems a little outdated. authored by Coquand.

## Procedural Syntax
### With `do/proc` Notation
### Loops/Structured Code?

## Optional Laziness

## Self Hosting
### LLVM
- Native Executable
- WASM?
- [LLVM's GCC Backend](https://llvm.org/pubs/2010-09-HASKELLSYM-LLVM-GHC.pdf)
### Minimal Implementation
- record types w/ subtyping
- non-dependent ADTs
- IO effects

## Syntax Extensions
### Mixfix Operators / Parser Extensions

## Tactics Language

## Memory Management
- manifested as built-in effect handlers
### Read & Write Effects
### Pointers & References

## Markdown & LaTeX Formats
### Structured Math Editor