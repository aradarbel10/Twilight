# Language Motivation / Overview
Twilight is based on the generalization of three big concepts: effects & handlers, type classes, and first-class modules. From modules we also get the extra bonus of (dependent) records.

The following document incrementally builds up to the entire model system by walking through the different manifestations of it:

## Algebraic Effects
We begin by looking at a simple effect system, which is a slightly modified version of what's actually used in Twilight. The goal of effect systems is to distinguish between pure values, and those that perform computation or invoke side effects.

An *effect signature* is a labeled collection of *operations*, giving each label a type. for example, a `state` effect parameterized over type `T`
```
state (T : Type) = [get : 𝟙 -> T, set : T -> 𝟙]
```
Or an effect that allows raising exceptions as operations which never return anything
```
exception = [raise : 𝟙 -> 𝟘]
```
If an expression uses any of the operations defined by an effect, this information is encoded as part of the type written as `e : t | E` where `e` is an expression, `t` is its type, and `E` is the effect that it uses. (Later we will see this modified to support a more polymorphic handler structure)

Effects also come with *handlers* which give definitions for the effect's operations. The `state` effect may be handled with the handler
```
handler_state s =
    [ return x = (x, s)
    , get      = resume (x, s)
    , set s    = resume (s, ⋆)
    ]
```
> Notation: to avoid confusion this example uses `(,)` notation for tuples, though in the future I'll probably use `[,]` for both tuples and named tuples (ie records (ie modules (ie effect handlers)))

> Notation: we also use `⋆` to denote the unique element of `𝟙`, while `𝟘` stands for the uninhabited type.

This handler uses two special keywords:
- `return` is a special operation which represents returning a pure value from a computation.
- `resume` is a continuation back to the calling code. it is used explicitly, which means a handler may resume multiple times (*multi-shot*), just one time (*one-shot*) as here, or not at all (*zero-shot*).

To get a pure value out of an effectful computation, we must handle all the effects that arise from it (kept track of in the type). We do this with the `with _ handle _` construct, which takes a handler and a computation. This so called computation may look like:
```
post_increment (_ : 𝟙) : int | state int =
    curr <- get ⋆;
    _ <- set (curr + 1);
    curr
```
> Notation: the `;`-block notation may change significantly in the future, this example is just to convey the idea of *sequential computations*.

here we see a function which takes unit and returns an int, but also has the side effect of incrementing a `state int`, which is encoded in the function's type with the `| state int`. (This is in fact a *latent* effect, but we'll not get into that right now)

This function is still impure, so we need to use some handler to eliminate the effect from the type:
```
with (handler_state 5) handle
    n <- post_increment;
    _ <- post_increment;
    return n
```
This expression initializes the internal state to `5`, then increments it twice, but keeps the original value to return at the end. The stateful effects are handled with the `handler_state` defined previously.

Quick reminder that this is effect system is just for demonstration purposes, and not the actual system used in Twilight. Before looking at the real effect system, let's look at a completely separate concept.

## Type Classes
*Type classes* introduce a set of functions into the global scope, parameterized over types. This allows generic functions to have different definitions for different parameter types ("ad-hoc polymorphism"). A type class only provides the signatures for its functions, for example:
```
group (T : Type) = [mult : T -> T -> T, inv : T -> T]
```
> Notation: defining type classes looks pretty similar to defining effect signatures... what a coincidence!

With the help of dependent types and the curry-howard correspondence, we can get a little more mathematical
```
group (T : Type) =
    [ mult : T -> T -> T
    , inv : T -> T

    , assoc : (a b c : T) -> mult (mult a b) c == mult a (mult b c)
    ... more group axioms
]
```
> Notation: sometimes it's cleaner to use line breaks instead of `;`s. I'm still not sure how I feel about it, how it will be to parse, or how it interferes with other parts of the language.

We'll stick with the simpler signature for now and come back to this one later. Now that we have a type class, we can define *instances*, for example how the reals are a group wrt addition:
```
instance group real =
    [ mult p q = p + q
    , inv r = -r
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
instance group real =
    [ mult p q = p * q
    , inv r = 1/r
    ]
```
This instance implements the same `group real` but with different definitions. Mathematically speaking, both of these are equally valid. So which one should we choose?

## Introducing: Scoped Type Classes
Answer: both! Here's a slight change in syntax which allows us to give a name for every instance of a type class
```
additive-real : group real =
    [ mult p q = p + q
    , inv r = -r
    ]

multiplicative-real : group real =
    [ mult p q = p * q
    , inv r = 1/r
    ]
```
But now we have a problem - how do we choose which of the instances to use when we see a `mult 3.0 3.0`? Let's introduce a new syntax construct `use _ in _` which is used like so
```
four_times (x : T) | group T = mult x (mult x (mult x x))

twelve     = use additive-real       ; (four_times 3.0)
eighty_one = use multiplicative-real ; (four_times 3.0)
```
How about using both at the same time? For this we will use `rename` sytnax, like so:
```
quadratic (a b c : real) (x : real) =
    use additive-real rename [plus = mult, neg = inv];
    use multiplicative-real renaming [mult = mult, recip = inv];
    plus (mult a (mult x x)) (plus (mult b x) c)
    -- same as 'a*x*x + b*x + c'
```

