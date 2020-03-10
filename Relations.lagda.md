---
title     : "Relations: Inductive definition of relations"
layout    : page
prev      : /Induction/
permalink : /Relations/
next      : /Equality/
---

```
module plfa.part1.Relations where
```

After having defined operations such as addition and multiplication,
the next step is to define relations, such as _less than or equal_.

## Imports

```
import Relation.Binary.PropositionalEquality as Eq
open Eq using (_≡_; refl; cong; sym)
open import Data.Nat using (ℕ; zero; suc; _+_; _*_)
open import Data.Nat.Properties using (+-comm; *-comm)
--import plfa.part1.Induction using (Bin)
open Eq.≡-Reasoning using (begin_; _≡⟨⟩_; _≡⟨_⟩_; _∎)


```
-- ; to; from; inc)
```

```


## Defining relations

The relation _less than or equal_ has an infinite number of
instances.  Here are a few of them:

    0 ≤ 0     0 ≤ 1     0 ≤ 2     0 ≤ 3     ...
              1 ≤ 1     1 ≤ 2     1 ≤ 3     ...
                        2 ≤ 2     2 ≤ 3     ...
                                  3 ≤ 3     ...
                                            ...

And yet, we can write a finite definition that encompasses
all of these instances in just a few lines.  Here is the
definition as a pair of inference rules:

    z≤n --------
        zero ≤ n

        m ≤ n
    s≤s -------------
        suc m ≤ suc n

And here is the definition in Agda:
```
data _≤_ : ℕ → ℕ → Set where

  z≤n : ∀ {n : ℕ}
      --------
    → zero ≤ n

  s≤s : ∀ {m n : ℕ}
    → m ≤ n
      -------------
    → suc m ≤ suc n
```
Here `z≤n` and `s≤s` (with no spaces) are constructor names, while
`zero ≤ n`, and `m ≤ n` and `suc m ≤ suc n` (with spaces) are types.
This is our first use of an _indexed_ datatype, where the type `m ≤ n`
is indexed by two naturals, `m` and `n`.  In Agda any line beginning
with two or more dashes is a comment, and here we have exploited that
convention to write our Agda code in a form that resembles the
corresponding inference rules, a trick we will use often from now on.

Both definitions above tell us the same two things:

* _Base case_: for all naturals `n`, the proposition `zero ≤ n` holds.
* _Inductive case_: for all naturals `m` and `n`, if the proposition
  `m ≤ n` holds, then the proposition `suc m ≤ suc n` holds.

In fact, they each give us a bit more detail:

* _Base case_: for all naturals `n`, the constructor `z≤n`
  produces evidence that `zero ≤ n` holds.
* _Inductive case_: for all naturals `m` and `n`, the constructor
  `s≤s` takes evidence that `m ≤ n` holds into evidence that
  `suc m ≤ suc n` holds.

For example, here in inference rule notation is the proof that
`2 ≤ 4`:

      z≤n -----
          0 ≤ 2
     s≤s -------
          1 ≤ 3
    s≤s ---------
          2 ≤ 4

And here is the corresponding Agda proof:
```
_ : 2 ≤ 4
_ = s≤s (s≤s z≤n)
```




## Implicit arguments

This is our first use of implicit arguments.  In the definition of
inequality, the two lines defining the constructors use `∀`, very
similar to our use of `∀` in propositions such as:

    +-comm : ∀ (m n : ℕ) → m + n ≡ n + m

However, here the declarations are surrounded by curly braces `{ }`
rather than parentheses `( )`.  This means that the arguments are
_implicit_ and need not be written explicitly; instead, they are
_inferred_ by Agda's typechecker. Thus, we write `+-comm m n` for the
proof that `m + n ≡ n + m`, but `z≤n` for the proof that `zero ≤ n`,
leaving `n` implicit.  Similarly, if `m≤n` is evidence that `m ≤ n`,
we write `s≤s m≤n` for evidence that `suc m ≤ suc n`, leaving both `m`
and `n` implicit.

If we wish, it is possible to provide implicit arguments explicitly by
writing the arguments inside curly braces.  For instance, here is the
Agda proof that `2 ≤ 4` repeated, with the implicit arguments made
explicit:
```
_ : 2 ≤ 4
_ = s≤s {1} {3} (s≤s {0} {2} (z≤n {2}))
```
One may also identify implicit arguments by name:
```
_ : 2 ≤ 4
_ = s≤s {m = 1} {n = 3} (s≤s {m = 0} {n = 2} (z≤n {n = 2}))
```
In the latter format, you may only supply some implicit arguments:
```
_ : 2 ≤ 4
_ = s≤s {n = 3} (s≤s {n = 2} z≤n)
```
It is not permitted to swap implicit arguments, even when named.


## Precedence

We declare the precedence for comparison as follows:
```
infix 4 _≤_
```
We set the precedence of `_≤_` at level 4, so it binds less tightly
than `_+_` at level 6 and hence `1 + 2 ≤ 3` parses as `(1 + 2) ≤ 3`.
We write `infix` to indicate that the operator does not associate to
either the left or right, as it makes no sense to parse `1 ≤ 2 ≤ 3` as
either `(1 ≤ 2) ≤ 3` or `1 ≤ (2 ≤ 3)`.


## Decidability

Given two numbers, it is straightforward to compute whether or not the
first is less than or equal to the second.  We don't give the code for
doing so here, but will return to this point in
Chapter [Decidable]({{ site.baseurl }}/Decidable/).


## Inversion

In our definitions, we go from smaller things to larger things.
For instance, from `m ≤ n` we can conclude `suc m ≤ suc n`,
where `suc m` is bigger than `m` (that is, the former contains
the latter), and `suc n` is bigger than `n`. But sometimes we
want to go from bigger things to smaller things.

