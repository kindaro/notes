## Finite maps, record sets and the initial map.

In the type system of Haskell, there is no subtyping. There are some roundabout
ways to provide type safe subtypes though. Notably, when the predicate is
defined ahead of time _(example: strictly ascending lists — a representation of
sets — as a subtype of all lists)_, abstract types with smart constructors may
be used. When a predicate is variable or arbitrary, there is no way to express
the subtypes thus obtained in the type system. But we can still think of them as
living in the category **Set** of sets and total functions — only in Haskell we
shall have to use partial functions instead.

A special case is finite sets. We can store all values of a finite set, and we
can define a finite map pointwise. It would, again, appear partial in the type
system of Haskell, but it is total on a known finite set of keys, so we can
always check that a given _«unsafe»_ access is valid. Nevertheless, I am going
to concede to the type system and speak of finite maps on subtypes as if they
are partial maps between the usual Haskell types. For such finite sets and maps,
we have ready-made definitions in `containers`.

I introduce a synonym _α ⇸ β_ for a partial map `Data.Map.Map α β` for ease of
reading. Note that these arrows compose. It is not feasible to define an
identity map in Haskell, but we can imagine that it is there and say that there
is a category of finite maps. Composition is also broken in the presence of
non-trivial equality of key sets, but I am going to ignore that as well because
we only use trivial equality where a value is equal to itself and nothing else.

I also use a notation _{A}_ for a finite set of values of type _A_, so there is
a monic arrow _{A} → A_ — this is analogous to the usual notation _[A]_ for a
list.

There is another, type safe way in Haskell to represent sets, but only of known
size — as record types. If a set is represented this way, type safe projections
are gained for free. A curious example is a fully general record, where each
field is its own type variable. Such records are used extensively in our Opaleye
code. You may consider a monomorphic record as a _«column set»_ of a table, with
typing judgement associated to every column.

Particularly, a table in a data base contains a set of identically typed
monomorphic records that we can extract. All the work we are going to do will be
with such a record set. It is the most informative entity we can possibly have,
so in some sense it is an initial object in the subcategory of **Set** that we
concern ourselves with.

Assume a type synonym _Γ_ for the type of records, so _x: Γ_ is a single entry
in the data base. An identity arrow on a record set has the type _Γ ⇸ Γ_ — it is
defined on a known subtype of _Γ_ and nowhere else. Looks boring, but see that
you may use _fmap (projection: Γ → X)_ to transform this map into _Γ ⇸ X_ — an
easy way to turn Haskell functions _Γ → α_ into finite maps! This is why the
identity map on the initial object is central to our work here.

## Fiber inverse, lattice of projections and lattice of equivalence classes.

Any projection _f: Γ ⇸ X_ has a finite range that we may recognize as a finite
subtype _X' ⊂ X_, so that the projection may be thought of as an epic arrow _Γ'
→ X'_ in the category of finite sets and total maps. Any epic arrow is a
quotient map for the source, so it induces a slicing of the source into
equivalence classes. We may obtain this slicing in Haskell as _f⁻¹: X ⇸ {Γ}_. I
call it _«fiber inverse»_ because the equivalence classes are the fibers of the
projection _f_.

What does this practically mean? For example, take the standard table
[`information_schema.columns`] — this table describes all the columns of all
other tables in the data base, with their types, nullability and so on. It has a
column `table_schema`. If we project along this column with _`table_schema`: Γ →
type of `table_schema`_, the fiber inverse _`table_schema` ⇸ {Γ}_ is a
classification of columns by the table schema which tables they are columns
of. Similarly with the column `table_name`.

Now apply fiber inverse to `table_schema` first and then map the application of
`table_name` over the result. Since a fiber inverse has finite sets _{Γ}_ as
their target type, this is perfectly legal and gives _`table_schema` ⇸
`table_name` ⇸ {Γ}_ — a classification of columns by schema and table! In other
words, every _{Γ}_ here is a finite set of columns in a given table of a given
schema. See also that we have the complete record type in the end, so we can
study it further — there is no loss of information.

Since we consider finite maps to be a subcategory of **Set**, which is cartesian
closed, we may curry, uncurry, partially apply and overall feel like home. In
particular, the map we obtained is the same as _⟨`table_schema`, `table_name`⟩ ⇸
{Γ}_ and we may add any number of projections here.

You can see from here that the lattice of subsets of the column set corresponds
to the fiber inverses we may take and induces a lattice structure on the
latter. For example, _⟨`table_schema`, `table_name`⟩ ⇸ {Γ}_ is a _«finer»_ fiber
inverse than _`table_schema` ⇸ {Γ}_. So what does it mean for one fiber inverse
to be _«finer»_ than another? It means that the equivalence classes the former
slices the source into are all subsets of the equivalence classes of the
latter. So, for example, the set of columns in a table is a subset of the
columns of the schema the table belongs to. We find that the lattice of
equivalence classes and refinements of a record set of a table corresponds to
the lattice of subsets of the column set of that table.


[`information_schema.columns`]: https://www.postgresql.org/docs/13/infoschema-columns.html
