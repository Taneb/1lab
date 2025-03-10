<!--
```agda
open import 1Lab.Prelude

open import Data.Nat.Properties
open import Data.Nat.Order
open import Data.Nat.Base
```
-->

```agda
module Data.Nat.Divisible where
```

# Divisibility

In the natural numbers, **divisibility**[^divide] is the property
expressing that a given number can be expressed as a multiple of
another, and we write $a | b$^[read "a divides b"] when there exists
some $k$ such that $b = ka$.

[^divide]: Not to be confused with a proper [division](Data.Nat.DivMod.html) algorithm.

Note the use of an existential quantifier, which is a bit annoying: For
divisibility to truly be a _property_, we must work under a truncation,
since otherwise there would be $\NN$-many proofs that $0 | 0$, since for
_any_ $n$, we have $0 = n0$. To avoid this sticky situation, we define
divisibility with a single step of indirection:

```agda
_∣_ : Nat → Nat → Type
zero  ∣ y = y ≡ zero
suc x ∣ y = fibre (_* suc x) y

infix 5 _∣_
```

In this way, we break the pathological case of $0 | 0$ by _decreeing_ it
to be the (contractible) type $0 = 0$. Every other natural number is
already handled, because the function "$(1 + x) *$" is injective. With
this indirection, we can prove that divisibility is a mere property:

```agda
∣-is-prop : ∀ x y → is-prop (x ∣ y)
∣-is-prop zero y n k = prop!
∣-is-prop (suc x) y (n , p) (n′ , q) = Σ-prop-path! (*-suc-inj x n n′ (p ∙ sym q))

instance
  H-Level-∣ : ∀ {x y} {n} → H-Level (x ∣ y) (suc n)
  H-Level-∣ = prop-instance (∣-is-prop _ _)
```

The type $x | y$ is, in fact, the [[propositional truncation]] of $(*
x)^{-1}(y)$ --- and it is logically equivalent to that type, too!

```agda
∣-is-truncation : ∀ {x y} → (x ∣ y) ≃ ∥ fibre (_* x) y ∥
∣-is-truncation {zero} {y} =
  prop-ext!
    (λ p → inc (y , *-zeror y ∙ sym p))
    (∥-∥-rec! (λ{ (x , p) → sym p ∙ *-zeror x }))
∣-is-truncation {suc x} {y} = Equiv.to is-prop≃equiv∥-∥ (∣-is-prop (suc x) y)

∣→fibre : ∀ {x y} → x ∣ y → fibre (_* x) y
∣→fibre {zero} wit  = 0 , sym wit
∣→fibre {suc x} wit = wit

fibre→∣ : ∀ {x y} → fibre (_* x) y → x ∣ y
fibre→∣ f = Equiv.from ∣-is-truncation (inc f)

divides : ∀ {x y} (q : Nat) → q * x ≡ y → x ∣ y
divides x p = fibre→∣ (x , p)
```

## As an ordering

The notion of divisibility equips the type $\NN$ with yet another
[[partial order]], i.e., the relation $x | y$ is reflexive, transitive,
and antisymmetric. We can establish the former two directly, but
antisymmetry will take a bit of working up to:

```agda
∣-refl : ∀ {x} → x ∣ x
∣-refl = divides 1 (*-onel _)

∣-trans : ∀ {x y z} → x ∣ y → y ∣ z → x ∣ z
∣-trans {zero} {zero} p q = q
∣-trans {zero} {suc y} p q = absurd (zero≠suc (sym p))
∣-trans {suc x} {zero} p q = 0 , sym q
∣-trans {suc x} {suc y} {z} (k , p) (k′ , q) = k′ * k , (
  k′ * k * suc x   ≡⟨ *-associative k′ k (suc x) ⟩
  k′ * (k * suc x) ≡⟨ ap (k′ *_) p ⟩
  k′ * suc y       ≡⟨ q ⟩
  z                ∎)
```

We note in passing that any number divides zero, and one divides every
number.

```agda
∣-zero : ∀ {x} → x ∣ 0
∣-zero = divides 0 refl

∣-one : ∀ {x} → 1 ∣ x
∣-one {x} = divides x (*-oner x)
```

A more important lemma is that if $x$ divides a non-zero number $y$,
then $x \le y$: the divisors of any positive $y$ are smaller than it.
Zero is a sticking point here since, again, any number divides zero!

```agda
m∣sn→m≤sn : ∀ {x y} → x ∣ suc y → x ≤ suc y
m∣sn→m≤sn {x} {y} p with ∣→fibre p
... | zero  , p = absurd (zero≠suc p)
... | suc k , p = difference→≤ (k * x) p
```

This will let us establish the antisymmetry we were looking for:

```agda
∣-antisym : ∀ {x y} → x ∣ y → y ∣ x → x ≡ y
∣-antisym {zero}  {y}     p q = sym p
∣-antisym {suc x} {zero}  p q = absurd (zero≠suc (sym q))
∣-antisym {suc x} {suc y} p q = ≤-antisym (m∣sn→m≤sn p) (m∣sn→m≤sn q)
```

## Elementary properties

Since divisibility is the "is-multiple-of" relation, we would certainly
expect a number to divide its multiples. Fortunately, this is the case:

```agda
∣-*l : ∀ {x y} → x ∣ x * y
∣-*l {y = y} = fibre→∣ (y , *-commutative y _)

∣-*r : ∀ {x y} → x ∣ y * x
∣-*r {y = y} = fibre→∣ (y , refl)
```