There is only one way to prove that `suc m ≤ suc n`, for any `m`
and `n`.  This lets us invert our previous rule.
```
inv-s≤s : ∀ {m n : ℕ}
  → suc m ≤ suc n
    -------------
  → m ≤ n
inv-s≤s (s≤s m≤n) = m≤n
```
Here `m≤n` (with no spaces) is a variable name while
`m ≤ n` (with spaces) is a type, and the latter
is the type of the former.  It is a common convention
in Agda to derive a variable name by removing
spaces from its type.

Not every rule is invertible; indeed, the rule for `z≤n` has
no non-implicit hypotheses, so there is nothing to invert.
But often inversions of this kind hold.

Another example of inversion is showing that there is
only one way a number can be less than or equal to zero.
```
inv-z≤n : ∀ {m : ℕ}
  → m ≤ zero
    --------
  → m ≡ zero
inv-z≤n z≤n = refl
```

## Properties of ordering relations

Relations pop up all the time, and mathematicians have agreed
on names for some of the most common properties.

* _Reflexive_. For all `n`, the relation `n ≤ n` holds.
* _Transitive_. For all `m`, `n`, and `p`, if `m ≤ n` and
`n ≤ p` hold, then `m ≤ p` holds.
* _Anti-symmetric_. For all `m` and `n`, if both `m ≤ n` and
`n ≤ m` hold, then `m ≡ n` holds.
* _Total_. For all `m` and `n`, either `m ≤ n` or `n ≤ m`
holds.

The relation `_≤_` satisfies all four of these properties.

There are also names for some combinations of these properties.

* _Preorder_. Any relation that is reflexive and transitive.
* _Partial order_. Any preorder that is also anti-symmetric.
* _Total order_. Any partial order that is also total.

If you ever bump into a relation at a party, you now know how
to make small talk, by asking it whether it is reflexive, transitive,
anti-symmetric, and total. Or instead you might ask whether it is a
preorder, partial order, or total order.

Less frivolously, if you ever bump into a relation while reading a
technical paper, this gives you a way to orient yourself, by checking
whether or not it is a preorder, partial order, or total order.  A
careful author will often call out these properties---or their
lack---for instance by saying that a newly introduced relation is a
partial order but not a total order.


#### Exercise `orderings` (practice) {#orderings}

Give an example of a preorder that is not a partial order.

```
-- on ℕ, _*_ is reflexive and transitive, but not anti-symmetric
-- Your code goes here
```

Give an example of a partial order that is not a total order.

```
-- on ℕ, _=_ is reflexive, transitive, and anti-symmetric, but not total (1 /= 2 and 2/= 1)
-- Your code goes here
```

## Reflexivity

The first property to prove about comparison is that it is reflexive:
for any natural `n`, the relation `n ≤ n` holds.  We follow the
convention in the standard library and make the argument implicit,
as that will make it easier to invoke reflexivity:
```
r' : ∀ {n : ℕ} → n ≤ n
r' {zero} = z≤n
r' {suc n} = s≤s (r' {n}) 

≤-refl : ∀ {n : ℕ}
    -----
  → n ≤ n
≤-refl {zero} = z≤n
≤-refl {suc n} = s≤s ≤-refl
```
The proof is a straightforward induction on the implicit argument `n`.
In the base case, `zero ≤ zero` holds by `z≤n`.  In the inductive
case, the inductive hypothesis `≤-refl {n}` gives us a proof of `n ≤
n`, and applying `s≤s` to that yields a proof of `suc n ≤ suc n`.

It is a good exercise to prove reflexivity interactively in Emacs,
using holes and the `C-c C-c`, `C-c C-,`, and `C-c C-r` commands.

## Transitivity

The second property to prove about comparison is that it is
transitive: for any naturals `m`, `n`, and `p`, if `m ≤ n` and `n ≤ p`
hold, then `m ≤ p` holds.  Again, `m`, `n`, and `p` are implicit:
```
t' : ∀ {m n p : ℕ} → m ≤ n → n ≤ p → m ≤ p
t' z≤n _ = z≤n
t' (s≤s m≤n) (s≤s n≤p) = s≤s (t' m≤n n≤p)


≤-trans' : ∀ {m n p : ℕ} → m ≤ n → n ≤ p → m ≤ p
≤-trans' z≤n  _ = z≤n
≤-trans' (s≤s m≤n) (s≤s n≤p) = s≤s (≤-trans' m≤n n≤p)

```






```
≤-trans : ∀ {m n p : ℕ}
  → m ≤ n
  → n ≤ p
    -----
  → m ≤ p
≤-trans z≤n       _          =  z≤n
≤-trans (s≤s m≤n) (s≤s n≤p)  =  s≤s (≤-trans m≤n n≤p)
```
Here the proof is by induction on the _evidence_ that `m ≤ n`.  In the
base case, the first inequality holds by `z≤n` and must show `zero ≤
p`, which follows immediately by `z≤n`.  In this case, the fact that
`n ≤ p` is irrelevant, and we write `_` as the pattern to indicate
that the corresponding evidence is unused.

In the inductive case, the first inequality holds by `s≤s m≤n`
and the second inequality by `s≤s n≤p`, and so we are given
`suc m ≤ suc n` and `suc n ≤ suc p`, and must show `suc m ≤ suc p`.
The inductive hypothesis `≤-trans m≤n n≤p` establishes
that `m ≤ p`, and our goal follows by applying `s≤s`.

