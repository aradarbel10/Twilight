# Twilight Lang
## The 4-Way Equivalence
Language Construct | Theory | Model | Operation | Axiom | Entailment | Introduction | Hierarchy | Access
:----|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
Model Theory | signature $T$ | $\mathcal{M}$ | operations | equations | $\mathcal{M}\vDash T$ |  | sub-$T$ |
Algebraic Effects | effect signature | handler |  |  |  | `with _ handle _` |  | open just in scope |
First-Class Modules | signature | module | member functions | dependent member functions |  | `import _` | "sub-module" | `M.x` or `open M` |
Dependent Records | `interface` | `struct` | members |  | `implements` | only local scope | subtyping/inheritance/`<:` | `M.x` |
Type Classes | (type) `class` | `instance` | functions | laws | implied from `instance` | global scope |  | always open |


## Language Motivation / Overview
Twilight is based on the generalization of three big concepts: effects & handlers, type classes, and first-class modules. From modules we also get the extra bonus of (dependent) records.

The following document incrementally builds up to the entire model system by walking through the different manifestations of it:

### Algebraic Effects
We begin by looking at a simple effect system, which is a slightly modified version of what's actually used in Twilight. The goal of effect systems is to distinguish between pure values, and those that perform computation or invoke side effects.

An *effect signature* is a labeled collection of *operations*, giving each label a type. for example, a `state` effect parameterized over type `T`
```
state (T : Type) = [get : 𝟙 -> T, set : T -> 𝟙];
```
Or an effect that allows raising exceptions as operations which never return anything
```
exception = [raise : 𝟙 -> 𝟘];
```
If an expression uses any of the operations defined by an effect, this information is encoded as part of the type written as `e : t | E` where `e` is an expression, `t` is its type, and `E` is the effect that it uses. (Later we will see this modified to support a more polymorphic handler structure)

Effects also come with *handlers* which give definitions for the effect's operations. The `state` effect may be handled with the handler
```
handler_state s = [
    return x = (x, s);
    get      = resume (x, s)
    set s    = resume (s, <>)
];
```
> Notation: to avoid confusion this example uses `(,)` notation for tuples, though in the future I might use `[,]` for both tuples and named tuples (ie records (ie modules (ie effect handlers)))

> Notation: we also use `<>` to denote the unique element of `𝟙`, while `𝟘` stands for the uninhabited type.

This handler uses two special keywords:
- `return` is a special operation which represents returning a pure value from a computation.
- `resume` is a continuation back to the calling code. it is used explicitly, which means a handler may resume multiple times (*multi-shot*), just one time (*one-shot*) as here, or not at all (*zero-shot*).

To get a pure value out of an effectful computation, we must handle all the effects that arise from it (kept track of in the type). We do this with the `with _ handle _` construct, which takes a handler and a computation. This so called computation may look like:
```
post_increment (_ : 𝟙) : int | state int =
    curr = get <>;
    set (curr + 1);
    return curr;
```
> Notation: the `;`-block notation may change significantly in the future, this example is just to convey the idea of *sequential computations*.

here we see a function which takes unit and returns an int, but also has the side effect of incrementing a `state int`, which is encoded in the function's type with the `| state int`. (This is in fact a *latent* effect, but we'll not get into that right now)

This function is still impure, so we need to use some handler to eliminate the effect from the type:
```
with (handler_state 5) handle
    n = post_increment;
    _ = post_increment;
    return n;
```
This expression initializes the internal state to `5`, then increments it twice, but keeps the original value to return at the end. The stateful effects are handled with the `handler_state` defined previously.

Quick reminder that this is effect system is just for demonstration purposes, and not the actual system used in Twilight. Before looking at the real effect system, let's look at a completely separate concept.

### Type Classes
*Type classes* introduce a set of functions into the global scope, parameterized over types. This allows generic functions to have different definitions for different parameter types ("ad-hoc polymorphism"). A type class only provides the signatures for its functions, for example:
```
group (T : Type) = [mult : T -> T -> T, inv : T -> T]
```
> Notation: defining type classes looks pretty similar to defining effect signatures... what a coincidence!

With the help of dependent types and the curry-howard correspondence, we can get a little more mathematical
```
group (T : Type) = [
    mult : T -> T -> T,
    inv : T -> T,

    assoc : (a b c : T) -> mult (mult a b) c == mult a (mult b c),
    ... more group axioms
]
```
We'll stick with the simpler signature for now and come back to this one later. Now that we have a type class, we can define *instances*, for example how the reals are a group wrt addition:
```
instance group real = [
    mult p q = p + q,
    inv r = -r
]
```
> Notation: the `instance` keyword is used for saying that the functions defined under it are actually the implementations of those from the signature of `group`.

