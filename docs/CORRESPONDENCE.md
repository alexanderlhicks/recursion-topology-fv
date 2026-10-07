# CORRESPONDENCE — model boundaries and Lean ↔ paper

There are two separate correspondence questions: **does the model describe the
chosen deployed zkVM?** and **does the Lean formalization match the paper?**
The ledger below answers the first by recording the paper's abstractions and
Lean's additional changes. The later fidelity matrix addresses the second.
Proving a paper theorem in Lean does not establish the missing correspondence
to concrete instructions, cryptographic algorithms, or deployed verifier code.

For the exact conditional theorem and its proof chain, see
[PROOF_MAP.md](PROOF_MAP.md). For refinement options and limits of the perfect
premises, see [SCOPE.md](SCOPE.md).

## Model-to-reality ledger

A zero-knowledge virtual machine (zkVM) uses proofs to certify machine
executions. Its instruction-set architecture (ISA) specifies those executions.
A succinct non-interactive argument of knowledge (SNARK) supplies short proofs
with a computational extraction guarantee. The paper describes a representative
architecture for such a system; the Lean theorem concerns the mathematical
relations defined in this repository. Neither is a specification of a pinned
implementation.

The **paper column describes the bundled PDF dated 8 September 2026**. Its
page/section links are evidence for the comparisons, not required background:
the differences and their consequences are stated here.