The case `≤-trans (s≤s m≤n) z≤n` cannot arise, since the first
inequality implies the middle value is `suc n` while the second
inequality implies that it is `zero`.  Agda can determine that such a
case cannot arise, and does not require (or permit) it to be listed.

Alternatively, we could make the implicit parameters explicit:
```
≤-trans′ : ∀ (m n p : ℕ)
  → m ≤ n
  → n ≤ p
    -----
  → m ≤ p
≤-trans′ zero    _       _       z≤n       _          =  z≤n
≤-trans′ (suc m) (suc n) (suc p) (s≤s m≤n) (s≤s n≤p)  =  s≤s (≤-trans′ m n p m≤n n≤p)
```
One might argue that this is clearer or one might argue that the extra
length obscures the essence of the proof.  We will usually opt for
shorter proofs.

The technique of induction on evidence that a property holds (e.g.,
inducting on evidence that `m ≤ n`)---rather than induction on
values of which the property holds (e.g., inducting on `m`)---will turn
out to be immensely valuable, and one that we use often.

Again, it is a good exercise to prove transitivity interactively in Emacs,
using holes and the `C-c C-c`, `C-c C-,`, and `C-c C-r` commands.


## Anti-symmetry

The third property to prove about comparison is that it is
antisymmetric: for all naturals `m` and `n`, if both `m ≤ n` and `n ≤
m` hold, then `m ≡ n` holds:
```
a' : ∀ {m n : ℕ} → m ≤ n → n ≤ m → m ≡ n
a' z≤n z≤n = refl
a' (s≤s m≤n) (s≤s n≤m) = cong suc (a' m≤n n≤m)

≤-antisym : ∀ {m n : ℕ}
  → m ≤ n
  → n ≤ m
    -----
  → m ≡ n
≤-antisym z≤n       z≤n        =  refl
≤-antisym (s≤s m≤n) (s≤s n≤m)  =  cong suc (≤-antisym m≤n n≤m)
```
Again, the proof is by induction over the evidence that `m ≤ n`
and `n ≤ m` hold.

In the base case, both inequalities hold by `z≤n`, and so we are given
`zero ≤ zero` and `zero ≤ zero` and must show `zero ≡ zero`, which
follows by reflexivity.  (Reflexivity of equality, that is, not
reflexivity of inequality.)

In the inductive case, the first inequality holds by `s≤s m≤n` and the
second inequality holds by `s≤s n≤m`, and so we are given `suc m ≤ suc n`
and `suc n ≤ suc m` and must show `suc m ≡ suc n`.  The inductive
hypothesis `≤-antisym m≤n n≤m` establishes that `m ≡ n`, and our goal
follows by congruence.

#### Exercise `≤-antisym-cases` (practice) {#leq-antisym-cases}

The above proof omits cases where one argument is `z≤n` and one
argument is `s≤s`.  Why is it ok to omit them?

```
-- I think because agda knows suc n never ≡ zero 
-- Your code goes here
```


## Total

The fourth property to prove about comparison is that it is total:
for any naturals `m` and `n` either `m ≤ n` or `n ≤ m`, or both if
`m` and `n` are equal.

We specify what it means for inequality to be total:
```
data Total (m n : ℕ) : Set where

  forward :
      m ≤ n
      ---------
    → Total m n

  flipped :
      n ≤ m
      ---------
    → Total m n
```
Evidence that `Total m n` holds is either of the form
`forward m≤n` or `flipped n≤m`, where `m≤n` and `n≤m` are
evidence of `m ≤ n` and `n ≤ m` respectively.

(For those familiar with logic, the above definition
could also be written as a disjunction. Disjunctions will
be introduced in Chapter [Connectives]({{ site.baseurl }}/Connectives/).)

This is our first use of a datatype with _parameters_,
in this case `m` and `n`.  It is equivalent to the following
indexed datatype:
```
data T' : ℕ → ℕ → Set where
  fore : ∀ {m n : ℕ} → m ≤ n → T' m n
  back : ∀ {m n : ℕ} → n ≤ m → T' m n

data T'' (m n : ℕ) : Set where
  fore' : m ≤ n → T'' m n
  back' : n ≤ m → T'' m n


data Total′ : ℕ → ℕ → Set where

  forward′ : ∀ {m n : ℕ}
    → m ≤ n
      ----------
    → Total′ m n

  flipped′ : ∀ {m n : ℕ}
    → n ≤ m
      ----------
    → Total′ m n
```
Each parameter of the type translates as an implicit parameter of each
constructor.  Unlike an indexed datatype, where the indexes can vary
(as in `zero ≤ n` and `suc m ≤ suc n`), in a parameterised datatype
the parameters must always be the same (as in `Total m n`).
Parameterised declarations are shorter, easier to read, and
occasionally aid Agda's termination checker, so we will use them in
preference to indexed types when possible.

With that preliminary out of the way, we specify and prove totality:
```
Tp : ∀ (m n : ℕ) → Total m n 
Tp zero n = forward z≤n
Tp m zero = flipped z≤n
Tp (suc m) (suc n) with Tp m n
...                   | forward m≤n = forward (s≤s m≤n)
...                   | flipped n≤m = flipped (s≤s n≤m)


Tph : ∀ (m n : ℕ) → Total m n 
Tph zero n = forward z≤n
Tph m zero = flipped z≤n
Tph (suc m) (suc n) = helper (Tph m n)
  where
  helper : Total m n → Total (suc m) (suc n)
  helper (forward m≤n) = forward (s≤s m≤n) 
  helper (flipped m≤n) = flipped (s≤s m≤n)

≤-total : ∀ (m n : ℕ) → Total m n
≤-total zero    n                         =  forward z≤n
≤-total (suc m) zero                      =  flipped z≤n
≤-total (suc m) (suc n) with ≤-total m n
...                        | forward m≤n  =  forward (s≤s m≤n)
...                        | flipped n≤m  =  flipped (s≤s n≤m)
```
In this case the proof is by induction over both the first
and second arguments.  We perform a case analysis:

