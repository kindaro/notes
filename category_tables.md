## Finite maps, record sets and the initial map.

### Ways to express finite sets and finite maps in Haskell.

In the type system of Haskell, there is no subtyping: every value belongs to
exactly one type. There are some roundabout ways to provide type safe subtypes
though. Notably, when the predicate is defined ahead of time _(example: strictly
ascending lists — a representation of sets — as a subtype of all lists)_,
abstract types with smart constructors may be used. When a predicate is variable
or arbitrary, there is no way to express the subtypes thus obtained in the type
system. But we can still think of them as living in the category **Set** of sets
and total functions — only in Haskell we shall have to use partial functions
instead. This means that we often keep two worlds in mind: the world **Set** of
things and arrows as we know they are, and the world **Haskell** of things and
arrows that we can explain to the type system.

#### Finite sets and maps as abstract subtypes.

A special case of a subtype is a finite set. We can store all values of a finite
set, and we can define a finite map pointwise. It would appear partial in the
type system of Haskell, but it is total on a known finite set of keys, so we can
always check that a given _«unsafe»_ access to a value of a finite set is
valid. Nevertheless, I have to concede to the type system and speak of finite
maps as if they are partial maps between the usual Haskell types. For such
finite sets and maps, we have ready-made definitions in `containers`.

For any finite type an identity map may trivially be constructed. Also, finite
maps compose. So, they form the category **Finite Set** of finite sets and
finite total maps — a subcategory of **Set**. Unfortunately, this category
cannot be realized in the type system. It is impossible to assign a type to a
finite total identity map between two given Haskell types — because there are
many, corresponding to different finite subtypes. Composition is also broken in
the presence of non-trivial equality on keys. So, we need to carefully track
which work is done in **Finite Set** and which in **Haskell**, even though we
have a common language for them in **Set**.

##### Some notation for the types of finite containers.

I introduce a synonym _α ⇸ β_ for a partial map `Data.Map.Map α β` for ease of
reading. These arrows associate to the right, just like the usual Haskell
function arrows, so _α ⇸ β ⇸ γ_ means _α ⇸ (β ⇸ γ)_.

