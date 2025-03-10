<!--
```agda
open import Cat.Functor.Properties
open import Cat.Prelude

import Cat.Reasoning
```
-->

```agda
module Cat.Functor.WideSubcategory {o h} {C : Precategory o h} where
```

<!--
```agda
private module C = Cat.Reasoning C
open Precategory
private variable
  ℓ : Level
```
-->

# Wide subcategories

A **wide subcategory** $\cD \mono \cC$ is specified by a [predicate] $P$
on the morphisms of $\cC$, rather than one on the objects. Since $P$ is
nontrivial, we must take care that the result actually form a category:
$P$ must contain the identities and be closed under composition.

[predicate]: 1Lab.HLevel.html#is-prop

To start, we package up all the data required to define a wide
subcategory up into a record.

```agda
record Wide-subcat (ℓ : Level) : Type (o ⊔ h ⊔ lsuc ℓ) where
  no-eta-equality
  field
    P      : ∀ {x y} → C.Hom x y → Type ℓ
    P-prop : ∀ {x y} (f : C.Hom x y) → is-prop (P f)

    P-id : ∀ {x} → P (C.id {x})
    P-∘  : ∀ {x y z} {f : C.Hom y z} {g : C.Hom x y}
         → P f → P g → P (f C.∘ g)

open Wide-subcat
```

Morphisms of wide subcategories are defined as morphisms in $\cC$ where
$P$ holds.

```agda
record Wide-hom (sub : Wide-subcat ℓ) (x y : C.Ob) : Type (h ⊔ ℓ) where
  no-eta-equality
  constructor wide
  field
    hom : C.Hom x y
    witness : sub .P hom
```

<!--
```agda
open Wide-hom public

Wide-hom-path
  : {sub : Wide-subcat ℓ}
  → {x y : C.Ob}
  → {f g : Wide-hom sub x y}
  → f .hom ≡ g .hom
  → f ≡ g
Wide-hom-path {f = f} {g = g} p i .hom = p i
Wide-hom-path {sub = sub} {f = f} {g = g} p i .witness =
  is-prop→pathp (λ i → sub .P-prop (p i)) (f .witness) (g .witness) i

Wide-hom-is-set
  : {sub : Wide-subcat ℓ}
  → {x y : C.Ob}
  → is-set (Wide-hom sub x y)
Wide-hom-is-set {sub = sub} = Iso→is-hlevel 2 eqv $
  Σ-is-hlevel 2 (C.Hom-set _ _) λ f → is-hlevel-suc 1 (sub .P-prop f)
  where unquoteDecl eqv = declare-record-iso eqv (quote Wide-hom)
```
-->

We can then use this data to construct a category.

```agda
Wide : Wide-subcat ℓ → Precategory o (h ⊔ ℓ)
Wide sub .Ob = C.Ob
Wide sub .Hom         = Wide-hom sub
Wide sub .Hom-set _ _ = Wide-hom-is-set

Wide sub .id .hom     = C.id
Wide sub .id .witness = sub .P-id

Wide sub ._∘_ f g .hom     = f .hom C.∘ g .hom
Wide sub ._∘_ f g .witness = sub .P-∘ (f .witness) (g .witness)

Wide sub .idr _ = Wide-hom-path $ C.idr _
Wide sub .idl _ = Wide-hom-path $ C.idl _
Wide sub .assoc _ _ _ = Wide-hom-path $ C.assoc _ _ _
```

## From split essentially surjective inclusions

There is another way of representing wide subcategories: By giving a
[[pseudomonic]] [[split essential surjection]] $F : \cD \epi \cC$.

<!--
```agda
module _ {o' h'} {D : Precategory o' h'} {F : Functor D C}
  (pseudomonic : is-pseudomonic F)
  (eso : is-split-eso F)
  where
  open Functor F
  private
    module D = Cat.Reasoning D
    module eso x =  C._≅_ (eso x .snd)
```
-->

We construct the wide subcategory by restricting to the morphisms in
$\cC$ that lie in the image of $F$. Since $F$ is a [[faithful functor]],
this is indeed a proposition.