* _First base case_: If the first argument is `zero` and the
  second argument is `n` then the forward case holds,
  with `z≤n` as evidence that `zero ≤ n`.

* _Second base case_: If the first argument is `suc m` and the
  second argument is `zero` then the flipped case holds, with
  `z≤n` as evidence that `zero ≤ suc m`.

* _Inductive case_: If the first argument is `suc m` and the
  second argument is `suc n`, then the inductive hypothesis
  `≤-total m n` establishes one of the following:

  + The forward case of the inductive hypothesis holds with `m≤n` as
    evidence that `m ≤ n`, from which it follows that the forward case of the
    proposition holds with `s≤s m≤n` as evidence that `suc m ≤ suc n`.

  + The flipped case of the inductive hypothesis holds with `n≤m` as
    evidence that `n ≤ m`, from which it follows that the flipped case of the
    proposition holds with `s≤s n≤m` as evidence that `suc n ≤ suc m`.

This is our first use of the `with` clause in Agda.  The keyword
`with` is followed by an expression and one or more subsequent lines.
Each line begins with an ellipsis (`...`) and a vertical bar (`|`),
followed by a pattern to be matched against the expression
and the right-hand side of the equation.

Every use of `with` is equivalent to defining a helper function.  For
example, the definition above is equivalent to the following:
```
≤-total′ : ∀ (m n : ℕ) → Total m n
≤-total′ zero    n        =  forward z≤n
≤-total′ (suc m) zero     =  flipped z≤n
≤-total′ (suc m) (suc n)  =  helper (≤-total′ m n)
  where
  helper : Total m n → Total (suc m) (suc n)
  helper (forward m≤n)  =  forward (s≤s m≤n)
  helper (flipped n≤m)  =  flipped (s≤s n≤m)
```
This is also our first use of a `where` clause in Agda.  The keyword `where` is
followed by one or more definitions, which must be indented.  Any variables
bound on the left-hand side of the preceding equation (in this case, `m` and
`n`) are in scope within the nested definition, and any identifiers bound in the
nested definition (in this case, `helper`) are in scope in the right-hand side
of the preceding equation.

If both arguments are equal, then both cases hold and we could return evidence
of either.  In the code above we return the forward case, but there is a
variant that returns the flipped case:
```
≤-total″ : ∀ (m n : ℕ) → Total m n
≤-total″ m       zero                      =  flipped z≤n
≤-total″ zero    (suc n)                   =  forward z≤n
≤-total″ (suc m) (suc n) with ≤-total″ m n
...                        | forward m≤n   =  forward (s≤s m≤n)
...                        | flipped n≤m   =  flipped (s≤s n≤m)
```
It differs from the original code in that it pattern
matches on the second argument before the first argument.


## Monotonicity

If one bumps into both an operator and an ordering at a party, one may ask if
the operator is _monotonic_ with regard to the ordering.  For example, addition
is monotonic with regard to inequality, meaning:

    ∀ {m n p q : ℕ} → m ≤ n → p ≤ q → m + p ≤ n + q

The proof is straightforward using the techniques we have learned, and is best
broken into three parts. First, we deal with the special case of showing
addition is monotonic on the right:
```
+-monoʳ-≤' : ∀ (n p q : ℕ) → p ≤ q → n + p ≤ n + q
+-monoʳ-≤' zero p q p≤q = p≤q    
+-monoʳ-≤' (suc n) p q p≤q = s≤s (+-monoʳ-≤' n p q p≤q)


+-monoʳ-≤ : ∀ (n p q : ℕ)
  → p ≤ q
    -------------
  → n + p ≤ n + q
+-monoʳ-≤ zero    p q p≤q  =  p≤q
+-monoʳ-≤ (suc n) p q p≤q  =  s≤s (+-monoʳ-≤ n p q p≤q)
```
The proof is by induction on the first argument.

* _Base case_: The first argument is `zero` in which case
  `zero + p ≤ zero + q` simplifies to `p ≤ q`, the evidence
  for which is given by the argument `p≤q`.

* _Inductive case_: The first argument is `suc n`, in which case
  `suc n + p ≤ suc n + q` simplifies to `suc (n + p) ≤ suc (n + q)`.
  The inductive hypothesis `+-monoʳ-≤ n p q p≤q` establishes that
  `n + p ≤ n + q`, and our goal follows by applying `s≤s`.

Second, we deal with the special case of showing addition is
monotonic on the left. This follows from the previous
result and the commutativity of addition:
```
+-monoˡ-≤' : ∀ (m n p : ℕ) → m ≤ n → m + p ≤ n + p
+-monoˡ-≤' m n p m≤n rewrite +-comm m p | +-comm n p = +-monoʳ-≤ p m n m≤n 

+-monoˡ-≤ : ∀ (m n p : ℕ)
  → m ≤ n
    -------------
  → m + p ≤ n + p
+-monoˡ-≤ m n p m≤n  rewrite +-comm m p | +-comm n p  = +-monoʳ-≤ p m n m≤n
```
Rewriting by `+-comm m p` and `+-comm n p` converts `m + p ≤ n + p` into
`p + m ≤ p + n`, which is proved by invoking `+-monoʳ-≤ p m n m≤n`.

