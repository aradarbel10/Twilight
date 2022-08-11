# Relevant Literature
## Effects
- [Type and Effect Systems](https://www.ccs.neu.edu/home/amal/course/7480-s12/effects-notes.pdf) gentle introduction, only about effects without handlers

- [Handling Algebraic Effects](https://homepages.inf.ed.ac.uk/gdp/publications/handling-algebraic-effects.pdf) Gordon & Plotkin thesis, VERY DETAILED
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

- [Handlers in Action](https://homepages.inf.ed.ac.uk/slindley/papers/handlers.pdf) algeffects implemented in haskell

- [Exceptional Syntax](https://www.cs.tufts.edu/~nr/cs257/archive/nick-benton/exceptional-syntax.pdf) semantics of exception handlers
- [Effect Handlers in Scope](https://www.cs.ox.ac.uk/people/nicolas.wu/papers/Scope.pdf)

### Builtin Effects
- `[]` (neutral element in the semilattice of effects)
- `io.read`, `io.write`
- `chaos` (equivalent to `top` for types)
### Builtin Handlers
- `stdio : a | io`
- [IO Inside](https://wiki.haskell.org/IO_inside#Running_with_the_RealWorld) how Haskell's IO monad is implemented.

### Inference
- [A Generic Type-and-Effect System](http://web.cs.ucla.edu/~todd/research/tldi09.pdf) thorough formalization of effects as prigileges
- arrow types, latent effects
    ![latent effects typing rules](latenteffects.png)
- [Type Directed Compilation of Row-typed Algebraic Effects](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/08/algeff-tr-2016-1.pdf)
- [Effect Handlers, Evidently](https://dl.acm.org/doi/pdf/10.1145/3408981) efficient compilation using evidence passing
- [First-class Handler Names](https://xnning.github.io/papers/hope21.pdf)

## Type Classes
- [Modular Implicits](https://arxiv.org/pdf/1512.01895.pdf) introducing ad-hoc polymorphism to ML's module system

- [Implementing, and Understanding Type Classes](https://okmij.org/ftp/Computation/typeclass.html) online webpage, general survey w/ Haskell and OCaml examples
- [Implementing Type Classes](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.127.8206&rep=rep1&type=pdf) seems approachable

- [First-Class Type Classes](https://sozeau.gitlabpages.inria.fr/www/research/publications/First-Class_Type_Classes.pdf) practical formalization of first-class type classes expressed as linearly-dependent record types.

## Modules
### Width & Depth Subtyping
### First-Class Modules
- [Modules Matter Most](http://macqueenfest.cs.uchicago.edu/slides/harper.pdf) slides, relates to typeclasses
### Dependent Record Types
- [Dependent Record Types Revisited](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.156.3187&rep=rep1&type=pdf#:~:text=A%20dependent%20record%20type%20is,type%20V%20ect(2)) rather simple formalization and application. briefly talks about structural subtyping and relation to sigma-types.
- [Typed Operational Semantics for Dependent Record Types](https://www.cs.rhul.ac.uk/~zhaohui/TYPES09.pdf) more thorough formalization of the system described in the previous paper, including proofs of many important properties.

- [Logical Relations as Types](https://www.cs.cmu.edu/~rwh/papers/lrat/jacm.pdf) scary thesis
    - [slightly less scary slides](https://www.jonmsterling.com/slides/sterling:2021:au:ccs.pdf)

## Subtyping
- [The Simple Essence of Algebraic Subtyping](https://dl.acm.org/doi/pdf/10.1145/3409006) SimpleSub paper
- [Type Inference with Structural Subtyping: A Faithful Formalization of an Efficient Constraint Solver](http://cristal.inria.fr/~simonet/publis/simonet-aplas03.pdf)
- [Subtyping Dependent Types](https://reader.elsevier.com/reader/sd/pii/S0304397500001754?token=615563B5B022E7BC6FA3AABE6AA5922F690CB8F4CCBBABB731F4E568F8B772820FE38F7EDD19C9D94F6750782CADE890&originRegion=eu-west-1&originCreation=20220731225919)

## Inductive Types
- [Foundation of Inductive Types](https://www.labri.fr/perso/casteran/CoqArt/chapter14.pdf)

## Dependent Types
### Dependent Elimination (Pattern Matching)
- [Elaborating dependent (co)pattern matching](https://www.cse.chalmers.se/~abela/icfp18.pdf) co-authored by Abel. considers dep types.
- [Focusing on Pattern Matching](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.86.2805&rep=rep1&type=pdf) considers relation to curry-howard. very formal. has sections on coverage checking & compilation.
- [Dependently Typed Pattern Matching](http://ats-lang.sourceforge.net/PAPER/DTPM-jucs03.pdf) looks like it has more explanations, appeals to intuition. still also has some formal descriptions.
- [Pattern Matching with Dependent Types](https://wonks.github.io/type-theory-reading-group/papers/proc92-coquand.pdf) uses coverings, not a lot of detail. seems a little outdated. authored by Coquand.

## Self Hosting
- [The Implementation of Functional Programming Languages](https://www.microsoft.com/en-us/research/uploads/prod/1987/01/slpj-book-1987.pdf) Simon L. Peyton Jones
    - semantics of functional languages
    - compilation of pattern matching
    - type checking
    - advanced graph reductions
    - optimizing general tail calls
- [Verified Optimizations for Functional Languages](https://zoep.github.io/thesis_final.pdf) $\lambda_{ANF}$ thesis
- [Tail Recursion Modulo Cons](https://arxiv.org/pdf/2102.09823.pdf)
### LLVM
- Native Executable
- [LLVM's GCC Backend](https://llvm.org/pubs/2010-09-HASKELLSYM-LLVM-GHC.pdf)
- WASM?
### Minimal Implementation
- record types w/ subtyping
- non-dependent ADTs
- IO effects