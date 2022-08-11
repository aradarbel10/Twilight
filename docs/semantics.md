# Semantic Formalization
## Type Semantics
First, the system supports a set of built in types denoted with $\mathcal B$ and constants $c_{\mathcal B}$. These constants don't generate any side effects;
$$
\frac{}{
    \Gamma\vdash c_{\mathcal B}:\mathcal B\,!\,\empty
}\;\;\mathrm{(literal)}
$$

The most fundamental rule is for dependent record typing (still not considering special control flow functions)
$$
\frac{
    \forall 1\leq i\leq n \;\big( \Gamma,\{\ell_k:\tau_k\}_{1\leq k<i}\vdash e_i:\tau_i \big)
}{
    \Gamma\vdash[\ell_i=e_i]_{1\leq i\leq n}:[\ell_i:\tau_i]_{1\leq i\leq n}\,!\,\bigcup_{1\leq i\leq n}{\phi_i}
}\;\;\mathrm{(record)}
$$
Labels are taken from a set $\ell \in \mathcal{L}\setminus\{\mathtt{return},\mathtt{finally}\}$


## Core Untyped Operational Semantics
Here we use the notation $e\Downarrow v$ to say $e$ evaluates to $v$. We also accompany every judgement with a binding context $\Phi$ which maps labels to values, built out of $x := v$ bindings.

$$
\frac{
\Phi(x)=e
}{
\Phi\vdash x\Downarrow e
}\;\;\mathrm{(var)}
$$

$$
\frac{
\Phi\vdash e_1\Downarrow \lambda x.d
\quad\quad
\Phi,x:=e_2\vdash d\Downarrow c
}{
\Phi\vdash e_1 \;e_2\Downarrow c
}\;\;\mathrm{(app)}
$$

$$
\frac{
\Phi,\{\ell_j=e_j\}\vdash e\Downarrow e'
}{
\Phi\vdash [\ell = e, \{\ell_j=e_j\}].\ell \Downarrow e'
}\;\;\mathrm{(proj)}
$$

$$
\frac{
    \Phi\vdash m\Downarrow[\mathtt{finally}=f,\{\ell_j=e_j\}]
    \quad\quad
    \Phi,\ell_j=e_j\vdash e\Downarrow e'
    \quad\quad
    \Phi\vdash f\;e'\Downarrow e''
}{
    \Phi\vdash\mathtt{use}\;m\,;e \Downarrow e''
}\;\;\mathrm{(use)}
$$

$$
\frac{
    \Phi,\mathtt{resume}=(\lambda x.c)\vdash e\Downarrow e'
    \quad\quad
    \Phi\vdash c\Downarrow c'
}{
    \Phi\vdash e\,;c\Downarrow c'
}\;\;\mathrm{(seq)}
$$

$$
\frac{
    \Phi,\mathtt{resume}=(\lambda x.c)\vdash e\Downarrow e'
    \quad\quad
    \Phi,x=e'\vdash c\Downarrow c'
}{
    \Phi\vdash x\leftarrow e\,;c\Downarrow c'
}\;\;\mathrm{(bind)}
$$