Third, we combine the two previous results:
```
+-mono-≤ : ∀ (m n p q : ℕ)
  → m ≤ n
  → p ≤ q
    -------------
  → m + p ≤ n + q
+-mono-≤ m n p q m≤n p≤q  =  ≤-trans (+-monoˡ-≤ m n p m≤n) (+-monoʳ-≤ n p q p≤q)
```
Invoking `+-monoˡ-≤ m n p m≤n` proves `m + p ≤ n + p` and invoking
`+-monoʳ-≤ n p q p≤q` proves `n + p ≤ n + q`, and combining these with
transitivity proves `m + p ≤ n + q`, as was to be shown.


#### Exercise `*-mono-≤` (stretch)

Show that multiplication is monotonic with regard to inequality.

```
*-monoʳ-≤ : ∀ (n p q : ℕ)
  → p ≤ q
    -------------
  → n * p ≤ n * q
*-monoʳ-≤ zero    p q p≤q  =  z≤n
*-monoʳ-≤ (suc n) p q p≤q  = +-mono-≤ p q (n * p) (n * q) p≤q (*-monoʳ-≤ n p q p≤q)

*-monol-≤ : ∀ (m n p : ℕ) → m ≤ n → m * p ≤ n * p
*-monol-≤ m n p m≤n rewrite *-comm m p | *-comm n p = *-monoʳ-≤ p m n m≤n

*-mono-≤ : ∀ (m n p q : ℕ) → m ≤ n → p ≤ q → m * p ≤ n * q
*-mono-≤ m n p q m≤n p≤q = ≤-trans (*-monol-≤ m n p m≤n) (*-monoʳ-≤ n p q p≤q)
-- Your code goes here
```


## Strict inequality {#strict-inequality}

We can define strict inequality similarly to inequality:
```
infix 4 _<'_
data _<'_ : ℕ → ℕ → Set where
  z<'s : ∀ {n : ℕ} → zero <' (suc n)
  s<'s : ∀ {m n : ℕ} → m <' n → suc m <' suc n

infix 4 _<_

data _<_ : ℕ → ℕ → Set where

  z<s : ∀ {n : ℕ}
      ------------
    → zero < suc n

  s<s : ∀ {m n : ℕ}
    → m < n
      -------------
    → suc m < suc n

```
The key difference is that zero is less than the successor of an
arbitrary number, but is not less than zero.

Clearly, strict inequality is not reflexive. However it is
_irreflexive_ in that `n < n` never holds for any value of `n`.
Like inequality, strict inequality is transitive.
Strict inequality is not total, but satisfies the closely related property of
_trichotomy_: for any `m` and `n`, exactly one of `m < n`, `m ≡ n`, or `m > n`
holds (where we define `m > n` to hold exactly when `n < m`).
It is also monotonic with regards to addition and multiplication.

Most of the above are considered in exercises below.  Irreflexivity
requires negation, as does the fact that the three cases in
trichotomy are mutually exclusive, so those points are deferred to
Chapter [Negation]({{ site.baseurl }}/Negation/).

It is straightforward to show that `suc m ≤ n` implies `m < n`,
and conversely.  One can then give an alternative derivation of the
properties of strict inequality, such as transitivity, by
exploiting the corresponding properties of inequality.

#### Exercise `<-trans` (recommended) {#less-trans}

Show that strict inequality is transitive.

```
<-trans : ∀ {m n p : ℕ} → m < n → n < p → m < p
<-trans z<s (s<s n<p) = z<s
<-trans (s<s m<n) (s<s n<p) = s<s (<-trans m<n n<p)
-- Your code goes here
```

#### Exercise `trichotomy` (practice) {#trichotomy}

Show that strict inequality satisfies a weak version of trichotomy, in
the sense that for any `m` and `n` that one of the following holds:
  * `m < n`,
  * `m ≡ n`, or
  * `m > n`.

Define `m > n` to be the same as `n < m`.
You will need a suitable data declaration,
similar to that used for totality.
(We will show that the three cases are exclusive after we introduce
[negation]({{ site.baseurl }}/Negation/).)

```
data Trichotomy (m n : ℕ) : Set where
  lt : m < n → Trichotomy m n
  gt : n < m → Trichotomy m n
  eq : m ≡ n → Trichotomy m n

<>≡trichotomy : ∀ (m n : ℕ) → Trichotomy m n
<>≡trichotomy zero zero = eq refl 
<>≡trichotomy zero (suc n) = lt z<s
<>≡trichotomy (suc m) zero = gt z<s  -- why not just gt n>m ? 
<>≡trichotomy (suc m) (suc n) with <>≡trichotomy m n
...                              | lt m<n = lt (s<s m<n)
...                              | gt n<m = gt (s<s n<m)
...                              | eq m≡n = eq (cong suc m≡n)

-- Your code goes here
```

#### Exercise `+-mono-<` (practice) {#plus-mono-less}

Show that addition is monotonic with respect to strict inequality.
As with inequality, some additional definitions may be required.

```
+-mono-r-< : ∀ (n p q : ℕ) → p < q → n + p < n + q   
+-mono-r-< zero  p q p<q  =  p<q
+-mono-r-< (suc n) p q p<q  =  s<s (+-mono-r-< n p q p<q)

+-mono-l-< : ∀ (m n p : ℕ) → m < n → m + p < n + p
+-mono-l-< m n p m<n rewrite +-comm m p | +-comm n p = +-mono-r-< p m n m<n

+-mono-< : ∀ (m n p q : ℕ) → m < n → p < q → m + p < n + q
+-mono-< m n p q m<n p<q = <-trans (+-mono-l-< m n p m<n) (+-mono-r-< n p q p<q)

-- Your code goes here
```

#### Exercise `≤-iff-<` (recommended) {#leq-iff-less}

