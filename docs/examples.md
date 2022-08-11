# Examples
- exceptions
```
error {A : Type} = [throw : 𝟙 -> A]
default_err (a : A) : error int = [throw ⋆ = a]

div (n m : int) : int | error {int} =
    if m == 0 then throw ⋆ else n / m

main : 𝟙 -> 𝟙 | io =
    use (default_err 100);
    print (div 6 3); -- 2
    print (div 5 0); -- 100
    return ⋆;
```

- same but inlined
```
div (n m : int) : int | [throw : 𝟙 -> int] =
    if m == 0 then throw ⋆ else n / m

main (_ : 𝟙) : 𝟙 | io =
    use [throw ⋆ = 100];
    print (div 6 3); -- 2
    print (div 5 0); -- 100
    return ⋆;
```

- errors as maybes
```
error {A : Type} = [throw : 𝟙 -> A]
maybe_err : error (maybe int) =
    [ throw ⋆ = return none
    , return x = return (some x)
    ]
```

- nondeterminism
```
nondet =
    [ fail : 𝟙 -> 𝟙
    , flip : 𝟙 -> bool
    ]

handle_nondet =
    [ return x = x :: nil
    , fail ⋆   = return nil
    , flip ⋆   = return (resume true ++ resume false)
    ]

quantum : int | nondet = if flip ⋆ then 4 else 5
main (_ : 𝟙) : 𝟙 | io =
    use handle_nondet;
    print quantum; -- 4 :: 5 :: nil
    return ⋆;
```

- reader
```
reader (a : Type) = [ask : a]
const_reader {a : Type} (x : a) : reader a = [ask = resume x]

add_twice : int | reader int = ask + ask
const_twice : int <- use (const_reader 21); add_twice
```
challenge: implemet `rand_reader` which propagates an RNG effect. how can a handler rely on another outer effect?

## Product & Sum Types
dependent records
- `[a : T, b : S]` is the type of `[a = t, b = s]`
- untagged tuples?
- list literals?
unions
- `int | string` is the type of `5` and `"hi"`
tagged unions
- `[a : T] | [b : S]` is the type of `[a = t]` and `[b = s]`
- better syntax?

- abstract syntax tree
```
ast = [numlit : int]
    | [boollit : bool]
    | [binop : [o : op, lhs : ast, rhs : ast]]
    | [cond : [c : ast, tb : ast, fb : ast]]
```
or with syntax sugar
```
ast = union
    numlit : int
    boollit : bool
    binop : [o : op, lhs : ast, rhs : ast]
    cond : [c : ast, tb : ast, fb : ast]
end
```

- pattern matching
```
match my_ast on
    numlit n = ...
    boollit b = ...
    binop [o = my_o, lhs = my_lhs, rhs = my_rhs] = ...
    cond [c = my_c, tb = my_tb, fb = my_fb] = ...
end
```