```agda
  Split-eso-inclusion→Wide-subcat : Precategory _ _
  Split-eso-inclusion→Wide-subcat = Wide sub where
    sub : Wide-subcat (h ⊔ h')
    sub .P {x = x} {y = y} f =
      Σ[ g ∈ D.Hom (eso x .fst) (eso y .fst) ]
      (eso.to y C.∘ F₁ g C.∘ eso.from x ≡ f)
    sub .P-prop {x} {y} f (g , p) (g' , q) =
      Σ-prop-path (λ _ → C.Hom-set _ _ _ _) $
      is-pseudomonic.faithful pseudomonic   $
      C.iso→epic (eso x .snd C.Iso⁻¹) _ _   $
      C.iso→monic (eso y .snd) _ _ (p ∙ sym q)
    sub .P-id {x} =
      (D.id , ap₂ C._∘_ refl (C.eliml F-id) ∙ C.invl (eso x .snd))
    sub .P-∘ {x} {y} {z} {f} {g} (f' , p) (g' , q) =
      f' D.∘ g' , (
        eso.to z C.∘ F₁ (f' D.∘ g') C.∘ eso.from x                                    ≡⟨ C.push-inner (F-∘ f' g') ⟩
        (eso.to z C.∘ F₁ f') C.∘ (F₁ g' C.∘ eso.from x)                               ≡⟨ C.insert-inner (eso.invr y) ⟩
        ((eso.to z C.∘ F₁ f') C.∘ eso.from y) C.∘ (eso.to y C.∘ F₁ g' C.∘ eso.from x) ≡⟨ ap₂ C._∘_ (sym (C.assoc _ _ _) ∙ p) q ⟩
        f C.∘ g ∎)
```

This canonical wide subcategory is equivalent to $\cD$.

```agda
  Wide-subcat→Split-eso-domain : Functor (Split-eso-inclusion→Wide-subcat) D
  Wide-subcat→Split-eso-domain .Functor.F₀ x = eso x .fst
  Wide-subcat→Split-eso-domain .Functor.F₁ f = f .witness .fst
  Wide-subcat→Split-eso-domain .Functor.F-id = refl
  Wide-subcat→Split-eso-domain .Functor.F-∘ _ _ = refl

  is-fully-faithful-Wide-subcat→domain : is-fully-faithful Wide-subcat→Split-eso-domain
  is-fully-faithful-Wide-subcat→domain = is-iso→is-equiv $ iso
    (λ f → wide (eso.to _ C.∘ F₁ f C.∘ eso.from _) (f , refl))
    (λ _ → refl)
    (λ f → Wide-hom-path (f .witness .snd))

  is-eso-Wide-subcat→domain : is-split-eso Wide-subcat→Split-eso-domain
  is-eso-Wide-subcat→domain x =
    F₀ x , pseudomonic→essentially-injective pseudomonic (eso (F₀ x) .snd)
```

We did cheat a bit earlier when defining wide subcategories: our
predicate isn't required to respect isomorphisms! This means that we
could form a "subcategory" that kills off all the isomorphisms, which
allows us to distinguish between isomorphic objects. Therefore,
wide subcategories are not invariant under equivalence of categories.

This in turn means that the forgetful functor from a wide subcategory is
*not* pseudomonic! To ensure that it is, we need to require that the
predicate holds on all isomorphisms. Arguably this is something that
should be part of the definition of a wide subcategory, but it isn't
**strictly** required, so we opt to leave it as a side condition.

<!--
```agda
module _ {sub : Wide-subcat ℓ} where
  private module Wide = Cat.Reasoning (Wide sub)
```
-->

```agda
  Forget-wide-subcat : Functor (Wide sub) C
  Forget-wide-subcat .Functor.F₀ x = x
  Forget-wide-subcat .Functor.F₁ f = f .hom
  Forget-wide-subcat .Functor.F-id = refl
  Forget-wide-subcat .Functor.F-∘ _ _ = refl

  is-faithful-Forget-wide-subcat : is-faithful Forget-wide-subcat
  is-faithful-Forget-wide-subcat = Wide-hom-path

  is-pseudomonic-Forget-wide-subcat
    : (P-invert : ∀ {x y} {f : C.Hom x y} → C.is-invertible f → sub .P f)
    → is-pseudomonic Forget-wide-subcat
  is-pseudomonic-Forget-wide-subcat P-invert .is-pseudomonic.faithful =
    is-faithful-Forget-wide-subcat
  is-pseudomonic-Forget-wide-subcat P-invert .is-pseudomonic.isos-full f =
    pure $
      Wide.make-iso
        (wide f.to (P-invert (C.iso→invertible f)))
        (wide f.from (P-invert (C.iso→invertible (f C.Iso⁻¹))))
        (Wide-hom-path f.invl)
        (Wide-hom-path f.invr) ,
      C.≅-pathp refl refl refl
    where module f = C._≅_ f

  is-split-eso-Forget-wide-subcat : is-split-eso Forget-wide-subcat
  is-split-eso-Forget-wide-subcat y = y , C.id-iso
```