We can now use this instance in our generic code. First define a generic function taking some parameter `x` of type `T` which multiplies it with itself four times, while making sure `T` is an instance of `group`
```
four_times (x : T) | group T = mult x (mult x (mult x x))
```
> Notation: hey, overlapping syntax again! The same restriction `| E` appeared for effects. A connection between the two might be getting more and more obvious, we will unify the two concepts soon.

And we can use the function we defined on real parameters. `four_times 3.0` results in `12.0`, while `four_times 'H'` gives a type error since we didn't define a `group` instance for characters!

Here's another interesting instance for `group`:
```
instance group real = [
    mult p q = p * q,
    inv r = 1/r
]
```
This instance implements the same `group real` but with different definitions. Mathematically speaking, both of these are equally valid. So which one should we choose?

### Introducing: Scoped Type Classes
Answer: both! Here's a slight change in syntax which allows us to give a name for every instance of a type class
```
additive-real : group real = [
    mult p q = p + q,
    inv r = -r
]

multiplicative-real : group real = [
    mult p q = p * q,
    inv r = 1/r
]
```
But now we have a problem - how do we choose which of the instances to use when we see a `mult 3.0 3.0`? Let's introduce a new syntax construct `use _ in _` which is used like so
```
four_times (x : T) | group T = mult x (mult x (mult x x))

twelve     = use additive-real       in (four_times 3.0)
eighty_one = use multiplicative-real in (four_times 3.0)
```
How about using both at the same time? For this we will use `rename` sytnax, like so:
```
quadratic (a b c : real) = lam (x : real) .
    use additive-real rename (plus = mult, neg = inv) in
    use multiplicative-real renaming (mult = mult, recip = inv) in
    plus (mult a (mult x x)) (plus (mult b x) c)
    -- same as 'a*x*x + b*x + c'
```

### Type Classes are Effects & Handlers
### Records
### Modules are Records
### Sub-Modules and Row Polymorphism
### Type Classes are Records
### Sub-Modules are Functors

### Bonus: Expressing ADTs with Records
dependent records
- `[a : T, b : S]` is the type of `[a = t, b = s]`
- untagged tuples?
- list literals?
variants
- `{a : T, b : S}` is the type of `{a = t}` and `{b = s}`
- better syntax?


# Relevant Literature
## Effects
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

### Builtin Effects
- `pure` (neutral element in the semilattice of effects)
- `io.read`, `io.write`
- `chaos` (equivalent to `top` for types)
### Builtin Handlers
- `stdio : a | io`
- [IO Inside](https://wiki.haskell.org/IO_inside#Running_with_the_RealWorld) how Haskell's IO monad is implemented.

### Inference
- [A Generic Type-and-Effect System](http://web.cs.ucla.edu/~todd/research/tldi09.pdf) thorough formalization of effects as prigileges
- arrow types, latent effects
    ![latent effects typing rules](latenteffects.png)

## Type Classes
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


## Inductive Types
- [Foundation of Inductive Types](https://www.labri.fr/perso/casteran/CoqArt/chapter14.pdf)

## Dependent Types
### Dependent Elimination (Pattern Matching)
- [Elaborating dependent (co)pattern matching](https://www.cse.chalmers.se/~abela/icfp18.pdf) co-authored by Abel. considers dep types.
- [Focusing on Pattern Matching](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.86.2805&rep=rep1&type=pdf) considers relation to curry-howard. very formal. has sections on coverage checking & compilation.
- [Dependently Typed Pattern Matching](http://ats-lang.sourceforge.net/PAPER/DTPM-jucs03.pdf) looks like it has more explanations, appeals to intuition. still also has some formal descriptions.
- [Pattern Matching with Dependent Types](https://wonks.github.io/type-theory-reading-group/papers/proc92-coquand.pdf) uses coverings, not a lot of detail. seems a little outdated. authored by Coquand.

## Self Hosting
### LLVM
- Native Executable
- [LLVM's GCC Backend](https://llvm.org/pubs/2010-09-HASKELLSYM-LLVM-GHC.pdf)
- WASM?
### Minimal Implementation
- record types w/ subtyping
- non-dependent ADTs
- IO effects