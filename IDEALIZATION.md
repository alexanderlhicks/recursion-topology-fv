# Notes on Idealization

In this repository, we (over-)idealize many cryptographic concepts. We explain below in detail.
We are of course aware that this is far from being cryptographically accurate, and we may change this in the future.

For the current theorem, its explicit premises, and the proof steps that use them,
see the [proof map](docs/PROOF_MAP.md). The [scope ledger](docs/SCOPE.md) also
records the ISA, bus, completeness, and implementation abstractions; those are
separate from the cryptographic idealizations below.

### Perfect model without probabilities and running times

*Summary*. We do not work with probabilities and running times, and instead assume perfect, error-free idealizations.

*Example Code*. Recall that a hash function family is collision-resistant if it is hard to find two distinct inputs mapping to the same output. Currently, we idealize this as follows:
```lean
def CollisionResistant (H : HashCommitment) : Prop :=
  ∀ b b' : H.Domain, H.hash b = H.hash b' → b = b'
```
Note that this essentially means we require injectivity. Of course, compressing hash functions cannot be injective (but we never explicitly require it to be compressing in the lean code). A more proper definition would give an efficient adversary a hash key and then ask it to find a collision.

*Rationale*. We may change this in the future and add probabilities, explicit adversaries, and running times, but for now the rationale is as follows: main goal of this lean project is to formalize that the recursive topology -- with all of its relations -- is structurally proving the right thing, i.e., the relations compose to ultimately yield correctness of the computation. We decided that to get quickly towards that goal -- without distraction by probabilities and running times -- such an idealization is a good first step.


### No use of security parameter, negligible functions, polynomial time

*Summary*. The code does not use or define an asymptotic security parameter, negligible functions, or polynomial time.

*Rationale*. Clearly, as we do not talk about running times or success probabilities at all, using negligible functions or polynomial time is not necessary. Also, ultimately the systems we deploy in practice will not be asymptotic objects but rather concrete objects with concrete security levels. Thus, even when we introduce proper formalism for probabilities and running times, we would most likely not follow an asymptotic treatment. The only argument for it may be to meaningfully talk about "constant depth" vs "logarithmic depth" vs "polynomial depth" of recursion.

### Notions of knowledge soundness

*Summary*. The current notion is perfect extraction by an unrestricted mathematical
function. It does not certify an efficient knowledge extractor for a succinct argument.

*Example Code.* Our current definition of knowledge soundness looks like this:
```lean
def KnowledgeSound {R : Relation} (AS : ArgumentSystem R) : Prop :=
  ∃ E : Extractor R AS, ∀ (x : R.Stmt) (p : AS.Proof),
    AS.verify x p → R.rel x (E.extract x p)
```
There is one witness function correct for every accepted statement-proof pair.
Neither its computability nor a running-time bound is required, and accepted
proofs are not restricted to those produced by efficient adversaries. For an
inhabited witness type, classical choice makes this property equivalent to
`∀ x p, AS.verify x p → ∃ w, R.rel x w`. It must not be read as a
resource-bounded knowledge claim.

Impossibility claims about efficient universal extraction require those
additional computational conditions; they are not consequences of this Lean
type alone. Nothing in the type requires succinctness, either. The accepting
sanity models establish consistency of the abstract assumptions, not their
realizability by a deployed succinct proof system.

Computability is **unspecified**, not required and not refuted by this definition.
The [computability inventory](docs/COMPUTABILITY.md) distinguishes supplied
functions, constructive composition and explicit classical dependencies in the
assembled extractor. A `noncomputable` definition is not a proof that no
equivalent computable implementation can exist for a concrete instance.

*Rationale.* The same rationale as for the perfect model above.

*Other notions for the future.*
This is most likely the most important thing to change in the future. Assuming, we use probabilities, adversaries, and running times at some point, here are some options:
- Non-universal extractor (i.e., for every adversary, there is one extractor), and this extractor gets the adversary's coins. This notion seems plausibly achievable but is often insufficient in applications: adversaries in practice do not pick their statements and forged proofs just based on random coins, but also based on inputs that they observe from the environment, e.g., public keys of other users.
- Non-universal extractor, extractor gets adversary's coins, and both adversary and extractor get a joint auxiliary input. The quantifiers are: for every adversary, there is one extractor, so that for every auxiliary input, ... . This models the side information obtained from the environment, but unfortunately there are impossibilities with respect to that model (see [this paper](https://eprint.iacr.org/2014/402)) unless we restrict the auxiliary input in some way. How to restrict it in a meaningful way is still under active discussion.

For constructions whose component extraction theorems are in the random-oracle
model, a separate argument is needed to justify a recursive extraction
interface. The current development supplies no such bridge. General
relativized succinct arguments in the ROM are
[ruled out](https://eprint.iacr.org/2024/728); this does not rule out every
recursive construction or a formalization of a suitably restricted model.
Any proposed bridge must state the construction, extractor/oracle access,
auxiliary inputs, conditioning, errors, and running times explicitly.