I also use a notation _{A}_ for the type of finite sets of values of type _A_,
so there is a monic arrow _id: (Α': {A}) ↣ A_ — this is analogous to the usual
notation _[A]_ for the list type. I shall then write _keys (f: A ⇸ B): {A}_ for
the set of values where _f_ is defined and _range (f: A ⇸ B): {B}_ for the set
of values _f_ takes its keys to.

#### Sets of fixed size.

There is another, type safe way in Haskell to represent sets, but only of
statically known size — as values of record types. If a set is represented this
way, type safe projections are gained for free. You may also see that a record
type is a finite map of known size by itself — a map from field labels to their
types. We may use set operations here, so one record type may be a subset of
another, or a union of two.

### Data base records.

A table in a data base contains a set of identically typed records that we can
extract. All the work we are going to do will be with such a set of records. It
is the most informative entity we can possibly have, so in some sense it is an
initial object in the subcategory of **Set** that we concern ourselves with.

Since all the records in a table are of the same type, we may see that the type
of a record is the same as the definition of the associated table, and we call
the field labels of the record type a _«column set»_. The subsets of the column
set then correspond to the views of the associated table.

### Initial map.

Assume a type synonym _Γ_ for the type of records, so _x: Γ_ is a single entry
in the associated table. An identity arrow on a record set has the type _Γ ⇸ Γ_
— it is defined on a finite subtype _Γ' = {Γ} ⊂ Γ_ and nowhere else. Looks
boring, but see that you may use _fmap (projection: Γ → X)_ to transform this
map into _Γ ⇸ X_ — an easy way to turn Haskell functions _Γ → α_ into finite
maps defined exactly on _Γ'_ — you may also see this as restriction of _projection_ to _Γ'_. This is why the identity map on the initial object
is central to our work here.

## Fiber inverse, lattice of projections and lattice of equivalence classes.

### Fiber inverse.

Any projection _f: Γ → X_, restricted to some finite set _Γ'_, has a finite
range that we may recognize as a finite subtype _X' ⊂ X_, so that the projection
may be thought of as an epic arrow _Γ' ↠ X'_ in the category of finite sets and
total maps. Any epic arrow is a quotient map for the source, so it induces a
slicing of the source into equivalence classes. We may obtain this slicing in
Haskell as _f⁻¹: X ⇸ {Γ}_. I call it _«fiber inverse»_ because the equivalence
classes are the fibers of _f_. So, whenever we have a finite set _Γ'_ and a
projection _f: Γ → X_, we may pass to a fiber inverse _f⁻¹: X ⇸ {Γ}_ defined
exactly on _Γ'_.

#### An example.

What does this practically mean? For example, take the standard table
[`information_schema.columns`] — this table describes all the columns of all
other tables in the data base, with their types, nullability and so on. It has a
column `table_schema`. Recall that the columns of a table correspond to the
field labels of the associated record type, which also act as projections. So,
if we project along the column `table_schema` with _`table_schema`: Γ → type of
`table_schema`_, the fiber inverse _`table_schema`⁻¹: type of `table_schema` ⇸
{Γ}_ is a classification of columns by the schema which tables they are the
columns of. Similarly, the column `table_name` induces a classification of
columns by the name of the table they are the columns of, and so on.

Now apply fiber inverse to `table_schema` first and then map the application of
`table_name` over the result. Since a fiber inverse has finite sets _{Γ}_ as
their target type, this is perfectly legal and gives _`table_schema` ⇸ Γ ⇸
`table_name`_. You may map fiber inverse now to obtain _`table_schema` ⇸
`table_name` ⇸ {Γ}_ _(remember that arrows are right associative)_ — a
classification of columns by schema and table! In other words, every _{Γ}_ here
is a finite set of columns in a given table of a given schema. See also that we
have the complete record type at the end, so we can study it further — there is
no loss of information.

### Lattice of projections.

Since the category **Finite Set** of finite sets and maps is a subcategory of
**Set**, which is cartesian closed, we may curry, uncurry, partially apply and
overall feel like home. In particular, the map we obtained above is the same as
_⟨`table_schema`; `table_name`⟩ ⇸ {Γ}_ and we may add any number of projections
on the left here.

You can see from here that the lattice of subsets of the column set corresponds
to the collection of fiber inverses we may take and induces a lattice structure
on the latter. For example, _⟨`table_schema`; `table_name`⟩ ⇸ {Γ}_ is a
_«finer»_ fiber inverse than _`table_schema` ⇸ {Γ}_.

### Lattice of equivalence classes.

So what does it mean for one fiber inverse to be _«finer»_ than another? It
means that the equivalence classes the former slices the source into are all
subsets of the equivalence classes of the latter. So, for example, the set of
columns in a table is a subset of the columns of the schema the table belongs
to. We find that the lattice of equivalence classes of a record set of a table
and their refinements corresponds to the lattice of subsets of the column set of
that table and their inclusions.

## Pre-pullbacks and fringes.

Consider two fibers _Γ₁, Γ₂: {Γ}_ of _`table_schema`_ restricted to a record set
_Γ'_. You may see that a fiber _Γᵢ = `table_schema`⁻¹ σᵢ_ contains all the
descriptions of columns belonging to the tables of the corresponding schema
_σᵢ_. What is a pullback of _Γ₁_ and _Γ₂_ along `table_schema`? It is a set of
all pairs of records that have the same `table_schema`. This sort of a
homogeneous soup of pairs is too big and not structured enough for our ends, but
we may stop short of computing it and have a smaller and better structured thing
sooner.

Given two maps _f: B ⇸ A_ and _g: C ⇸ A_, we may take fiber inverses _f⁻¹: A ⇸
{B}_ and _g⁻¹: A ⇸ {C}_ and combine them into _h = f⁻¹ ▵ g⁻¹ = λx. ⟨f⁻¹ x; g⁻¹
x⟩: A ⇸ ⟨{B}; {C}⟩_, so that whenever _h (x: A) = ⟨u; v⟩_, it is true that _∀ b
∈ u, c ∈ v. f b = g c_ insofar as both are defined. You may see that the usual
fiber product may be obtained by taking the cartesian products of these pairs of
sets, so _h_ is strictly more informative. I call it a _«pre-pullback»_. It is
also not hard to augment _h_ with the elements of the source sets of _f_ and _g_
that are not found anywhere in the fiber product — I shall be calling the
evident maps from _C_ to these sets the _«fringes»_ of a pullback.

So, to summarize, a _«pre-pullback with fringes»_ of sets _A_ and _B_ along some
arrows with a common target _C_ is a tuple of type _⟨C ⇸ {A}; C ⇸ {B} ;C ⇸ ⟨{A};
{B}⟩ ⟩_, that may be computed efficiently and from which a usual pullback may be
further obtained. Since it is just as good and even better than a usual fiber
product, I shall refer to this construction simply as _«pullback»_.


[`information_schema.columns`]: https://www.postgresql.org/docs/13/infoschema-columns.html