## Type Classes + Control Flow = Effects & Handlers
The similarities between type classes and effects are hopefully somewhat apparent at this point. Let's now introduce some new constructs to bridge over the differences. The most obvious difference is in the use of two sepcial keywords in effect definitions: `return` and `resume`. Type classes don't have anything that corresponds to `return`, since they always `resume` to the caller. Moreover they always `resume` exactly once (as do all pure functions) (see *tail resumptive operations* in Koka docs). To overcome these, I decided to introduce new general control flow structures.

- `return`: is a special function that can be defined in any model (*model* would be the general name we use for type classes, handlers, and modules later on). Wherever the model defining this `return` is `use`d, is where the `return` jumps to when it's called. A bit more concretely: if we use some model `M = [return = e, ...]` with `use M; c`, and then somewhere in `c` something calls `return`, the entire `use` expression will evaluate to `e` (the full system has an extra special function `finally` which alters this behavior slightly). This is very similar to C-style return.

- `resume`: is a function that can be used within any function defined in a model. `resume` stands for the current continuation of the location where the function is called, so intuitively it's like an implicit parameter passed to the function from the call site. For example, when we write the sequential code `b <- f a; g`, calling `resume` within the definition of `f` will be like calling `g`.

- Note, in traditional effect handlers the operation typically returns to the point of `use`, not to the caller, and if they want to return a result to the caller they use `resume` (Koka even has special syntax sugar for tail-resumption which I find ugly and want to avoid at all cost). Now this is not the case anymore. Functions return to the caller by default, and `return` is the uniform mechanism for escaping back to the handler.

- `finally`: is a QoL construct that appears in many existing algebraic effect systems. It allows having an internal representation for the effect's data while exposing a transformed version of it. When a model is applied using `use M; c`, and evaluates (either through c or through a return) to some type `T`, the function `finally : T -> S` will be applied on it to produce the final value of the entire `use` expression.

- example: state

## Type-and-Reference Systems
One of the most useful aspects of effect systems is the ability to reason about code's effectful behavior through the language's type system in a safe and explicit manner. For that, we allow annotating every type with an *effect annotation* using `|`, so now our typing judgements will look like `x : τ | ε` where `ε` is a record type containing all operation signatures.

More concretely, if `e : τ | [op : σ, ...]` then the operation `op` of type `σ` can be used in the scope of the expression `e`.

> Does that remind you of something? modules also behave similarly, importing a module allows us to use its signature, but this is never made part of the type system! We'll explore this idea further later on.

What would be the meaning of these annotations under the classes-as-handlers interpretation? This is explicit tracking of which class instances must be defined in scope for the expression `e` to work properly. This isn't anything special, Haskell already includes that kind of annotation in functions in the form of `e : Class c => P c`, we just change a little bit how it looks.

Note we can still annotate with the "entire" typeclass, as in `e : τ | group int` because `group int` still results in a vector. It's still possible to annotate with two or more type-classes, all this requires is a notion of record union, which we will talk about in the modules-as-records interpretation.

A final destinction to make is between *unannotated* types and *trivially annotated* ones, i.e. `e : τ` vs `e : τ | []`. They might seem identical at first, but they are not:
- `e : τ` as far as we are concerned may perform *any* side effect or depend on *any* external definitions. These are left for the type checker to deduce.
- `e : τ | []` specifically performs no side effects, in other words it is a completely pure.

The deeper meaning for this difference is in the subsumption rule induced by effect annotations:
$$
\frac{
    \Gamma\vdash e:\tau\,|\,\varepsilon\quad\quad \varepsilon'\sqsubseteq\varepsilon
}{
    \Gamma\vdash e:\tau\,|\,\varepsilon'
}
$$

## Example: When Type Classes and Handlers Meet
So far we've seen some examples of how models can be viewed as effect handlers, and some other tiems they are more like type classes. Next let's look at an example which is inherently both at the same time:
```
field =
    [ τ : Type

    , return : τ

    , zero : τ
    , unit : τ

    , add_inv : τ -> τ
    , mul_inv : τ -> τ              -- | exception

    , add : τ -> τ -> τ
    , mul : τ -> τ -> τ             -- | exception
    ]                               -- | exception
```
This is a type class defining the algebraic structure called *field*, which many of the common sets of numbers adhere to. But the operations of fields can definitely fail, such as division by zero, so every field instance must be prepared to handle a `return` with a sentient value.

