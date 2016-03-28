---
layout: post
title: "Example-Directed Synthesis: a Type-Theoretic Interpretation"
author: michele
tags: program synthesis, type theory, proof search, refinement types
published: true
---

**Authors:** Frankle, Osera, Walker, Zdancewic\\
**Conference:** [POPL 2016](http://conf.researchr.org/event/POPL-2016/popl-2016-papers-example-directed-synthesis-a-type-theoretic-interpretation)


This is a very interesting paper about a cool and rather hot topic: **program
synthesis**. Since the paper has been accepted at POPL 2016, one expects a
theoretical investigation of the subject. As the title of the paper
suggests, it is indeed the case. A surprise comes from a quite detailed
explanation on the implementation challenges the authors have faced in
developing a prototype.

<!--more-->
-----

Program synthesis is about code generation with respect to some high-level
specification of a computational problem. This sounds like magic and,
although feasible only to some extent, can even be quite useful in
programming.

In recent years, many specification mechanisms have been proposed, some of them
inherited or influenced by research on program verification. For example, this
is the case for methods based on function contracts (as pre/post conditions).

One of the most user-friendly and intuitive way of specifying behaviors is
that of input-output examples. This has notably been proposed within
typeful settings such as functional programming languages. The idea is to
provide the programmer with a language to express the behavior (output) of
a function on a certain argument (input). So, for instance in a ML-like
language, one may write something like

          [] -> 0 && [1] -> 1 && [2;1] -> 2

to specify the function `length` on lists.

The paper discussed here deals with two major questions about
example-driven synthesis in this kind of settings:

- What is the semantical meaning of *examples*? In other words, is there a
  theoretical approach to reveal and exploit?
- Example based spefications can be verbose, so much that one may wonder if
  it is faster and terser to code altogether. In light of the first
  question, how to define a rich specification language in order to express
  concise examples?

The authors argue that each example can be given a type-theoretical
interpretation in the sense of a **refinement type**, hence synthesis the
process of finding an inhabitant of such a type. This *"[...] enables us to
exploit decades of research in type theory as well as its correspondence
with intuitionistic logic rather than designing ad hoc theoretical
frameworks for synthesis."*.

<div class="message">Although somehow related, there are two apparently
different notions of refinement types out there. The first, due to Freeman
and Pfenning in "Refinement Types for ML", extends structural typing with
the notions of intersection and union types (here put in use). The second,
due to Flanagan in "Hybrid Type Checking", refines structural typing by
means of decidable predicates.</div>

The authors develop a *"[...] synchronized sequent calculus with
intersection, union, function and syngleton types"* as the formalization
for specifications based on input-output examples. They refine this
calculus from a more standard unsynchronized sequent calculus, itself
derived from the classic formulation of refinement types in natural
deduction. Soundness among the calculi is proved to hold.

This choice is quite natural as sequent calculus is a better fit when
dealing with proof search, since it provides some handy properties such as
the *subformula property* (there is no *cut rule* in sequent calculus),
*invertible rules* and *focusing*.

This is indeed a smart move as a well-established foundational theory and
semantics is known to entail good properties in the originated
framework. This type-theoretic interpretation pays off as they *"[...]
develop a prototype implementation that extends the core calculus with a
variety of additional feature"* (*i.e.* base types, algebraic datatypes,
parametric polymorphism, etc.). They even integrate to some extent
libraries into the synthesis process.

The authors show that their synthesizer *"[...] is competitive with
state-of-the-art example-directed systems"*, while *"[...] able to condense
the specifications, commonly seeing decreases of 10-20% compared to
previous work.".* When polymorphism is possible, they achieve *"even more
dramatic specification reductions sometimes nearing 75%."*.

This is mainly due to the support of base types (like `nat`) as valid
refinements, union types and parametric polymorphism. The first permits to
describe generic datatype values in a concise way like, for instance,
non-empty lists as `Cons(nat x list)`. The second, unions, permits to
combine refinements into larger types, eliminating some redundancy. Using
disjunctions, the first two cases of the `decrement` function specification

              0 -> 0 && 1 -> 0 && 2 -> 1

can be narrowed down to `(0 || 1) -> 0`. This is true even for inductively
defined datatypes, such as lists, when refinements share
constructors. *"For instance, lists `[0,1]` and `[0]` can be combined into
`Cons(0 x (Nil || Cons(1 x Nil)))`."*. The latter, parametric polymorphism,
helps in describing terse specifications and eases synthesis because of the
"free theorems" that polymorphic types imply.

Negation is not part of the refinement types language here. They are
encoded with De-Morgan's laws though, as negations are important when
refining specifications after a first synthesis failure. Indeed, consider
once again the above specification for the `length` function. That can be
seen as the specification for the `head` function as well (as *GB* has
explained sometime during my presentation). Authors *"[...] offer a similar
capability"* to counterexample-guided inductive synthesis (CEGIS) *"in the
form of negation, which makes it possible to articulate that a program is
not an inhabitant of a particular refinement."*. (This is correct to some
extent since real CEGIS is automatic.) Having negation, the original
refinement for `length` can be constrained by adding `[0] -> not(0)`.

The last part of the paper is dedicated to the prototype implementation. In
particular, when the context of a sequent allows different choices in how
to proceed the proof/program construction, *"[...] making the correct
decision in a performance-friendly manner proved to be the most difficult
aspect of designing our prototype."*. The authors detail different
approaches with respective performances.

One may expect that an "intellingent", refinement-guided proof search would
perform the best. The authors show that this is not the case. Quite the
opposite. They ended up implementing a "classic" type-based
enumeration. This means enumerating all possible values of a given
refinement type, followed by some filtering.

Why this phenomenon? *"[...] part of the answer is that some spaces"*, even
if the largest possible as that given by typing only, *"are easier to
traverse than others. The remainder is that the irregular shapes of the
refinement-constrained search spaces make their results harder to
cache."*. Indeed, the authors explain that optimizations are of course
necessary, such as caching results of term-enumeration, evaluation and
typechecking along with deep hash-consing of refinements, expressions and
types.

There are some limitations. The authors point out that:

- *"[...] the system does not automatically genereate helper functions --
  auxiliary recursive functions that take additional or different
  arguments."*,
- synthesis of recursive functions requires *trace complete* refinements,
  *i.e.* refinements should also subsume specifications that clarify the
  results of structurally recursive calls,
- the system *"[...] does not infer instantiation of polymorphic library
  functions"*,
- *"the theoretical framework restricts the combination of union and
  intersection types"*.