Show that `suc m ≤ n` implies `m < n`, and conversely.

```
<-if-≤  : ∀ {m n : ℕ} → (suc m) ≤ n → m < n
<-if-≤  {zero} {suc n} m≤n = z<s
<-if-≤  {suc m} {suc n} m≤n = s<s (<-if-≤ (inv-s≤s m≤n))  -- I have these: s<s s≤s z<s z≤n
--have (suc (suc m)) ≤ (suc n)

inv-s<s : ∀ {m n : ℕ} → suc m < suc n → m < n
inv-s<s (s<s m<n) = m<n

≤-if-< : ∀ {m n : ℕ} → m < n → (suc m) ≤ n
≤-if-< {zero} {suc n} m<n = s≤s z≤n  
≤-if-< {suc m} {suc n} m<n = s≤s (≤-if-< (inv-s<s m<n))

-- Your code goes here
```

#### Exercise `<-trans-revisited` (practice) {#less-trans-revisited}

Give an alternative proof that strict inequality is transitive,
using the relation between strict inequality and inequality and
the fact that inequality is transitive.
```
n≤sn : ∀ {n : ℕ} → n ≤ suc n
n≤sn {n} = +-monoˡ-≤ 0 1 n z≤n

<-trans-revisited : ∀ {m n p : ℕ} → m < n → n < p → m < p
<-trans-revisited m<n n<p = <-if-≤ (≤-trans (≤-if-< m<n) (≤-trans n≤sn (≤-if-< n<p)))

```
if1 : ∀ {m n : ℕ} → m < n → suc m ≤ n
  if1 z<s = ≤-if-< z<s
  if1 (s<s m<n) = ≤-if-< (s<s m<n)

  if2 : ∀ {n p : ℕ} → n < p → suc n ≤ p
 -- if2 z<s = ≤-if-< z<s
  if2 (s<s m<n) = ≤-if-< (s<s m<n)
```
--(≤-if-< m<n) (≤-if-< n<p)) 

-- Your code goes here
```


## Even and odd

As a further example, let's specify even and odd numbers.  Inequality
and strict inequality are _binary relations_, while even and odd are
_unary relations_, sometimes called _predicates_:
```
data even : ℕ → Set
data odd  : ℕ → Set

data even where

  zero :
      ---------
      even zero

  suc  : ∀ {n : ℕ}
    → odd n
      ------------
    → even (suc n)

data odd where

  suc   : ∀ {n : ℕ}
    → even n
      -----------
    → odd (suc n)
```
A number is even if it is zero or the successor of an odd number,
and odd if it is the successor of an even number.

This is our first use of a mutually recursive datatype declaration.
Since each identifier must be defined before it is used, we first
declare the indexed types `even` and `odd` (omitting the `where`
keyword and the declarations of the constructors) and then
declare the constructors (omitting the signatures `ℕ → Set`
which were given earlier).

This is also our first use of _overloaded_ constructors,
that is, using the same name for constructors of different types.
Here `suc` means one of three constructors:

    suc : ℕ → ℕ

    suc : ∀ {n : ℕ}
      → odd n
        ------------
      → even (suc n)

    suc : ∀ {n : ℕ}
      → even n
        -----------
      → odd (suc n)

Similarly, `zero` refers to one of two constructors. Due to how it
does type inference, Agda does not allow overloading of defined names,
but does allow overloading of constructors.  It is recommended that
one restrict overloading to related meanings, as we have done here,
but it is not required.

We show that the sum of two even numbers is even:
```
e+e≡e : ∀ {m n : ℕ}
  → even m
  → even n
    ------------
  → even (m + n)

o+e≡o : ∀ {m n : ℕ}
  → odd m
  → even n
    -----------
  → odd (m + n)

e+e≡e zero     en  =  en
e+e≡e (suc om) en  =  suc (o+e≡o om en)

o+e≡o (suc em) en  =  suc (e+e≡e em en)
```
Corresponding to the mutually recursive types, we use two mutually recursive
functions, one to show that the sum of two even numbers is even, and the other
to show that the sum of an odd and an even number is odd.

This is our first use of mutually recursive functions.  Since each identifier
must be defined before it is used, we first give the signatures for both
functions and then the equations that define them.

To show that the sum of two even numbers is even, consider the
evidence that the first number is even. If it is because it is zero,
then the sum is even because the second number is even.  If it is
because it is the successor of an odd number, then the result is even
because it is the successor of the sum of an odd and an even number,
which is odd.

To show that the sum of an odd and even number is odd, consider the
evidence that the first number is odd. If it is because it is the
successor of an even number, then the result is odd because it is the
successor of the sum of two even numbers, which is even.

#### Exercise `o+o≡e` (stretch) {#odd-plus-odd}

Show that the sum of two odd numbers is even.

```
o+o≡e : ∀ {m n : ℕ} → odd m → odd n → even (m + n)
e+o≡o : ∀ {m n : ℕ} → even m → odd n → odd (m + n)

o+o≡e (suc em) on =  suc (e+o≡o em on)
e+o≡o (suc om) on  =  suc (o+o≡e om on)
e+o≡o zero on = on
-- Your code goes here
```

#### Exercise `Bin-predicates` (stretch) {#Bin-predicates}