> Realisticly this definition would probably be a little more involved, mainly it will  depend on a separate `exception` handler, as written in the comments to the right of the code. This is explained a bit more deeply in the next section:

## Class Extensions are Effectful Handlers
A relatively simple but important thing to notice so far is the following: if any type can be annotated with a dependency annotation, so can handler signatures. The type `[a : T, ...] | ε` means that the record itself may cause side effects that need to be handled.

This corresponds exactly to subclasses/class extensions. To explain, consider the example
```
functor (F : Type -> Type) = [map : {a b : Type} -> (a -> b) -> (F a -> F b)]

applicative (F : Type -> Type) =
    [ pure : {a : Type} -> a -> F a
    , (<*>) : {a b : Type} -> F (a -> b) -> (F a -> F b)
    ] | functor F
```
This signifies that `applicative` instances must also have a `functor` definition.

## Records
Keeping the two interpretations above in mind (effect handlers & type classes), we shall now talk about some more fundamental properties of records themselves.

A *record* is a (not necessarily ordered) mapping from labels, or *fields*, to values. Written as `[ℓ₁ = e₁, ..., ℓₙ = eₙ]`. We can specify the type of a record simply by supplying a type for each field, `[ℓ₁ : τ₁, ..., ℓₙ : τₙ]`. We can also *project* a value out of a record by writing `r.ℓ` where `r` is a record and `ℓ` is a label defined in it.

In twilight we take this one step further and make use of *dependent records*, in which the *type* of one field can depend on the *values* of all those before it. Hence the fields of a record must have a specified order, namely the order in which they are written. Some common examples for a dependent record types might be
- A vector of known length `[n : Int, v : Vect n a]`
- Another definition for groups `[t : Type, e : t, compose : t -> t -> t]`

Another common operation to be performed on records is union, which constructs a new record with all the labels of the parameters. We denote record unioning with `&`. For example, `[ℓ₁ : τ₁] & [ℓ₂ : τ₂] == [ℓ₁ : τ₁, ℓ₂ : τ₂]`. This union is specifically a *disjoint union*, meaning the same label can't appear in both records. I might think of a way for introducing unrestricted unions in the future (they definitely have their important use cases) once I double down on the semantics.

Records form a pre-order with the subset relation `⊑` we've seen earlier.

## Syntax Sugar
So far we've seen signatures and models written as (respectively)
```
sig =
    [ x1 : t1
    , x2 : t2
    , ...
    ]

mod =
    [ x1 = e1
    , x2 = e2
    , ...
    ]
```
This syntax works nicely on single lines but gets a little messy when we spread out the definition over many lines. For this some sugar is introduced (respectively)
```
sig = signature
    x1 : t1
    x2 : t2
    ...
end

mod = model
    x1 = e1
    x2 = e2
    ...
end
```
These syntactic cosntructs are whitespace-sensitive and enfore consistent indentation.

## Modules are Dependent Records
Modules are a form of program organization, for me one of the main advantages of them is being able to separate conceptually-unrelated code to different files. But certain languages take this concept one step further, and make modules a first class entity with all sorts of cool and unique behaviors. In this section though I'd like to talk just about the code organization part, and introduce some useful constructs to manage that organization which will turn out to be useful for the other interpretations as well.

- Importing: Sometimes we want to write modules in different files. the `import "path/to/file"` operation allows us to take a file from a completely different directory and use the model defined within it. So if we have in one file `a.twt`
```
[...]
```
... and in another file `b.twt`
```
...
a = import "a.twt"
```
This would be equivalent to just writing the model directly
```
a = [...]
```

- Opening / "Using": This is done by the operation `use M; e`, which takes all the names from `M` and puts them in scope for `e`. `use` also serves as the purpose of being the target of a `return` within its scope.

- Qualifying: Instead of entirely opening a model, you can `use M as N; e` which introduces all the interior names qualified under `N`, meaning in `e` they are accessed with `N.ℓ`.

- Renaming: Give new names to labels. Written as `use M rename (ℓ' = ℓ, ...); e` which will rename every `ℓ` to `ℓ'` within the scope of `e`. This is also allowed in combination with `use M as N rename (ℓ' = ℓ, ...); e`.

- Exposing: All members of a model are public by default, hence they are allowed to be accessed from the outside. You can keep a member private by writing `[priv ℓ = x, ...]`, or make it explicitly public with `[pub ℓ = x, ...]` which won't have any special behavior since public is the default.

## Sub-Modules and Row Polymorphism

## Type Classes are Records

## Modules are Handlers
- ex: introducing a memory allocator as a library
- challenges with transferring control flow to a library

## Model Inheritence