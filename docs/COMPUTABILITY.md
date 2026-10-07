# Computability and execution boundaries

**The current theorem proves existence and correctness of mathematical witness
functions. It does not establish an executable end-to-end extractor.**
Some composition functions are constructive relative to their inputs, three
memory definitions are explicitly `noncomputable`, and the abstract interfaces
leave the computability of their supplied functions unspecified. These are
different statuses; none should be inferred from the word “extractor” alone.

The [proof map](PROOF_MAP.md) explains what those functions accomplish logically.
The [model-to-reality ledger](CORRESPONDENCE.md#model-to-reality-ledger) separately
compares the paper and Lean with a concrete zkVM.

## What the classifications mean

- **Computability unspecified:** a field has a function or predicate type, with
  no computational requirement. It can have computable instances; its type
  does not imply that all instances are computable.
- **Constructive relative to inputs:** the definition uses explicit projections,
  pattern matching, ordinary natural-number tests or terminating recursion. It
  can compute when its data and called functions have computational
  implementations. This source observation is not a formal complexity theorem
  or a proof of computability for arbitrary supplied functions.
- **Explicitly `noncomputable`:** the present data definition relies on classical
  operations for which Lean does not generate runtime code. This does not prove
  that an equivalent computable implementation is impossible for a concrete
  instance.
- **Logical specification:** a proposition or theorem describes correctness. It
  is not, by itself, a Boolean checker, a witness generator or an algorithm.

Computability means an effective procedure on a specified representation;
polynomial-time efficiency additionally needs a cost and input-size model.
Computability relative to an explicitly permitted oracle is a further distinct
claim. None of these requirements is encoded by the current `KnowledgeSound`
or CTE types. The paper's Definition 3, in contrast, requires a probabilistic
polynomial-time extractor under its idealized security interface.

A Lean function type alone does not impose computability. Likewise, `P : Prop`
does not provide an executable decision procedure for `P`. An effective checker
needs a computational decision procedure, or an executable Boolean function
proved equivalent to the predicate. A classical `Decidable P` instance does not
supply an effective procedure. Proofs are erased from compiled code; extracting
data from an existential proof by classical choice is not a runtime search.
See the official Lean references on
[propositions](https://lean-lang.org/doc/reference/latest/The-Type-System/Propositions/)
and [axioms and compilation](https://lean-lang.org/doc/reference/latest/Axioms/).
These references explain the language; this audit uses the repository's pinned
[`lean-toolchain`](../lean-toolchain).

## Inventory of the primary interfaces and composition

The following covers every non-sanity source module. Rows grouping predicates
include their relation/argument-system packaging; that packaging does not add
an algorithm. `Word := ℕ` is an effective natural-number type, but its presence
does not make arbitrary functions over words effective.

| Component / declarations | Computational status | Exact boundary |
|---|---|---|
| [`Relation.rel`, `ArgumentSystem.verify`](../recursion-topology-fv/Preliminaries/ArgumentSystem.lean) | Computability unspecified; logical predicates | Arbitrary `Prop` fields, with no decision procedure, finite encoding or executable verifier correspondence |
| `Extractor.extract`; `KnowledgeSound` | Function computability unspecified; property is logical | `∃ E, ∀ x p, verify x p → rel x (E.extract x p)` neither constructs an algorithm nor requires one. For inhabited witnesses, classical choice equates it with existence of a witness for each accepted pair |
| [`VectorCommitment.commit`, `openProof`, `verify`](../recursion-topology-fv/Preliminaries/VectorCommitment.lean) | Function/predicate computability unspecified | Arbitrary types and supplied maps; no encoding, effective commitment/opening implementation or executable verification interface |
| `Complete`, `PositionBinding`, `UpdateBinding` | Logical specifications | Correctness/binding propositions, not generators, decision algorithms or resource bounds |
| [`HashCommitment.hash`, `CollisionResistant`](../recursion-topology-fv/Preliminaries/HashCommitment.lean) | Hash computability unspecified; injectivity is logical | The function is supplied, not implemented. Injectivity proves uniqueness, not an effective inverse or an effective collision search |
| [`ZkVM.initial`, `terminal`, `step`, `verify`; `TraceValid`](../recursion-topology-fv/Specification/Zkvm.lean) | Boundary-map computability unspecified; predicates logical | Abstract states/proofs/private inputs and no computational decision procedures |
| [`Rstar`, `ASstar`, `CTE`, `cte_iff_knowledgeSound`](../recursion-topology-fv/Specification/Cte.lean) | Logical specifications and repackaging | CTE asserts a total mathematical function yielding private input and trace; the equivalence theorem adds no computational guarantee |
| [`VMStateWith`, `FullVMState`, `MemStep`, `CommitInv`](../recursion-topology-fv/VMs/Memory.lean), [`Word`, register fields](../recursion-topology-fv/VMs/State.lean) | Data representations; invariant logical | Registers and memories are total functions, not supplied finite serialized objects. Some values can be computable functions; the types do not require this |
| [`CommittedMemory`, `FullMemory` predicates and memory lemmas](../recursion-topology-fv/VMs/Memory.lean) | Logical specifications/proofs | Checking a supplied transition and proving preservation do not generate its witness. Pointwise write specifications avoid needing executable address equality |
| `stepReconstruct`, `reconstructTrace`, `chooseMemStep` | Explicitly `noncomputable` | Classical equality for writes; classical witness selection. Detailed below |
| [`ISA.System.code`, `indexOfWord`, `valueOfWord`, `memFreePred`; ISA relations](../recursion-topology-fv/VMs/ISA.lean) | Supplied-map computability unspecified; relations logical | `OperationClass` has constructive `DecidableEq`, but that does not decide the arbitrary instruction predicate, functional-state equality or existence of a memory witness |
| [`StepInterface`, `StepInterface.BusBridge`](../recursion-topology-fv/VMs/Step.lean) | Interfaces and logical implications | A bridge proves one predicate implies another; it does not implement an executable step checker |
| [`BusState.ofState`, `HashCall.ofStates`](../recursion-topology-fv/VMs/Bus.lean) | Constructive projections/record construction | Copy the supplied PC/register data; do not decide equality of register functions |
| `Bus.System.busHash`, `hashChipAt`, chip predicates, verifiers and bus relations | Supplied-map computability unspecified; predicates logical | Finite lists do not make equality or chip predicates decidable when entries contain arbitrary register functions. No actual hashing or lookup checker is supplied |
| `Bus.System.segment_extract` | Constructive witness composition **relative to supplied extractors** | The exhibited function applies segment extraction then inner-step extraction, retaining `states`, `steps` and `bus`. The other three extractors and injectivity justify correctness; they do not implement a bus-equality test |
| [`concatTrace`](../recursion-topology-fv/Preliminaries/Trace.lean) | Constructive relative to supplied state/segment functions | Recursion on segment count and decidable natural-number arithmetic. Returns a state-query function, not a materialized finite trace |
| [`MultiStep.toCommitted`, `System.m`, `toZkVM`, recursive relations](../recursion-topology-fv/VMs/MultiStep/MultiStep.lean) | Projection/arithmetic and interface packaging; supplied-function dependence | Boundary conversion calls `VC.commit`; the final verifier calls supplied `embedVerify`. Packaging these calls does not establish their computability |
| `MultiStep.System.buildTrace` | Constructive relative to supplied extractors and state data | Pattern matches proof tags; tests natural-number counts; recursive measure strictly decreases. Invalid count/tag cases have explicit fallback traces. No classical witness selection is introduced here |
| `MultiStep.System.committedTrace_extract` | Constructive composition relative to component extractors | Exhibits embed extraction followed by `buildTrace`. Its logical hypotheses supply extractor existence, not executable implementations |
| `MultiStep.System.traceValid_full`, `cte` | Logical correctness plus an explicitly noncomputable exhibited reconstruction | `cte` calls both `chooseMemStep` and `reconstructTrace`; constructive tree traversal does not remove these dependencies |
| [`VanillaVM.System.toMultiStep`, `leaf_sound`, `cte_main`](../recursion-topology-fv/VMs/VanillaVM/VanillaVM.lean) | Assembly/interface packaging and logical proofs | The derived leaf extractor projects the segment extractor's states; final CTE inherits the MultiStep reconstruction and all primitive computational gaps |
| [`NonDeterministic.System.hardcode`, `cte`](../recursion-topology-fv/VMs/NonDeterministic/Wrapper.lean) | Map computability unspecified; wrapper composition constructive relative to supplied extractors/maps | Outer extraction yields private input and a base proof, then base extraction is applied. No new witness search is introduced, but the base/outer computational requirements remain unspecified |

In particular, “straight-line” in existing interface descriptions names the
intended statement/proof input shape. With no operational model for `E`, that
input shape alone does not certify an executable straight-line algorithm or
its oracle/adversary access.

The extraction theorems themselves conclude `∃ E, ...` in `Prop`. Their proofs
exhibit mathematical terms, but calling such a theorem is not a runtime API
that returns an extractor. An executable composition must receive actual
computational extractor functions, rather than merely proofs that some
functions exist. No new classical choice is needed in the explicit composition
body once those functions are supplied; that is the limited sense of
“constructive relative to inputs” used above.

## The two explicit obstacles in memory reconstruction

All **three** explicit `noncomputable` data definitions in the project sources
are in [`Memory.lean`](../recursion-topology-fv/VMs/Memory.lean):

1. Private `stepReconstruct` branches on the `MemStep` constructor. A write
   returns `fun j => if j = addr then v else S.mem j`. Since the arbitrary
   `VC.Index` has no supplied effective equality, this uses classical equality.
   `reconstructTrace` recursively calls that function and inherits the issue.
   Reads and `.other` preserve memory directly; their mathematical equations
   are simple, but the present combined definitions remain `noncomputable`.
2. `chooseMemStep` branches on `∃ w, accepts (Ŝ k) (Ŝ (k + 1)) w` for an arbitrary
   predicate, then returns `h.choose` or `.other`. No effective procedure is
   given to decide that existential, find a witness or recognize failure.
   Even decidability of `accepts` on individual candidates would not alone give
   a terminating decision/search procedure for an unbounded witness domain.

The second obstacle is introduced by the final assembly even though segment
extraction already recovers explicit `SegmentTrace.steps`. The generic
committed-trace interface passes only states plus a **proof** of an existential
`committedStep`. The full-memory extractor then selects witnesses again. A
constructive refinement can instead carry the extracted `MemStep` sequence
alongside the states through concatenation and recursion. This also preserves
opening provenance for later computational security reductions.

Finite, effectively represented addresses can supply executable equality, and
retained witnesses can remove the need to search for openings. Neither step
alone supplies computable primitive extractors, executable commitment/verifier
implementations or a polynomial-time security theorem.

## Other compositions and sanity examples

These are audit coverage notes, not additional stages of the main MultiStep
architecture. TwoStep remains an optional introductory example.

- [`TwoStep`](../recursion-topology-fv/VMs/TwoStep/TwoStep.lean) exhibits flat
  committed-trace composition using `concatTrace`; it is constructive relative
  to its supplied extractors. Its full CTE theorem uses the same
  `chooseMemStep`/`reconstructTrace` definitions as MultiStep.
- [`WithBus`](../recursion-topology-fv/VMs/TwoStep/WithBus.lean) has an explicit
  `extractedExecution` helper retaining each segment's bus and memory witnesses.
  Its final `cte` delegates to TwoStep's full-memory result and still inherits
  that result's noncomputable reconstruction. Retention in an intermediate
  result is not enough if the final interface discards it.
- [`trivialAS`](../recursion-topology-fv/Preliminaries/ArgumentSystemSanity.lean)
  has an identity witness extractor: no search is needed. Its verifier is the
  arbitrary relation predicate, so a computable extractor does not make every
  such instance's verifier computable.
- [`MemorySanity`](../recursion-topology-fv/VMs/MemorySanity.lean) supplies
  explicit Bool-memory maps, identity-style commitments and counterexamples.
  Their elementary data maps can be evaluated; their existence is not a
  deployed commitment algorithm or security theorem.
- [`StepSanity`](../recursion-topology-fv/VMs/StepSanity.lean),
  [`ISASanity`](../recursion-topology-fv/VMs/ISASanity.lean),
  [`TwoStepSanity`](../recursion-topology-fv/VMs/TwoStep/TwoStepSanity.lean),
  [`WithBusSanity`](../recursion-topology-fv/VMs/TwoStep/WithBusSanity.lean),
  [`MultiStepSanity`](../recursion-topology-fv/VMs/MultiStep/MultiStepSanity.lean),
  [`VanillaVMSanity`](../recursion-topology-fv/VMs/VanillaVM/VanillaVMSanity.lean)
  and [`WrapperSanity`](../recursion-topology-fv/VMs/NonDeterministic/WrapperSanity.lean)
  construct accepting/rejected examples and use explicit proof projections or
  simple data functions where needed. They prove satisfiable hypotheses, not
  a general computational property of all instances. Instantiating the present
  final reconstruction with finite types does not automatically replace its
  classical definitions with compiled equivalents.

Total functional representations are not inherently uncomputable: a trace
function can effectively answer each index, and only indices up to fixed `T`
are checked. However, arbitrary function values need not be effective, and
query access to a trace does not provide a finite serialized output or its
total generation cost. `buildTrace` can invoke extraction again during state
queries; memoization and cost bounds require a separate analysis.

## Evidence and limits of the audit

Source inspection covers all 25 project modules plus the import-only umbrella.
It finds three explicit `noncomputable` data definitions; this count excludes
generated declarations and is not a count of every unimplemented algorithm.
Inspection of parameters and theorem bodies supplies the distinctions above.

Kernel axiom checks on the pinned toolchain give:

| Declaration | Transitive axioms |
|---|---|
| `concatTrace`, `BusState.ofState`, `HashCall.ofStates`, `MultiStep.toCommitted` | None |
| `Bus.System.segment_extract`, `NonDeterministic.System.cte` | None |
| `MultiStep.System.buildTrace`, `committedTrace_extract` | `propext`, `Quot.sound` |
| `chooseMemStep`, `reconstructTrace`, `MultiStep.System.cte`, `VanillaVM.System.cte_main` | `propext`, `Classical.choice`, `Quot.sound` |

These footprints corroborate the reconstruction boundary; they do not certify
computability of arbitrary supplied functions. Conversely, axioms used solely
in erased proofs need not prevent otherwise executable data computations.
A compile probe returning a constructor tag from `chooseMemStep`, even for a
Bool-memory instance and the predicate `True`, is rejected with
`lean.dependsOnNoncomputable`. An ordinary `concatTrace` evaluation succeeds.
Neither probe proves that every possible equivalent extractor is uncomputable.

To recheck the source markers and selected dependency footprints:

```bash
rg -n 'noncomputable|Classical\.|\.choose' recursion-topology-fv --glob '*.lean'
```

After importing the project umbrella in a temporary Lean file, run
`#print axioms` on the fully qualified declarations in the table (all under
`VanillaZkVM`) using `lake env lean`. `lake build` and the whole-project axiom
gate check compilation/proof hygiene; neither is a formal computability or
complexity audit.

No generic verifier is proved undecidable here, and no particular concrete
extractor is proved to lack a computable replacement. Those would require
different, explicit statements. The established limits are missing
computational premises and identified classical data dependencies.

## What an executable extraction claim must add

1. Specify effective finite encodings of proofs, statements, states, openings,
   buses and private inputs, with representation/refinement invariants.
2. Supply executable primitive extractors, commitment/hash functions and
   verifiers, and prove their relation to the mathematical interfaces.
3. Preserve extracted memory witnesses through recursive composition and use
   effective address equality in a constructive reconstruction.
4. Prove termination and correctness of the resulting executable pipeline on
   accepted inputs, with specified behavior on rejected/malformed inputs.
5. Separately prove the applicable computational extraction/security experiment
   and runtime/error bounds, including any oracle access and output costs.

These are future proof obligations, not properties asserted by the current
CTE theorem. Adding an executable witness would strengthen its interpretation;
moving to error-aware computational security would also require a different
security interface and an applicable composition theorem.
