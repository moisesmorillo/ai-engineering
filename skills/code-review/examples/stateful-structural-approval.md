# Stateful Structural Approval Examples

These examples calibrate the [`Stateful structural approval gate`](../SKILL.md#stateful-structural-approval-gate). They are semantic ownership examples, not size, class, file-layout, or control-flow rules.

## A — Large stateful module is genuinely cohesive

A transaction executor is large because it implements one atomic commit protocol. Its state is a closed `TransactionPhase`; events are typed execution results; one transition function owns `phase + event + commit evidence -> next phase/action`; and small collaborators perform journal I/O, locking, and transport mechanics without deciding transitions. Retry budgets are inputs to the same transition owner. Focused table tests cover material rows, terminal states, invalid cross-products, and recovery after an uncertain commit.

The reviewer reconstructs the matrix and confirms:

- the transition function is the authoritative semantic owner;
- callers cannot bypass or independently reproduce its decisions;
- helpers own mechanics rather than competing commit policy;
- adding a phase or event produces an exhaustive compile-time/test obligation at the owner; and
- extracting portions of the transition policy would split atomic commit invariants and make correctness harder to audit.

**Expected result:** no cohesion finding. The module may remain large. Approval is supported by concrete ownership and coupling evidence—not by a broad “transaction workflow” label, proximity, or test status alone.

## B — Coordinator facade owns several independent policies

A coordinator contains bootstrap handshake and admission, inventory/reporting, positive reconciliation, coalescing/readiness, unresolved-intent recovery, mutation-effect handling, retries, durable-state transformations, and failure classification. It uses many small methods, ordinary guard clauses, dedicated constants, and passing tests.

The reviewer selects it as a structural candidate and reconstructs a conceptual matrix such as:

```text
lifecycle phase
+ input/event kind
+ retry/recovery budget
+ mutation effect certainty
+ observed reconciliation evidence
+ admission/readiness/reservation state
→ admit / transform durable state / acknowledge / retry / pause / block / recover
```

Tracing rows shows that bootstrap helpers decide admission, reconciliation helpers infer effect evidence, retry helpers reinterpret the same failure classes, and durable-state helpers choose recovery transitions. A maintainer adding one uncertain-effect recovery row must edit several unrelated helper clusters and understand implicit precedence among them. Tests mostly exercise public methods independently and happy paths; they do not map material cross-products. Calling all of this “synchronization” does not identify an authoritative policy owner.

**Expected result:** retain an actionable structural finding based on distributed policy ownership and auditability risk. Recommend semantic owners for bootstrap/admission, reconciliation/evidence, retry/effect, durable-state transition, and failure-classification policy as the repository's architecture supports. The coordinator may remain as a facade that sequences those owners. Do not propose `utils.ts`, a line limit, one helper per file, or replacing every guard with a `switch`.

Implementation-local types, constants, and helper tails strengthen the evidence only when they model those independent policies. Trivial local types and true coordinator mechanics remain colocated.

## C — Correctness bugs fixed, distributed state machine still open

### Initial review

A stateful coordinator was selected as a structural candidate. The review found incorrect acknowledgement matching and retry-budget handling, plus a structural concern: admission, evidence, retry, and recovery transitions were distributed across helper clusters without one authoritative owner.

### Corrective change

The correction fixes both functional bugs, adds regression tests, and extracts repeated event names into constants. The final coordinator still requires coordinated edits across admission, reconciliation, retry, and state-transformation helpers to add one transition. The new tests prove the two repaired cases but do not exercise the conceptual phase/event/budget/effect/evidence matrix.

### Expected corrective review

The reviewer carries the candidate forward, reconstructs the matrix from the final code, and compares policy ownership before versus after:

| Concern | Disposition | Evidence |
|---|---|---|
| Acknowledgement matching bug | `fixed` | Final behavior and focused regression test cover the original mismatch. |
| Retry-budget bug | `fixed` | The terminal budget row now produces the intended outcome and is tested. |
| Distributed state-policy ownership | `still open — partially remediated` | Two rows are correct and constants centralize spelling, but transition decisions remain split across independent helper clusters. |

**Expected result:** plain `APPROVE` is invalid while the actionable structural finding remains. If its evidenced severity is non-blocking and explicitly dispositioned under repository policy, `APPROVE WITH NOTES` may be appropriate; otherwise use `REQUEST CHANGES`. Functional closure does not imply structural closure.

## D — Facade remains after policy ownership is corrected

A corrective design retains the coordinator's public API and sequencing role, but delegates decisions to cohesive collaborators:

- a bootstrap/admission policy owns handshake, fencing, and readiness decisions;
- a reconciliation policy owns evidence interpretation and acknowledgement matching;
- an effect/retry policy owns failure classification, effect certainty, budgets, and retry/pause outcomes; and
- a durable-state transition owner applies the resulting typed decisions atomically.

The facade passes typed observations and executes returned decisions; it does not reinterpret them. It is authoritative only for sequencing and the one atomic lifecycle invariant it preserves. Each closed policy has exhaustive handling where appropriate and focused matrix tests, while facade tests verify that composition invariant. Repository search finds no competing transition implementations.

The reviewer reconstructs the end-to-end matrix and confirms that each row's semantic decision has one owner, layer boundaries are intentional, and changing one policy does not require editing unrelated facade helpers.

**Expected result:** the prior structural concern can be `fixed`, and plain `APPROVE` is allowed when no other actionable finding remains. The facade need not disappear, and the design need not place every policy in a class or replace every conditional with a table.