| Area | Paper abstraction or idealization | Lean treatment | Missing obligation for a deployed claim |
|---|---|---|---|
| Instruction semantics | RV32IM plus precompiles is the intended machine, but operation predicates are treated as black boxes; instruction internals are not supplied ([Ch. 1](vanillaVM.pdf#page=2), [Ch. 3](vanillaVM.pdf#page=6)) | [`ISA.System`](../recursion-topology-fv/VMs/ISA.lean) has five operation classes with supplied predicates and interpretation maps. Code selection and memory effects are checked, but the predicates are not identified with decoded RISC-V instructions | Prove decoding, opcode coverage, finite-word arithmetic, and refinement of all supported operations to the selected executable ISA specification, such as Sail |
| State representation | Describes 32-bit PC/register values, a finite register file and byte-addressed memory ([Ch. 1](vanillaVM.pdf#page=2)) | [`Word := ℕ`](../recursion-topology-fv/VMs/State.lean), registers `ℕ → Word`, and memory `Index → Value` for supplied types; these types alone enforce neither 32-bit bounds nor the architectural register/byte domains | Supply concrete encodings and state invariants; prove that every modeled transition preserves them and corresponds to the intended machine state |
| Deferred operations and bus | Uses a shared bus and chip predicates; asks implementers to specify the lookup mechanism ([Ch. 3](vanillaVM.pdf#page=5)) | [`Bus`](../recursion-topology-fv/VMs/Bus.lean) uses three lists, membership and universal chip validity. Injectivity equates the four inner witnesses' buses within a segment. Extra valid entries are permitted | Connect actual interaction messages, multiplicities and lookup constraints to these relations; verify hash/range operations and any global program/memory interaction semantics |
| Extraction and recursive security | Postulates one efficient straight-line extractor from statement/proof alone, with negligible error and additive runtime overhead. Remark 2 explicitly idealizes its composition across recursion and warns against reading the bounds as a concrete security level ([§5.1](vanillaVM.pdf#page=13)) | [`KnowledgeSound`](../recursion-topology-fv/Preliminaries/ArgumentSystem.lean) instead requires one unrestricted total witness function correct for **every** accepted pair. There are no adversaries, probabilities, oracle model or runtime bounds. Eight such properties are premises of the assembly | Establish a construction-specific extraction experiment, including oracle access and auxiliary inputs, and prove an applicable composition theorem. Ordinary efficient knowledge soundness does not automatically instantiate the perfect predicate |
| Memory and bus commitments | Defines probabilistic position/update binding and bus collision resistance against efficient adversaries. Suggests Merkle trees but leaves their binding proof out; commitment keys are omitted ([§5.1](vanillaVM.pdf#page=14)) | [`PositionBinding`, `UpdateBinding`, `Complete`](../recursion-topology-fv/Preliminaries/VectorCommitment.lean) are universal error-free predicates; [`CollisionResistant`](../recursion-topology-fv/Preliminaries/HashCommitment.lean) is global injectivity. No concrete compressing scheme, keyed setup or binding reduction is supplied | Specify algorithms, encodings, parameters and any keys; prove computational binding/collision bounds and propagate them through composition. Perfect injectivity on a larger domain than a finite digest space is impossible; replacing these premises requires a different security interface |
| Circuit and verifier correspondence | Describes the inner/segment/convert/combine/embed relations and verifier hierarchy schematically ([Ch. 4](vanillaVM.pdf#page=9)); does not prove correspondence to deployed code | Relations and supplied verifiers are Lean predicates. [`MultiStep`](../recursion-topology-fv/VMs/MultiStep/MultiStep.lean) proves their qualitative composition; [`VanillaVM`](../recursion-topology-fv/VMs/VanillaVM/VanillaVM.lean) derives its leaf guarantee from the bus proof. No constraint extraction or executable-verifier bridge is included | Prove that the actual constraints implement each relation and that recursive/final verifier code implements the modeled checks, including field encodings, transcripts and statement/key binding |
| Lengths and public endpoints | Fixes code and duration `T`, requires equal padded segment lengths, `n ∣ T`, `T ≥ 2n` and `T ≤ poly(λ)`. CTE statements contain full start/end memory; the resulting boundary verifier is expressly not succinct ([Ch. 2](vanillaVM.pdf#page=4), [Defs. 9–11](vanillaVM.pdf#page=15)) | Retains positive fixed segment length, divisibility, minimum duration and full-state endpoints. There is no security parameter or polynomial-duration condition; padding, boot/halt behavior and a serialized public receipt are not implemented | Prove the actual length/padding conventions, boot and termination rules, public input/output encoding, and program/key identity. Relate a practical receipt to the full-state statement |
| Quantitative security | Theorem 27 accounts for component extraction, bus collisions, per-step memory binding and a **separate terminal-memory binding** event; runtime bookkeeping remains subject to the extraction idealization ([§5.4](vanillaVM.pdf#page=29)) | [`cte_main`](../recursion-topology-fv/VMs/VanillaVM/VanillaVM.lean) proves a perfect conditional trace theorem, with no advantage inequality or runtime theorem. The digest records paper bounds; recording them does not mechanize them | Formalize the security games, reductions, total error and runtime bounds for the actual construction and parameters before quoting a security level |
| Completeness and privacy | Explicitly leaves completeness definitions to implementers (§5.1); CTE is an extraction/soundness property, not a privacy theorem | No prover, end-to-end honest-proving completeness or zero-knowledge simulation theorem. Commitment `Complete` only checks honest openings; accepting sanity instances only witness satisfiable hypotheses | Show that every allowed valid execution has correctly generated witnesses and accepted final proofs. Prove any claimed privacy property separately |
| Application claim | Motivates the architecture using an Ethereum state-transition program and initial/final state ([Ch. 1](vanillaVM.pdf#page=2)); the security relation is parameterized by operation predicates and fixed code | The assembly proves a trace for the **configured ISA** between supplied full states. It does not verify a compiler, guest application or Ethereum transition function | Connect program compilation/loading, permitted inputs or hints, and public outputs to the application's specification |

Two examples clarify the distinction. A proof that all extracted hash calls
satisfy a supplied predicate does not establish Keccak correctness until that
predicate is connected to the intended permutation and its constraints. A proof
that equal bus digests imply equal buses uses **injectivity** here; a real finite
digest instead needs a computational collision argument and a failure term.

The paper already idealizes recursive extraction; Lean further changes the
security interface by dropping its error and resource accounting. The Lean
interface is stronger about correctness of every accepted pair, while imposing
fewer computational requirements on the witness function. It is therefore not
simply the paper's computational theorem with its bounds set to zero.
For inhabited witness types, classical choice equates the current extraction
predicate with existence of a valid witness for each accepted pair; this alone
does not produce an executable extractor. See [IDEALIZATION.md](../IDEALIZATION.md).

### Computability of extraction

The paper's Definition 3 **requires a probabilistic polynomial-time extractor**:
an algorithm, not just a witness-selection function. In its intended
random-oracle model, the algorithm's permitted oracle access must also be
specified. The paper's extraction and composition assumptions are idealized;
writing down that requirement does not construct such an extractor for a
deployed proof system.

The Lean result does not establish that computational requirement. There are
three separate gaps:

1. **The primitive interface does not require an algorithm.**
   [`Extractor.extract`](../recursion-topology-fv/Preliminaries/ArgumentSystem.lean)
   is a function between arbitrary types. `KnowledgeSound` asserts its existence
   and correctness, without finite encodings, computability or runtime premises.
   It permits computable instances but does not require them.
2. **The assembled extractor selects memory witnesses by classical choice.**
   [`MultiStep.System.cte`](../recursion-topology-fv/VMs/MultiStep/MultiStep.lean)
   uses [`chooseMemStep`](../recursion-topology-fv/VMs/Memory.lean), explicitly
   declared `noncomputable`. It tests whether *some* witness satisfies an
   arbitrary acceptance predicate and uses `h.choose` to select one. This is
   not a search algorithm. Even computable primitive extractors would not, by
   themselves, make this assembly executable. A constructive assembly should
   retain the extracted memory witnesses through recursive trace composition.
3. **Generic state reconstruction uses classical equality.**
   `stepReconstruct` and `reconstructTrace` are also `noncomputable`: a write
   tests equality in the arbitrary memory-index type, without requiring a
   supplied decision procedure. Concrete finite addresses can have decidable
   equality; that representation and a constructive reconstruction still need
   to be supplied, together with computational interfaces for the other data.

These observations do **not** prove that every concrete instance lacks a
computable extractor. A `noncomputable` declaration identifies how the present
definition was constructed; it is not a proof that no equivalent algorithm can
exist. Nor do these gaps invalidate the kernel-checked existence/correctness
theorem. They prevent reading that theorem as a verified executable extractor
or a polynomial-time knowledge guarantee. Establishing computability and
bounding runtime are distinct further obligations.

The [computability inventory](COMPUTABILITY.md) audits every source module,
including the supplied verifiers/commitments, constructive trace composition,
classical reconstruction, private-input wrapper and accepting examples. It
also records the checked axiom footprints and explains why neither `#print
axioms` nor the absence of a `noncomputable` marker certifies an arbitrary
interface function's computability.

## Paper-to-Lean fidelity matrix

The remaining sections have one row per formalized paper item. A reviewer's
sign-off is a per-row judgment that a Lean declaration faithfully and completely
formalizes the selected paper item. Read those cells together with the
model-to-reality ledger: they do not certify a concrete instantiation or imply
that probability/runtime content has been mechanized. The existing reviewer
cells are retained as the review record.

**Status vocabulary:** `stated` (statement written, proof may be `sorry`) · `proved` (proof
complete, axiom-clean) · `pending(N)` (blocked on N other rows) · `n/a` (Lean-only scaffolding, no
paper counterpart).

**Reviewer columns:** `Fidelity` = does the Lean statement mean the paper statement? ·
`Complete` = does it cover the *whole* paper statement, not a faithful fragment? (finality learned
"faithful but partial" is a distinct failure mode.) · `Reviewer` = collaborator who signed off +
date.

**Rule (I1, CI-checked):** every declaration formalizing a paper notion has a docstring citing the
paper label AND a row here; the named Lean declaration must actually elaborate (`#check`).

The automated correspondence check elaborates qualifying declaration names.
Docstring/paper fidelity, completeness of coverage, and reviewer sign-offs
remain human review obligations; the separate hygiene gate checks admissions
and axiom dependencies.

**Paper:** [`vanillaVM.pdf`](vanillaVM.pdf). In particular, `code` and `T` are
fixed system parameters in `def:cte`.

This matrix remains **paper-item keyed**. Frozen Lean infrastructure that is not itself the
selected formalization of a complete paper item (`Relation`, `Extractor`, the abstract `ZkVM`
packaging, `TraceValid`, and the `StepInterface` coordination scaffold) belongs to I4 but does not
receive an artificial row here.

## Core

**Frozen kernel (I4)** — the stable heart. **Provisional** — the commitment layer, expected to
change (see notes below the table).

| Paper label | Lean declaration | Tier | Status | Fidelity | Complete | Reviewer |
|---|---|---|---|---|---|---|
| Argument system Π=(Prove,Verify) | `VanillaZkVM.ArgumentSystem` | frozen | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| Knowledge soundness (`def:extractable`) | `VanillaZkVM.KnowledgeSound` | frozen | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| — (consistency floor for KS) | `VanillaZkVM.knowledgeSound_trivialAS` | scaffolding | proved (n/a) | n/a | n/a | Dmitry 2026-07-29 |
| VM state S=(pc,regs,mem) (ch01) | `VanillaZkVM.VMStateWith` | support | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| Committed state Ŝ (ch02) | `VanillaZkVM.CommittedVMState` | support | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| Correct-execution relation R* (`eq:relation-star`) | `VanillaZkVM.ZkVM.Rstar` | frozen | proved | — (see §) | ✗ (see ‡) | _re-review pending_ (previously Dmitry 2026-07-29) |
| Correct-trace extractability (`def:cte`) | `VanillaZkVM.ZkVM.CTE` | frozen | proved | — (see §) | ✗ (see †) | _re-review pending_ (previously Dmitry 2026-07-29) |
| CTE ⇔ KS (`rem:cte-ks`) | `VanillaZkVM.ZkVM.cte_iff_knowledgeSound` | frozen | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| Merkle memory commitment `Com_mem` (ch02) | `VanillaZkVM.VectorCommitment` | provisional | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| Commitment completeness (instruction before `def:binding`) | `VanillaZkVM.VectorCommitment.Complete` | provisional | proved | — | — | _unreviewed_ |
| Position binding (`def:binding`) | `VanillaZkVM.VectorCommitment.PositionBinding` | provisional | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| Update binding (`def:binding`) | `VanillaZkVM.VectorCommitment.UpdateBinding` | provisional | proved | — | — | _unreviewed_ |
| Bus commitment `Com_bus` (`def:bus-cr`) | `VanillaZkVM.HashCommitment` | provisional | proved | ✓ | ✓ | Dmitry 2026-07-29 |
| Collision resistance (`Adv^cr`) | `VanillaZkVM.CollisionResistant` | provisional | proved | ✓ | ✓ | Dmitry 2026-07-29 |

> **§ Private inputs (2026-09-22) — `ZkVM`, `TraceValid`, `Rstar`, and `CTE` generalize the
> paper.** The abstract `ZkVM` carries a private-input type `PrivInput`, both boundary
> projections take a private input, `TraceValid` is stated under one, the `R*` witness is a
> pair (private input, trace), and the `CTE` extractor returns such a pair. The paper's
> `def:cte` and `eq:relation-star` are deterministic; they are recovered exactly by
> `PrivInput := Unit`, which every current VM instance uses. The two rows above were signed
> before this generalization, so their fidelity cells are reopened: the reviewer must confirm
> that the deterministic specialization is the paper's statement and that the generalized
> reading is the intended one. Every downstream instance row is unaffected in meaning.

> **‡ `Rstar` is an abstract relation skeleton, not yet the whole concrete `R*`.** Its witness is a
> trace satisfying the selected `ZkVM.step`, but the current abstract
> declaration neither requires statements to contain full-memory boundary states nor constrains
> the verifier to commit those states as specified immediately after `eq:relation-star`. The assembly
> supplies that concrete boundary/verifier package in the full Vanilla VM instance. Fidelity to the
> trace-validity component is signed; completeness is not.

> **† `CTE` completeness is deliberately left unsigned.** `ZkVM.CTE` is *faithful* to
> `def:cte` (fidelity ✓), but it is not signed off as covering the paper definition in full: the
> paper's `def:cte` is **too verbose** and should probably itself be **refactored**. Until that
> refactor happens, claiming completeness against the current paper text would be signing off on a
> statement we expect to restate. Re-open this row once `def:cte` is tightened.

> **Note (I4).** The paper states position binding and update binding as
> **independent** properties. Update binding supplies the commitment-realizability
> guarantee needed after a write; the punctured non-equivocation condition did
> not. The entire `VectorCommitment` binding layer and `CollisionResistant` remain
> *provisional* and may gain keyed/algorithmic variants.

## Memory extractability

These declarations formalize the memory component only. `MemFreePredicate`
abstracts the PC/register transition and cannot itself connect a separate
`MemStep` value to registers.

For these rows, human review must separately check (a) that the read/write
opening and memory equations match the paper's memory slice, (b) that
`trace_mem_extract` constructs each next represented state without assuming
that the second committed state already represents a full state, and (c) that
no row is marked complete merely because
the later ISA/bus conjuncts live in another module. This distinguishes a
faithful memory component from the complete committed `φ̂_step`.

| Paper label | Lean declaration | Status | Fidelity | Complete | Reviewer |
|---|---|---|---|---|---|
| full/committed memory invariant (`rem:mem-inheritance`) | `VanillaZkVM.CommitInv` | proved | — | — | _unreviewed_ |
| committed memory read/write (`eq:op-mem-comm-read`, `eq:op-mem-comm-write`) | `VanillaZkVM.CommittedMemory.read` / `CommittedMemory.write` | proved | — | — | _unreviewed_ |
| full-memory read/write (`eq:mem-op-read`, `eq:mem-op-write`) | `VanillaZkVM.FullMemory.read` / `FullMemory.write` | proved | — | — | _unreviewed_ |
| committed/full memory step (memory component of `φ̂_step`/`φ_step`) | `VanillaZkVM.CommittedMemory.step` / `FullMemory.step` | proved | — | — | _unreviewed_ |
| memory-inheritance step constructing the next full state (`rem:mem-inheritance`, `thm:main` Step 6) | `VanillaZkVM.trace_mem_extract` | proved | — | — | _unreviewed_ |
| trace reconstruction invariant (`rem:mem-inheritance`) | `VanillaZkVM.trace_mem_extract` | proved | — | — | _unreviewed_ |
| — (joint satisfiability model, I6) | `VanillaZkVM.MemorySanity.exactVC_bindingAssumptions` | proved (n/a) | — | — | _unreviewed_ |
| — (punctured-condition countermodel, I6) | `VanillaZkVM.MemorySanity.appendBitVC_not_updateBinding` | proved (n/a) | — | — | _unreviewed_ |

## Representative ISA

The paper's full opcode taxonomy is deliberately reduced to five representative
operation classes. `code` records only the class at each program counter, while
the PC/register predicates may still specify the exact fixed instruction there.
The class check and every memory equation are explicit. This issue therefore
specifies the five-way step structure, not an opcode decoder or an RV32IM
correctness proof. The `ISASanity` module gives private accepted and rejected
examples and checks that `stepPlain` can be used directly as `ZkVM.step`.
The two-step VM now performs that assignment publicly and requires each
committed `MemStep` to agree with the selected operation before reconstruction.
The read clause also requires unchanged memory, as stated by `eq:mem-op-read`
and the ch03 prose.

Human review must check more than elaboration: verify that the five classes are
an explicit representative simplification; that every `operation` contains the
fetch equation; that read/write address and value fields come from the stated
registers; that `committedOperation` rejects a mismatched `MemStep`; and that
`TwoStep.System.toZkVM.step` is exactly `stepPlain`. The reviewer should also
check that `indexOfWord` and `valueOfWord` merely convert machine words to the
memory's address and value types; the actual instruction requirements must
remain in `memFreePred`.

**Reviewer action:** after checking the PR head, replace the em dashes and
`_unreviewed_` entries below with the fidelity/completeness decision, reviewer
name, and date **in a separate review commit**. Do not edit these cells in the
implementation commit: the separate commit records human sign-off on a stable
code version.

| Paper label | Lean declaration | Status | Fidelity | Complete | Reviewer |
|---|---|---|---|---|---|
| operation taxonomy (ch03, deliberately simplified) | `VanillaZkVM.ISA.OperationClass` | proved | — | — | _unreviewed_ |
| fixed `code` class, selected per-operation `φ'_op`, and register-word interpretation (`eq:phiop`) | `VanillaZkVM.ISA.System` / `VanillaZkVM.ISA.System.selectedMemFreePred` | proved | — | — | _unreviewed_ |
| `φ_op`, including fetch and memory equations (`eq:phiop`, `eq:phi-read-decomp`, `eq:phi-write-decomp`) | `VanillaZkVM.ISA.System.operation` | proved | — | — | _unreviewed_ |
| disjunctive `φ_step` (`eq:step`) | `VanillaZkVM.ISA.System.stepPlain` | proved | — | — | _unreviewed_ |
| program-selected committed operation, without bus (`eq:step-bus2`) | `VanillaZkVM.ISA.System.committedOperation` / `VanillaZkVM.ISA.System.committedStep` | proved | — | — | _unreviewed_ |
| instruction selection by `code[pc]` (`eq:op`, `eq:phiop`) | `VanillaZkVM.ISA.System.stepPlain_iff_operation_at_pc` | proved | — | — | _unreviewed_ |
| committed/full operation correspondence (`prop:memory-extractability`, `thm:main` Step 6) | `VanillaZkVM.ISA.System.committedOperation_stepPlain` | proved | — | — | _unreviewed_ |

## Segment bus

Each segment has its own `SegmentBus`. Separate lists hold Keccak calls,
Poseidon calls, and range-check inputs. The step proof checks reads, writes, and
ordinary arithmetic directly. For hash and range operations, it instead adds
the required entry to the bus; the three chip proofs check every entry in their
respective lists. The explicit `MemStep` is also kept for later memory
reconstruction. Bus entries contain only program counters and registers because
the hash and range checks inspect only those fields; `stepBus` separately
requires the memory commitment to remain unchanged. This is the
five-class version of the paper's bus. Each hash-list predicate also checks
whether the fixed program assigns that call to Keccak or Poseidon. Lean uses
lists, making order and duplicate entries part of the committed bus value; the
paper leaves the concrete collection representation unspecified. Required
entries must be present and every stored entry must pass its chip check, but an
additional valid entry or duplicate is allowed. Exact RV32IM and chip
implementations remain outside this issue.

The reusable declarations in `VMs/Bus.lean` depend only on the commitment
scheme, segment length, representative ISA, and segment/inner verifiers. The
non-recursive execution and CTE demonstration is deliberately isolated in
`VMs/TwoStep/WithBus.lean`; the assembly consumes the same one-segment
system from the recursive VM without importing the two-layer connection module.

Human review must verify four points independently. First,
`stepWithBus_committedOperation` must derive the existing
`ISA.System.committedOperation` for the same `MemStep`, not an unrelated
execution predicate or an assumed refinement theorem; the concrete
`TwoStepSystem.busBridge` must use that same `MemStep` to prove the weaker
interface statement that a suitable memory witness exists. Second, collision
resistance may identify only the four buses extracted under one segment's
public commitment; it must not imply that different segments have equal buses. Third,
`execution_extract` must retain each segment's bus and `MemStep` values and use
the shared `chain_flatten` theorem to join the state traces. Fourth, the reviewer
must approve the concrete choice of lists for the paper's bus collections.

**Reviewer action:** after checking the PR head, replace the em dashes and
`_unreviewed_` cells below in a separate review commit.

| Paper label | Lean declaration | Status | Fidelity | Complete | Reviewer |
|---|---|---|---|---|---|
| segment bus collection (ch02/ch03) | `VanillaZkVM.Bus.SegmentBus` | proved | ✓ | ✓ | Dmitry |
| committed step with hash/range checks recorded in the bus (`eq:step-expanded`, `eq:step-bus2`) | `VanillaZkVM.Bus.System.stepBus` / `VanillaZkVM.Bus.System.stepWithBus` | proved | ✓ | ✓ | Dmitry |
| bus-to-committed-step implication (`eq:step-bus2`) | `VanillaZkVM.Bus.System.stepWithBus_committedOperation` / `VanillaZkVM.Bus.TwoStepSystem.busBridge` | proved | ✓ | ✓ | Dmitry |
| `R_{0,step}` (`eq:rel-inner-step`) | `VanillaZkVM.Bus.System.RInnerStep` | proved | ✓ | ✓ | Dmitry |
| `R_{0,keccak}`, `R_{0,poseidon}`, `R_{0,range}` (ch04) | `VanillaZkVM.Bus.System.RInnerKeccak` / `VanillaZkVM.Bus.System.RInnerPoseidon` / `VanillaZkVM.Bus.System.RInnerRange` | proved | ✓ | ✓ | Dmitry |
| `R_1` segment relation (ch04) | `VanillaZkVM.Bus.System.RSegment` | proved | ✓ | ✓ | Dmitry |
| segment extraction and agreement of its four recovered buses (`lem:segment`) | `VanillaZkVM.Bus.System.segment_extract` | proved | ✓ | ✓ | Dmitry |
| per-segment extraction and concatenation (`thm:main` Steps 4–5) | `VanillaZkVM.Bus.TwoStepSystem.execution_extract` | proved | ✓ | ✓ | Dmitry |
| two-step VM with segment buses and full-memory CTE (`def:cte`, `thm:main` Steps 4–6, non-recursive specialization) | `VanillaZkVM.Bus.TwoStepSystem.toZkVM` / `VanillaZkVM.Bus.TwoStepSystem.cte` | proved | ✓ | ✓ | Dmitry |

## Two-step toy (intermediate)

| Paper label | Lean declaration | Status | Fidelity | Complete | Reviewer |
|---|---|---|---|---|---|
| segment/final toy relations | `TwoStep.System.RSeg` / `RFinal` | proved | — | — | _unreviewed_ |
| representative `φ_step` used as the concrete toy VM step (`eq:step`) | `VanillaZkVM.TwoStep.System.toZkVM` | proved | — | — | _unreviewed_ |
| — (two-layer committed-chain extraction, intermediate) | `VanillaZkVM.TwoStep.System.committedTrace_extract` | proved (n/a) | n/a | n/a | _unreviewed_ |
| toy CTE over full memory (`def:cte`, `prop:memory-extractability`) | `VanillaZkVM.TwoStep.System.cte` | proved | — | — | _unreviewed_ |

## Multi-step recursion tower

| Paper label | Lean declaration | Status | Fidelity | Complete | Reviewer |
|---|---|---|---|---|---|
| `R_2` convert (`lem:convert`) | `MultiStep.System.RConvert` | proved | — | — | _unreviewed_ |
| `R_3` combine + tree unrolling (`lem:combine`) | `MultiStep.System.RCombine` / `combine_tree` | proved | — | — | _unreviewed_ |
| — (explicit tree-unrolling extraction procedure, `rem:wellfounded`) | `MultiStep.System.buildTrace` | proved (n/a) | n/a | n/a | _unreviewed_ |
| `R_4` embed (`lem:embed`) | `MultiStep.System.REmbed` | proved | — | — | _unreviewed_ |
| — (committed-trace extraction via tree unrolling) | `MultiStep.System.committedTrace_extract` | proved (n/a) | n/a | n/a | _unreviewed_ |
| multi-step CTE over full memory (`def:cte`, `prop:memory-extractability`) | `MultiStep.System.cte` | proved | — | — | _unreviewed_ |
| — (joint satisfiability model, I6) | `VMs/MultiStep/MultiStepSanity.lean` (all declarations private — no public name to `#check`) | n/a | — | — | _unreviewed_ |

## Full Vanilla VM assembly

The assembled system uses `Bus.System.segment_extract` to prove that an
accepted base segment proof yields the valid committed trace required by the
recursive `MultiStep` theorem. It does not assume this fact independently. Its
verifier commits the initial and final full-memory states before invoking the
embed verifier, and its only plain execution predicate remains
`ISA.System.stepPlain`.

The private model in `VMs/VanillaVM/VanillaVMSanity.lean` witnesses that the
complete assumption structure and an accepted proof can occur together. Human
review must confirm that this structure lists every probability-free
assumption used by the paper and that the private connection to the recursive
system preserves the `MemStep` recovered by the bus proof. This theorem
deliberately omits the paper's numerical advantage bound.

| Paper label | Lean declaration | Status | Fidelity | Complete | Reviewer |
|---|---|---|---|---|---|
| full zkVM with committed memory boundaries (`def:zkvm`, paragraph after `eq:relation-star`) | `VanillaZkVM.VanillaVM.System` / `VanillaZkVM.VanillaVM.System.toMultiStep` / `VanillaZkVM.VanillaVM.System.toZkVM` | proved | — | — | _unreviewed_ |
| main CTE theorem, perfect-model form (`thm:main`) | `VanillaZkVM.VanillaVM.System.Assumptions` / `VanillaZkVM.VanillaVM.System.cte_main` | proved | — | ✗ (advantage bound) | _unreviewed_ |