Recall that
Exercise [Bin]({{ site.baseurl }}/Naturals/#Bin)
defines a datatype `Bin` of bitstrings representing natural numbers.
Representations are not unique due to leading zeros.
Hence, eleven may be represented by both of the following:

    ⟨⟩ I O I I
    ⟨⟩ O O I O I I

Define a predicate

    Can : Bin → Set

over all bitstrings that holds if the bitstring is canonical, meaning
it has no leading zeros; the first representation of eleven above is
canonical, and the second is not.  To define it, you will need an
auxiliary predicate

    One : Bin → Set

```
data Bin : Set where
  ⟨⟩ : Bin
  _O : Bin → Bin
  _I : Bin → Bin

inc : Bin → Bin
inc ⟨⟩ = (⟨⟩ I)
inc (x O) = (x I)
inc (x I) = ((inc x) O)

to : ℕ → Bin
to 0 = (⟨⟩ O)
to (suc n) = (inc (to n))

from : Bin → ℕ
from ⟨⟩ = zero
from (x O) = 2 * (from x)
from (x I) = suc (2 * (from x))

-- is my problem that I define One to be just is the pred was One? So kinda defined here in
  -- terms of Bin instead of just a series of I and O that happen to start w I
  -- alt def:
```
  data One : Bin → Set where
  isOne : One (x1 nil)
  zeroAfter : ∀ {b} → One b → One (x0 b)
  oneAfter : ∀ {b} → One b → One (x1 b)
```
data One : Bin → Set where
  bin-one : One (⟨⟩ I)
  bin-inc-one : ∀ {b : Bin} → One b → One (inc b)

data Can : Bin → Set where
  single-zero-bin : Can (⟨⟩ O)
  can-bin-inc-one : ∀ {b : Bin} → One b → Can b 
```

that holds only if the bistring has a leading one.  A bitstring is
canonical if it has a leading one (representing a positive number) or
if it consists of a single zero (representing zero).

Show that increment preserves canonical bitstrings:

    Can b
    ------------
    Can (inc b)

Show that converting a natural to a bitstring always yields a
canonical bitstring:

x    ----------
    Can (to n)

Show that converting a canonical bitstring to a natural
and back is the identity:

    Can b
    ---------------
    to (from b) ≡ b

(Hint: For each of these, you may first need to prove related
properties of `One`.)

```
incCan->Can : ∀ {b : Bin} → Can b -> Can (inc b)
incCan->Can single-zero-bin = can-bin-inc-one bin-one
incCan->Can (can-bin-inc-one ob) = can-bin-inc-one (bin-inc-one ob) 

*-identityzr : ∀ (m : ℕ) → m * zero ≡ zero
*-identityzr zero =
  begin
    zero * zero
  ≡⟨⟩
    zero
  ∎
*-identityzr (suc m) =
  begin
    suc m * zero
  ≡⟨⟩
    zero + (m * zero)
  ≡⟨⟩
    m * zero
  ≡⟨ *-identityzr m ⟩
    zero
  ∎


twoTimesFromBin : ∀ (b : Bin) → 2 * (from b) ≡ from (b O)
twoTimesFromBin ⟨⟩ =
  begin
    2 * (from ⟨⟩)
  ≡⟨⟩
    2 * zero
  ≡⟨ *-identityzr 2 ⟩
    zero
  ≡⟨⟩
    from (⟨⟩ O)
  ∎
twoTimesFromBin b =
  begin
    2 * (from b)
  ≡⟨⟩
    from (b O)
  ∎

+-assoc : ∀ (m n p : ℕ) → (m + n) + p ≡ m + (n + p)
+-assoc zero n p =
  begin
    (zero + n) + p
  ≡⟨⟩
    n + p
  ≡⟨⟩
    zero + (n + p)
  ∎
+-assoc (suc m) n p =
  begin
    (suc m + n) + p
  ≡⟨⟩
    suc (m + n) + p
  ≡⟨⟩
    suc ((m + n) + p)
  ≡⟨ cong suc (+-assoc m n p) ⟩
    suc (m + (n + p))
  ≡⟨⟩
    suc m + (n + p)
  ∎


_*_toMult : ∀ (m p : ℕ) → (suc m) * p ≡ (p + (m * p))
_*_toMult zero p =
  begin
    suc zero * p
  ≡⟨⟩
    p + (zero * p)
  ∎
_*_toMult (suc m) p =
  begin
    suc (suc m) * p
  ≡⟨⟩
    p + (suc m * p)
  ∎
``` 

*-distrib-+ : ∀ (m n p : ℕ) → (m + n) * p ≡ m * p + n * p
*-distrib-+ zero n p =
  begin
    (zero + n) * p
  ≡⟨⟩
    n * p
  ≡⟨⟩
    zero * p + n * p
  ∎
*-distrib-+ (suc m) n p =
  begin
    (suc m + n) * p
  ≡⟨⟩
    suc (m + n) * p
  ≡⟨⟩
    p + ((m + n) * p)
  ≡⟨ cong (p +_) (*-distrib-+ m n p) ⟩
    p + ((m * p) + (n * p))
  ≡⟨ sym (+-assoc p (m * p) (n * p))⟩
    (p + (m * p)) + (n * p)
  ≡⟨⟩
_*_toMult : ∀ (m p : ℕ) → (suc m) * p ≡ (p + (m * p))
_*_toMult zero p =
  begin
    suc zero * p
  ≡⟨⟩
    p + (zero * p)
  ∎
_*_toMult (suc m) p =
  begin
    suc (suc m) * p
  ≡⟨⟩
    p + (suc m * p)
  ∎
``` 

*-distrib-+ : ∀ (m n p : ℕ) → (m + n) * p ≡ m * p + n * p
*-distrib-+ zero n p =
  begin
    (zero + n) * p
  ≡⟨⟩
    n * p
  ≡⟨⟩
    zero * p + n * p
  ∎
*-distrib-+ (suc m) n p =
  begin
    (suc m + n) * p
  ≡⟨⟩
    suc (m + n) * p
  ≡⟨⟩
    p + ((m + n) * p)
  ≡⟨ cong (p +_) (*-distrib-+ m n p) ⟩
    p + ((m * p) + (n * p))
  ≡⟨ sym (+-assoc p (m * p) (n * p))⟩
    (p + (m * p)) + (n * p)
  ≡⟨⟩
    ((suc m) * p) + (n * p)
  ∎



+-suc-inc-Bin : ∀ (b : Bin) →  from (inc b) ≡ suc (from b)
+-suc-inc-Bin ⟨⟩ = refl
+-suc-inc-Bin (b O) = refl
+-suc-inc-Bin (b I) =
  begin
    from (inc (b I))
  ≡⟨⟩
    from ((inc b) O)
  ≡⟨⟩
    2 * (from (inc b))
  ≡⟨ cong (2 *_ ) (+-suc-inc-Bin b) ⟩
    2 * (1 + (from b))
  ≡⟨ *-comm 2 (1 + (from b)) ⟩
    (1 + (from b)) * 2
  ≡⟨ *-distrib-+ 1 (from b) 2 ⟩
    1 * 2 + (from b) * 2
  ≡⟨⟩
    2 + ((from b) * 2)
  ≡⟨ cong (2 +_ ) (*-comm (from b) 2) ⟩
    2 + (2 * (from b))
  ≡⟨ cong (2 +_ ) (twoTimesFromBin b) ⟩
    2 + (from (b O))
  ≡⟨⟩
    suc (from (b I))
  ∎
```
+-suc-inc-Bin : ∀ (b : Bin) →  from (inc b) ≡ suc (from b)
I need   (to (suc n)) = inc (to n)
incCan->Can : ∀ {b : Bin} → Can b → Can (inc b) 
```

incOne->One : ∀ {b : Bin} → One b → One (inc b)
incOne->One bin-one = bin-inc-one bin-one
incOne->One (bin-inc-one b) = bin-inc-one (incOne->One b)

toN->One : ∀ {n : ℕ} → One (to (suc n))
toN->One {zero} = bin-one
toN->One {suc n} = bin-inc-one (toN->One {n})



toN->Can : ∀ {n : ℕ} → Can (to n)
toN->Can {zero} = single-zero-bin
toN->Can {suc n} = incCan->Can (toN->Can {n})

-- only true when bin /= ⟨⟩ or ⟨⟩ O
```
one≤fromB : ∀ (b : Bin) → 1 ≤ from (inc b)
one≤fromB ⟨⟩ = s≤s z≤n
```

binFromOne : ∀ {b : Bin} → One b -> Bin
binFromOne bin-one = ⟨⟩ I
binFromOne (bin-inc-one b) = to (1 + (from (binFromOne b)))

binFromCan : ∀ {b : Bin} → Can b -> Bin
binFromCan single-zero-bin = ⟨⟩ O
binFromCan (can-bin-inc-one o) = binFromOne o
```
-- I need One b → to (from b) ≡ b
oneB->idB : ∀ {b : Bin} → One b → to (from b) ≡ b
oneB->idB bin-one = refl
oneB->idB (bin-inc-one o) = to (from (binFromOne o)) ≡ binFromOne o
```
subst : ∀ {A : Set} {x y : A} (P : A → Set)
  → x ≡ y
    ---------
  → P x → P y
subst P refl px = px

cong' : ∀ {A B : Set} (f : A → B) {x y : A}
  → x ≡ y
    ---------
  → f x ≡ f y
cong' f refl  =  refl

CanA->CanB : ∀ {a b : Bin} → a ≡ b → Can a → Can b
CanA->CanB x y = subst Can x y
```
-- use binFromCan
CanAB->AB : ∀ {a b : Bin} → Can a ≡ Can b → a ≡ b
CanAB->AB x = cong' binFromCan x
```
```
--Can b → to (from b) ≡ b
canB->idB : ∀ {b : Bin} → Can b → to (from b) ≡ b
canB->idB single-zero-bin = refl
canB->idB (can-bin-inc-one bin-one) = refl
canB->idB (can-bin-inc-one (bin-inc-one b)) = toN->One {b} ≡ b
```
--Your code goes here
```
data One : Bin → Set where
  bin-one : One (⟨⟩ I)
  bin-inc-one : ∀ {b : Bin} → One b → One (inc b) 

data Can : Bin → Set where
  single-zero-bin : Can (⟨⟩ O)
  can-bin-inc-one : ∀ {b : Bin} → One b → Can b 
## Standard library

Definitions similar to those in this chapter can be found in the standard library:
```
import Data.Nat using (_≤_; z≤n; s≤s)
import Data.Nat.Properties using (≤-refl; ≤-trans; ≤-antisym; ≤-total;
                                  +-monoʳ-≤; +-monoˡ-≤; +-mono-≤)
```
In the standard library, `≤-total` is formalised in terms of
disjunction (which we define in
Chapter [Connectives]({{ site.baseurl }}/Connectives/)),
and `+-monoʳ-≤`, `+-monoˡ-≤`, `+-mono-≤` are proved differently than here,
and more arguments are implicit.


## Unicode

This chapter uses the following unicode:

    ≤  U+2264  LESS-THAN OR EQUAL TO (\<=, \le)
    ≥  U+2265  GREATER-THAN OR EQUAL TO (\>=, \ge)
    ˡ  U+02E1  MODIFIER LETTER SMALL L (\^l)
    ʳ  U+02B3  MODIFIER LETTER SMALL R (\^r)

The commands `\^l` and `\^r` give access to a variety of superscript
leftward and rightward arrows in addition to superscript letters `l` and `r`.