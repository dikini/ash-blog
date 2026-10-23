---
title: "Ash Verification Design: Discussion Summary and Working Foundation"
description: "Working notes on Ash verification, laws, propositions, evidence, compile-time proof construction, and the logical foundations of proof assistants and SMT."
author: "Vlad"
draft: true
published: null
updated: null
topics:
  - verification
  - language-design
  - type-theory
  - proof-assistants
  - smt
---

# Ash Verification Design: Discussion Summary and Working Foundation

**Status:** Design discussion synthesis, not a language specification  
**Prepared:** 23 October 2026  
**Reference revision:** 23 October 2026; primary literature and official documentation added  
**Source:** [Laws And Properties Differences](https://chatgpt.com/c/6adaf761-3bdc-83eb-89c4-64ba115bf0f9)

## Source scope and interpretation

This document brings together the requested subjects: laws and properties; propositions, contracts, and invariants; evidence; the relationship to types and kinds; Curry–Howard alternatives; compile-time verification; ordinary sandboxed Ash proof-construction code; and independent certification.

The accessible conversation record contains five recent user messages. Its assistant replies are unresolved content references, and it provides no older turns or attachments. The original web conversation was unavailable without a separate sign-in. Consequently, this is a comprehensive working synthesis of the requested topics, **not a fully recovered transcript or a claim that every explanation below was previously agreed**.

Three status labels distinguish provenance:

- **Recorded direction:** Explicitly supported by the accessible user messages.
- **Working interpretation:** A coherent explanation or architectural reconstruction, rather than a recovered agreement.
- **Open:** A design choice not established by the accessible record.

The references added in this revision establish the intellectual background, relevant precedents, and tradeoffs. They do not retroactively establish what the missing conversation agreed. “Rocq” refers to the proof assistant formerly named Coq; historical publications retain their original titles. Historical papers are distinguished from current system documentation. Implications proposed for Ash are explicitly interpretations, rather than claims about an author's historical motivation.

All syntax is newly written, illustrative pseudocode. Keywords, APIs, inference rules, and declaration forms are not established Ash syntax. The record uses “Ascii” in the message about ordinary code; this document follows the current request's name, **Ash**, without inferring a separate language or an ASCII-only requirement.

## Contents

1. [Core direction](#1-core-direction)
2. [Laws, properties, propositions, contracts, and invariants](#2-laws-properties-propositions-contracts-and-invariants)
3. [Evidence, assumptions, and partial ordering](#3-evidence-assumptions-and-partial-ordering)
4. [Types, propositions, universes, and kinds](#4-types-propositions-universes-and-kinds)
5. [Curry–Howard and alternative architectures](#5-curryhoward-and-alternative-architectures)
6. [Static verification during compilation](#6-static-verification-during-compilation)
7. [Ordinary sandboxed Ash proof-construction code](#7-ordinary-sandboxed-ash-proof-construction-code)
8. [Independent certification and trust](#8-independent-certification-and-trust)
9. [Illustrative syntax and end-to-end examples](#9-illustrative-syntax-and-end-to-end-examples)
10. [Decisions, interpretations, and open questions](#10-decisions-interpretations-and-open-questions)
11. [Discussion paths and conceptual corrections](#11-discussion-paths-and-conceptual-corrections)
12. [Compact glossary](#12-compact-glossary)
13. [Logical foundations and the reasons behind the choices](#13-logical-foundations-and-the-reasons-behind-the-choices)
14. [Annotated primary references](#14-annotated-primary-references)

## 1. Core direction

**Recorded direction.** Ash's verification system and type system may share a logical foundation, but the design needs to distinguish their other dimensions. The discussion explicitly raises separate type and property/proposition universes, or different kinds within a uniform universe, as possible arrangements.

The operational intention is that verification happens statically during compilation. The preferred way to express proof-construction activity is appropriately sandboxed ordinary Ash code executed at compile time. Other complications were explicitly deferred. The final accessible message accepts that a foundation has been reached, but the missing assistant replies prevent recovering the exact formulation accepted.

**Working interpretation.** A useful decomposition is:

```text
statement       What is claimed?
role            Is it an algebraic law, contract, invariant, or other property?
evidence        What supports it, over which scope, under which assumptions?
construction    How is the evidence produced?
certification   How is the evidence checked against the claim?
phase           When do construction and checking occur?
policy          Which evidence is sufficient for a particular use?
```

These dimensions should not be collapsed into a single keyword or a single “verified” flag. A universally quantified algebraic law, for example, can be merely assumed, supported by tests, or formally proved. Its role as a law does not itself determine its evidence status.

## 2. Laws, properties, propositions, contracts, and invariants

**Working interpretation.** These terms distinguish logical content from the purpose a statement serves in a program.

| Term | Meaning in this synthesis | Example |
| --- | --- | --- |
| Proposition | A logical statement interpreted under a context | For every list `xs`, reversing twice returns `xs` |
| Property | A characteristic expressed as a predicate or proposition | An output list is sorted |
| Law | A property serving as an algebraic or abstraction-level obligation | An operation is associative |
| Contract | An obligation or guarantee attached to an interface or operation | A nonempty input yields an element of that input |
| Invariant | A property maintained at specified states or boundaries | A search tree maintains its ordering relation |
| Axiom or assumption | A proposition admitted without a derivation in the current system | An abstract operation satisfies associativity |
| Proof | A derivation establishing a proposition from explicit premises | A checked induction proving double reversal |
| Test evidence | Observations obtained by executing selected cases | Double reversal holds for generated lists |

### 2.1 Laws versus properties

“Property” is the broader descriptive category. A law usually states a relationship expected of an abstraction's operations, often universally quantified. Associativity, identity, and functor identity/composition are examples of this role.

A statement can be both a property and a law. The distinction is not intrinsically “testable versus provable,” “informal versus formal,” or “weak versus strong.” Those classifications concern expression and evidence, not the statement's role.

A law may be a requirement on every implementation of an interface. Each implementation then needs its own evidence, or an explicit assumption. Merely implementing operations with the right ordinary types does not generally establish their laws.

### 2.2 Propositions versus Boolean predicates

A predicate such as `is_sorted(xs)` can compute a Boolean. A proposition such as `forall xs: List<Int>. is_sorted(sort(xs))` claims something over an entire domain. Evaluating the predicate on some inputs does not establish that quantified claim.

Ash therefore needs a defined bridge between executable predicates and logical statements. Not every proposition must be executable or decidable, and an arbitrary effectful or nonterminating Boolean function cannot automatically be treated as a well-behaved logical predicate.

### 2.3 Contracts and invariants

Preconditions describe what callers must establish. Postconditions describe what an operation guarantees under those preconditions. Invariants describe conditions preserved at defined boundaries: for example, on entry and exit from public operations, or at a loop boundary.

The boundary matters. A data structure might temporarily violate an invariant during an internal update while preserving it at every externally observable point. A loop invariant alone also does not prove loop termination.

**Foundational reference.** Hoare's program logic supplies the classic basis for reasoning from preconditions to postconditions with inference rules, including reasoning about loops. It supports the separation between a program and propositions about its behavior; it does not prescribe Ash's annotation syntax or evidence policies. [Hoare 1969][R07]

Static verification can establish a contract about future executions without executing those future inputs during compilation. A claim about untrusted external input still requires a modeled assumption, a checked representation, or some other justified bridge. Declaring the input valid does not establish its validity.

## 3. Evidence, assumptions, and partial ordering

### 3.1 From a tower of validity to structured evidence

**Recorded direction.** The discussion proposed a tower or continuum involving axioms, tests, and proofs, while questioning whether “validity” was the right word.

**Working interpretation.** Evidence or assurance is more useful terminology. In logic, validity has a technical meaning; it is not naturally a numerical measure of how much a proposition has been tested. A proposition's truth, a proof's correctness, and an engineer's confidence are distinct concepts.

The continuum intuition captures increasing empirical coverage, but a single scale loses important distinctions. An axiom is not a weak test or an incomplete proof. It is an assumption on which other derivations may depend.

| Evidence or status | What it can establish | Limits that must remain visible |
| --- | --- | --- |
| Declaration without evidence | The claim has been stated | No justification follows from declaration alone |
| Assumption or axiom | The claim is available as a premise | Conclusions remain conditional on it |
| Example test | The observed execution meets the predicate | Limited to the case and execution semantics |
| Property-based testing | Many generated cases meet the predicate | Depends on sampling, generators, and coverage |
| Exhaustive finite checking | Every case in a precisely finite domain meets the predicate | Enumeration and domain completeness must be justified |
| Bounded model checking | A modeled claim holds within a stated bound | Does not automatically cover larger bounds or the implementation |
| Checked formal proof | A conclusion follows in a logic from its premises | Depends on assumptions, semantics, and checker soundness |

A concrete counterexample can refute a universal claim, provided it is a valid instance under the same semantics. A search timeout, failed tactic, or incomplete proof does not refute the claim.

### 3.2 Why a partial order is a better starting point

Two bodies of evidence may be incomparable. One test suite can cover inputs omitted by another. A proof about an ideal mathematical model may cover all model inputs while omitting an implementation detail exercised by a test. Neither should automatically replace the other.

For tests under the same semantics, evidence over a superset of cases can subsume evidence over a subset. For derivations, comparison also involves assumptions, the conclusion, the modeled program, and the interpretation of the domain.

Strictly, a raw subsumption relation may be a **preorder**: distinct evidence artifacts can justify equivalent uses. A partial order can be obtained by identifying artifacts equivalent under that relation. This is a mathematical clarification of the requested ordering idea, not a recovered decision about Ash's implementation.

A useful evidence description could include:

```text
claim + subject + scope + assumptions + method + certificate + provenance
```

This is a conceptual record, not an agreed data format. Its purpose is to prevent a claim proven for one domain, implementation, or assumption set from being silently reused for another.

### 3.3 Evidence and policy are different

A compiler policy can require a certified proof before using a law to justify an optimization. A development workflow might also collect tests for the same law. The test record remains useful without acquiring the logical force of a universal proof.

Formal verification is relative to premises. Ash should be able to distinguish “proved with these assumptions” from “assumed directly,” and expose transitive assumptions where they affect acceptance.

**References and limits.** QuickCheck is a cornerstone precedent for expressing executable properties and testing generated inputs. It supports the testing branch of this discussion, not a universal certification rule. Lean's account of axioms provides a concrete example of how logical assumptions interact with proofs and computation. The proposed evidence preorder is this document's synthesis; neither reference specifies it. [Claessen and Hughes 2000][R08]; [Lean, Axioms and Computation][R12]

## 4. Types, propositions, universes, and kinds

### 4.1 Shared logical foundation, different judgments

**Recorded direction.** The discussion observes that the type system is itself a system of propositions and asks how its relationship to verification should be organized.

**Working interpretation.** A typing judgment and a verification judgment can be represented schematically as:

```text
Gamma |- expression : Type
Gamma; assumptions |- evidence establishes Proposition
```

Both are judgments governed by rules. That does not force identical syntax, identical internal representations, or unrestricted interchange between ordinary values and proof evidence.

Types can describe representational and operational constraints. Propositions can express relationships among values, functions, states, or executions. Rich type systems can encode many such relationships; simpler type systems may delegate them to a separate verification layer.

### 4.2 Architectural alternatives

| Alternative | Organization | Principal design consequence |
| --- | --- | --- |
| Separate logical layer | Ordinary typing and verification use distinct judgments with explicit bridges | The bridge must define how program semantics enters the logic |
| Common framework, distinguished sorts | `Type` and `Prop` are distinct categories within shared machinery | Sharing is possible while preserving different formation and use rules |
| Propositions represented as types | A proof of `P` is a term inhabiting the type representing `P` | Logical consistency depends on the permitted proof terms and computation |

These are not entirely exclusive. A shared core might distinguish `Type` and `Prop` and still interpret proofs as inhabitants of propositions internally.

### 4.3 Kind is not the same question as universe level

A kind or sort classifies entities such as types, type constructors, or propositions, depending on the language. A universe level controls stratification in systems that need it. A phase determines when something is computed. Runtime representation and erasure are further independent choices.

Therefore, “propositions are a different kind in a uniform universe” is a plausible design question, but not a complete specification. It still leaves open:

- Which expressions form propositions, and which form computational types?
- Are propositions first-class compile-time values?
- Can propositions quantify over values, types, functions, or propositions?
- Is there a universe hierarchy, and what rules prevent inconsistency?
- Can evidence influence runtime computation, or is it erased?
- Are proofs of the same proposition interchangeable for computational purposes?
- Which facts can pass between type checking and verification?

**Open.** The accessible record asks these questions without establishing a final universe or kind design.

**Foundational references.** Martin-Löf distinguishes propositions from judgments and develops dependent types and universes. Barendregt's pure type systems provide a vocabulary for comparing sorts and permitted dependencies. Lean, Rocq, and Agda then demonstrate different concrete arrangements. Section 13 explains which choices protect consistency and which change expressiveness or usability. [Martin-Löf 1984][R02]; [Barendregt 1991][R04]; [Lean universes][R11]; [Rocq sorts][R13]; [Agda universes][R16]

## 5. Curry–Howard and alternative architectures

### 5.1 The correspondence and its limits

Curry–Howard relates propositions to types and proofs to terms/programs in appropriately structured systems. It does not mean that every arbitrary program in a general-purpose language is a valid logical proof.

The recorded discussion connects the intention of static compilation with a distinction from Curry–Howard. The important clarification is that **compile-time checking alone does not rule out Curry–Howard**. A system can type-check proof terms during compilation and erase them before runtime.

The more decisive question is whether Ash identifies propositions with types and proofs with inhabitants, or instead treats ordinary Ash programs as tools that construct separately checked derivations or certificates. The implementation could also combine these approaches.

### 5.2 Alternatives explored in this synthesis

1. **Proofs directly as typed terms.** Evidence is accepted by checking a term in a sound logical fragment. Unrestricted nontermination cannot simply count as inhabitation that establishes arbitrary propositions.
2. **Ordinary programs that construct proofs.** General Ash code builds a proof artifact, while a separate checker decides whether that artifact proves the goal.
3. **Verification-condition generation.** The compiler translates program obligations into logical formulas and discharges them through provers or other procedures.
4. **Hybrid architecture.** Ordinary Ash code orchestrates automation; a logical core validates the resulting evidence. The core itself may use proof terms and types.

The recorded preference for ordinary sandboxed compile-time code is compatible with the second and fourth arrangements. It does not settle the internal logical representation or prove that a Curry–Howard interpretation is absent.

**Foundational references.** Howard is the classic formulae-as-types source. LF shows how judgments in an object logic can be represented as types in a framework, while LCF demonstrates programmable proof construction controlled by an abstract theorem interface. These are related but distinct precedents for Ash. [Howard 1980][R01]; [Harper, Honsell, and Plotkin 1993][R06]; [Gordon, Milner, and Wadsworth 1979][R09]

## 6. Static verification during compilation

**Recorded direction.** Verification is intended to happen statically as part of compilation.

**Working interpretation.** The following dependency flow makes that intention concrete without fixing a compiler implementation:

```text
source declarations and implementations
                  |
                  v
name resolution, typing, and obligation formation
                  |
                  v
sandboxed compile-time Ash proof construction
                  |
                  v
candidate evidence or proof certificate
                  |
                  v
certification against goal, context, and program semantics
                  |
                  v
evidence-dependent acceptance and code generation
```

This is a logical dependency graph. Type inference and verification may need to interleave rather than occupy isolated passes. Certification nevertheless must precede any compiler decision that relies on the certified fact.

Static verification is not synonymous with evaluating an assertion on concrete values. It can prove a universally quantified fact about runtime behavior using symbolic reasoning. Compile-time tests remain tests unless they exhaust a justified finite domain or contribute to a valid derivation.

Useful distinct outcomes include:

| Outcome | Meaning |
| --- | --- |
| Certified | The evidence establishes the stated goal under its recorded premises |
| Counterexample found | A valid case contradicts the claim |
| Incomplete or unknown | The attempted method did not establish the claim |
| Invalid certificate | The submitted artifact fails the certification rules |
| Resource limit reached | Construction or checking exceeded its budget |
| Assumed | The proposition is admitted as a premise rather than derived |

**Open.** Exact diagnostics and whether an unresolved obligation rejects compilation depend on how the obligation is used and on policies not yet specified. A required proof cannot be silently replaced by a timeout or a successful sample test.

**Reference.** F* combines dependent types, verification conditions, SMT automation, and a separation between pure specifications and effectful computations. This is a precedent for static verification integrated into a programming language, not a claim that F* implements Ash's proposed independent certification policy. [Swamy et al. 2016][R19]

## 7. Ordinary sandboxed Ash proof-construction code

### 7.1 The recorded preference

The preference is to use ordinary Ash code, executed at compile time in an appropriate sandbox. This avoids requiring a separate user-facing proof-programming language as the only means of constructing evidence.

“Ordinary” concerns the programming language used for construction. It does not imply that all runtime capabilities must be available during compilation, or that an arbitrary return value becomes a proof.

### 7.2 Construction versus justification

An ordinary program can search, recurse, manipulate syntax, select lemmas, and call permitted automation. Its output is a candidate artifact. The checker must establish that the artifact derives the requested proposition.

This separates two questions:

```text
Did the construction program execute successfully?
Does its result actually establish this goal in this context?
```

Successful execution answers only the first question. Returning `true`, producing an object with a `Proof` label, or asserting that a solver succeeded is insufficient unless the certification architecture explicitly justifies that transition.

A buggy or malicious constructor should be able to fail, time out, or submit rejected evidence. It should not be able to forge accepted evidence by bypassing the checker.

### 7.3 What sandboxing needs to address

**Working interpretation; details remain open.** A sandbox must protect the build environment and support predictable compilation. Candidate controls include explicit capabilities, restricted file/network access, time and memory budgets, and defined treatment of randomness and other ambient inputs.

Sandbox safety and logical soundness are separate. A perfectly isolated program can still generate a false argument. Conversely, a logically correct certificate does not make its constructor safe to execute with unrestricted host access.

Ordinary construction code need not necessarily be total if the system can terminate its execution and reject unresolved obligations. Whether Ash instead requires a restricted total fragment is a separate choice. The accessible record does not decide it.

**References.** LCF's ML establishes the historical precedent for using a programming language to implement proof procedures. Lean's architecture and Meta-F* provide further examples of programmable automation. These references motivate the construction/checking distinction; they do not establish a host-security sandbox for Ash. [Edinburgh LCF][R09]; [Lean system description][R10]; [Martínez et al. 2019][R20]

## 8. Independent certification and trust

**Working interpretation.** Independent certification is a way to preserve the freedom of ordinary proof-construction code while keeping acceptance tied to a controlled logical basis. It is included because the current request specifically asks for it; its exact prior treatment is unavailable.

“Independent” means the certifier can validate the evidence against the goal without trusting the constructor's assertion of success. It need not mean a separate executable, process, vendor, or language.

The certifier needs enough context to determine:

- Which proposition is established and which premises are used.
- Which program definitions and semantic interpretation the proof concerns.
- Whether every inference step is permitted.
- Whether references to lemmas and assumptions are valid.
- Whether the submitted artifact is applicable to the present compilation.

Opaque certificate constructors and controlled evidence registration are possible implementation techniques. They are not a substitute for sound checking rules.

### 8.1 External solvers

If automation relies on an external solver, the design can require a checkable certificate, replay a derivation in the logical core, or explicitly trust that solver for designated obligations. These options have different trust implications and should be distinguishable in evidence metadata.

A bare solver verdict is not independently certified merely because the solver runs during compilation. Likewise, a successful compiler run does not establish more than the obligations and assumptions its verification policy actually covers.

### 8.2 The trusted boundary

A small checker is an attractive goal, but the full trust story also includes the logical rules, admitted axioms, the connection between the logic and Ash semantics, and the translation from source programs to verified representations. If a verified source property is relied on for executable behavior, preservation through code generation matters too.

Thus, “the constructor need not be trusted for logical correctness” is narrower and more precise than “only the checker needs to be trusted.”

**References.** Proof-carrying code separates evidence production from consumer-side validation. SMTCoq demonstrates a proved-correct checker for certificates from external solvers. These support independent certification as an architectural option without making either a mandatory implementation for Ash. [Necula 1997][R25]; [Ekici et al. 2017][R23]

## 9. Illustrative syntax and end-to-end examples

**All examples below are proposed pseudocode, not recovered syntax or executable Ash.** They illustrate distinctions rather than specify grammar. Operations such as `logic.forall` are conceptual proof-construction APIs.

### 9.1 A proposition and an algebraic law

```text
prop Associative<T>(op: (T, T) -> T) =
    forall a: T, b: T, c: T.
        op(op(a, b), c) == op(a, op(b, c))

interface Semigroup<T> {
    fn combine(a: T, b: T) -> T
    law associativity: Associative(combine)
}
```

`Associative` expresses a proposition schema. `law` gives that proposition a role in the abstraction's interface. Neither declaration alone proves that a particular `combine` implementation satisfies it.

The domain and equality matter. Mathematical integers, fixed-width machine integers, and floating-point numbers do not automatically support the same algebraic reasoning. Any transferred law needs the actual operation's semantics.

### 9.2 Contracts and an invariant

```text
fn maximum(xs: List<Int>) -> Int
    requires xs.length > 0
    ensures result in xs
    ensures forall x in xs. x <= result
{
    // Implementation omitted.
}

struct OrderedTree<T> {
    // Representation omitted.
    invariant ordered_by(comparator, self)
}
```

The contract generates caller and implementation obligations. The invariant additionally needs a specified preservation boundary and requirements on `comparator`. A compile-time system must justify those obligations; the annotations are not their evidence.

### 9.3 Test evidence for a law

```text
comptime {
    samples = generate_cases(seed = 42, count = 10000)
    report = test_associativity(my_combine, samples)
    record_test_evidence(Associative(my_combine), report)
}
```

The report supports the tested instances. It is not a universal proof merely because the test loop ran at compile time. The seed, generator, actual cases, and operation semantics define the report's scope.

### 9.4 Ordinary Ash code constructs a candidate proof

Consider the simple equality reflexivity proposition:

```text
prop ReflexiveEquality<T> =
    forall x: T. x == x

fn construct_reflexivity(t: TypeDescriptor) -> CandidateProof {
    return logic.forall(t, fn(x) {
        return logic.equality_reflexivity(x)
    })
}

comptime {
    goal = ReflexiveEquality<Int>
    candidate = construct_reflexivity(Int)
    certified = certify(goal, candidate)
    register_evidence(goal, certified)
}
```

This assumes `==` denotes logical equality with a reflexivity rule. It must not silently stand for an arbitrary user-defined comparison or an operation with different semantics. The constructor is ordinary code; the certifier validates the claimed universal derivation.

The example leaves representations such as `TypeDescriptor`, quotation, binding, and hygienic symbolic variables unspecified. Those mechanisms are needed for a real API but were not settled by the accessible discussion.

### 9.5 Assumptions and evidence reuse

```text
assume backend_combine_associative:
    Associative(backend.combine)

derive aggregate_regrouping
    using backend_combine_associative
```

The derived result remains conditional on the backend assumption. Registering it must not erase that dependency. Reusing a certificate for a changed backend or a different operation would require justification that the original proposition still applies.

## 10. Decisions, interpretations, and open questions

### 10.1 Recorded directions

| Direction | Support and limit |
| --- | --- |
| Investigate evidence beyond a simple validity label | The user explicitly discusses axioms, tests, and proofs and questions “validity” |
| Relate verification and typing through shared logical foundations | Explicitly raised, with separation along other dimensions requested |
| Explore a proposition/property universe alongside types | Explicit question; final architecture not visible |
| Perform verification statically during compilation | Explicit stated intention |
| Use appropriately sandboxed ordinary compile-time code | Explicit stated preference; this document uses the requested name Ash |
| Defer additional complications | Explicit instruction in the source discussion |
| Treat the discussion as having reached a foundation | Explicit final acceptance, with the accepted assistant formulation unavailable |

### 10.2 Working interpretations, not established decisions

- Laws, contracts, and invariants can be understood as roles for propositions.
- Evidence should preserve scope and assumptions instead of using a single scalar strength.
- Evidence subsumption suggests a preorder or a partial order after equivalence is accounted for.
- Proof construction should be distinguished from certification.
- Independent certification can support ordinary Ash construction code without trusting it as a logical authority.
- Static phase selection alone does not decide whether Curry–Howard is used.
- A shared logical core can coexist with distinct typing and verification judgments.

### 10.3 Open questions

| Area | Question still requiring a decision |
| --- | --- |
| Logical language | Which connectives, quantifiers, equality rules, induction principles, and axioms exist? |
| Kinds and universes | Are `Type` and `Prop` separate sorts, separate layers, or related through an embedding? |
| Expressiveness | Which program values, states, effects, functions, and types can propositions mention? |
| Proof representation | Are artifacts proof terms, derivation trees, solver certificates, or several formats? |
| Construction API | How does ordinary Ash quote propositions, bind symbols, select lemmas, and compose derivations? |
| Certification | What is trusted, and can every accepted certificate be checked independently of its producer? |
| Assumptions | How are direct and transitive assumptions declared, displayed, and constrained? |
| Evidence comparison | What exactly counts as subsumption, and which kinds of evidence are incomparable? |
| Tests | Are tests first-class evidence, and when can finite exhaustive checking certify a proposition? |
| Sandbox | Which capabilities, budgets, determinism guarantees, and failure behavior apply? |
| Typing integration | Which obligations affect type formation, overload resolution, or generic instantiation? |
| Static failure policy | Which unresolved obligations reject compilation, and which can remain informational? |
| Runtime boundary | Is evidence erased, and how are external input assumptions represented without hiding obligations? |
| Program semantics | How are overflow, floating point, effects, divergence, and other behavior represented in the logic? |
| Artifact reuse | How are certificates associated with definitions and invalidated after changes? |
| Syntax | What declaration forms and terminology best express the final distinctions? |

These questions record boundaries for future discussion. They are not additional requirements imposed on the current foundation.

## 11. Discussion paths and conceptual corrections

Only the latter part of the sequence is directly recoverable. The earlier laws/properties framing is supplied by the title and current request. The following map distinguishes the visible progression from this document's reconstruction.

| Path | Question or intuition | Resulting distinction | Provenance |
| --- | --- | --- | --- |
| Laws versus properties | Do these name different sorts of claim? | Broad property versus algebraic/interface role | Requested topic; original turns unavailable |
| A tower of validity | How do axioms, tests, and proofs relate? | Truth, assumptions, evidence, and confidence are different | Tower intuition recorded; refinement reconstructed |
| From continuum to ordering | Can evidence be arranged from weak to strong? | Scope and premises can make evidence incomparable | Requested topic; formalization reconstructed |
| Typing and propositions | Does verification sit beside or mesh with typing? | Shared logical basis need not mean identical judgments | Recorded question; explanation reconstructed |
| Universes and kinds | Are propositions another kind in one universe? | Classification, universe level, phase, and erasure are separate | Recorded question; alternatives reconstructed |
| Expression and checking time | How are proofs written, and when are they validated? | Surface language, proof representation, construction, and checking phase | Recorded question |
| Curry–Howard comparison | Does static verification make the design distinct? | Compile-time checking is compatible with Curry–Howard; architecture decides the relationship | Recorded intuition; clarification added here |
| Ordinary compile-time code | Can the host language construct proofs? | General construction code and certified evidence serve distinct roles | Preference recorded; architecture reconstructed |
| Independent certification | How can ordinary construction remain logically controlled? | Constructor success and proof acceptance must be distinguished | Requested topic; treatment reconstructed |
| Scope boundary | Should all related complications be resolved now? | Keep a foundation while leaving mechanisms open | Deferral recorded |

The strongest common thread is separation of concerns: what is claimed, what role the claim plays, what evidence exists, how that evidence is obtained, what accepts it, and when acceptance occurs.

## 12. Compact glossary

| Term | Meaning |
| --- | --- |
| Proposition | A statement in the logical language |
| Property | A characteristic expressible as a predicate or proposition |
| Law | A property used as an algebraic or abstraction-level obligation |
| Contract | Preconditions, postconditions, or related interface obligations |
| Invariant | A property required at defined states or boundaries |
| Assumption | A proposition admitted as a premise |
| Evidence | Support for a claim, with a defined scope and premises |
| Proof | A derivation in a specified logic |
| Certificate | An artifact submitted for checking; it is not accepted merely by being labeled a certificate |
| Proof constructor | A program that builds candidate evidence |
| Certifier | The mechanism checking evidence against a proposition and context |
| Kind or sort | A classification of language entities, according to the language's rules |
| Universe | A logical classification level or domain; its exact meaning must be specified |
| Static phase | Work performed during compilation rather than application execution |
| Sandbox | Restrictions on the capabilities and resources of executed construction code |
| Trusted computing base | Components and assumptions whose correctness underlies accepted conclusions |

## 13. Logical foundations and the reasons behind the choices

This section adds a literature-based rationale to the preceding design discussion. The cited works document logical systems and architectures. Statements headed **Implication for Ash** are design interpretations, not newly adopted decisions or claims that the papers uniquely determine Ash's design.

### 13.1 Propositions, judgments, and metalanguages

Martin-Löf's *Intuitionistic Type Theory* distinguishes a proposition from a judgment asserting its truth. It also distinguishes formation, inhabitation, and equality judgments. This is a stronger foundation than treating every sentence about a program as the same kind of object. [Martin-Löf 1984, opening chapters][R02]

For Ash, the corresponding distinctions could be written schematically as:

```text
Gamma |- P proposition             P is a well-formed claim
Gamma |- e : T                     e is a well-typed program expression
Gamma; A |- pi proves P            pi derives P under assumptions A
certify(Gamma, A, P, certificate)   an algorithm checks a submitted derivation
```

The last line describes an implementation activity; it is not an additional logical axiom. A claim about successful certification would need its own connection to the derivability judgment.

LF provides a disciplined way to represent judgments and derivations through dependent types in a metalanguage. Isabelle provides another precedent for distinguishing a framework's meta-logic from the object logics represented in it. Thus, using types to implement a checker does not force every Ash runtime type to double as a user-facing proposition. [Harper, Honsell, and Plotkin 1993][R06]; [Paulson 1989][R18]

**Implication for Ash.** Separate the object being verified, its specification language, and the language implementing proof procedures before deciding whether their syntax should be unified.

### 13.2 Types, sorts, kinds, and dependencies

Barendregt's lambda cube distinguishes polymorphism, dependencies of types on terms, and type-level operators. Pure type systems generalize the presentation using sorts, axioms assigning sorts to sorts, and rules governing dependent product formation. Their value here is that a choice of “kind” acquires meaning through explicit rules, not merely through a name. [Barendregt 1991][R04]

For example, the following questions are independent:

| Question | What the answer controls |
| --- | --- |
| Can a type mention a value, as in `Vector<T, n>`? | Dependency of types on terms |
| Can a definition quantify over a type? | Polymorphism |
| Can type constructors take type constructors as arguments? | Higher-order type-level structure, often classified with kinds |
| Does a proposition inhabit a distinguished sort? | Its formation, elimination, and conversion rules |
| At what level does a collection of types live? | Universe stratification |
| When is an expression evaluated? | Staging and execution phase |

The Calculus of Constructions is a cornerstone of the dependent-type lineage relevant to Lean and Rocq. It should not be conflated with the entire modern Calculus of Inductive Constructions: inductive definitions and their rules add further structure. [Coquand and Huet 1988][R03]; [Lean system description][R10]; [Rocq inductive types][R14]

**Implication for Ash.** “One uniform universe with different kinds” remains underspecified until the system states the sort-assignment rules, permitted dependencies, conversion relation, and universe constraints.

Terminology also varies across systems. In Isabelle, a *sort* is a finite intersection of type classes used to constrain type variables, rather than a dependent-type universe level. Ash should define its own terminology rather than assume that “sort” has one meaning across all proof assistants. [Isabelle/Isar Implementation, §2.1](https://isabelle.in.tum.de/doc/implementation.pdf)

### 13.3 Why universe stratification exists

An unrestricted self-typing universe in a sufficiently expressive dependent type theory can destroy consistency. Hurkens gives a compact contradiction in the pure type system `U−`, following Girard's earlier paradox. The precise combination of product-formation and universe rules matters: this is not a claim that every notation resembling self-reference is inconsistent. [Hurkens 1995][R05]

Universe hierarchies supply a controlled alternative. Their point is to permit reasoning about types without making all types, including their classifier, collapse into one unrestricted collection. Universe polymorphism then allows one definition to be reused at multiple levels. Agda explicitly motivates its hierarchy and level polymorphism in these terms. [Agda, Universe Levels][R16]

The schematic rule is:

```text
Type[u] : Type[u + 1]
```

This is not the rule `Type[u] : Type[u]`. Hiding level annotations in surface syntax does not eliminate the constraints.

**Implication for Ash.** If propositions and types become first-class objects within an expressive logical core, the compiler needs a justified stratification or another established consistency-preserving discipline. If the core instead uses a simpler object logic, it need not import every dependent-universe mechanism. This is an architectural choice, not a requirement to copy Lean.

### 13.4 Predicativity, impredicativity, cumulativity, and proof identity

These choices answer different questions:

- **Predicativity:** Does quantifying over a domain force the result into a sufficiently large universe?
- **Impredicativity:** Can some quantification range over a collection including the proposition/type being formed without raising its universe?
- **Cumulativity:** Can a type accepted in a lower universe also be accepted in a higher one through a subtyping rule?
- **Proof irrelevance:** Are different proofs of the same proposition identified? If so, is that identity definitional or a theorem/axiom?

Lean has an impredicative `Prop = Sort 0`, a predicative `Type u = Sort (u + 1)` hierarchy, and noncumulative universes. Its type system makes proofs of the same proposition definitionally equal. Rocq has a different sort/subtyping organization, including `Prop`, `Set`, and `Type`; `SProp` explicitly supplies definitional proof irrelevance. Rocq's ordinary `Prop` should not be described as having Lean's blanket definitional proof irrelevance. [Lean universes][R11]; [Lean type system][R26]; [Rocq sorts][R13]; [Rocq SProp][R15]

Agda provides a useful counterexample to treating impredicativity as inevitable: its optional proof-irrelevant `Prop` has a predicative hierarchy. A distinguished proposition sort therefore does not, by itself, settle universe closure rules. [Agda Prop][R17]

Rocq's standard rules make `Prop` impredicative and provide a cumulative `Type` hierarchy: lower universe membership can be used at a higher level through subtyping. Its `SProp` is excluded from that cumulativity relation. This makes the contrast with Lean precise: stratification, closure under quantification, and cumulativity are separate rules, not three names for a single hierarchy. [Rocq 9.0.1, Typing Rules](https://rocq-prover.org/doc/v9.0/refman/language/cic.html)

**Rationale for consideration in Ash.** An impredicative proposition sort can keep logical formulas in one sort when they quantify over large domains. Predicativity keeps universe dependencies explicit. Cumulativity can reduce the need for explicit lifting, while noncumulativity uses stricter level matching. Definitional proof irrelevance can simplify comparison of terms containing proofs, but commits the logic to treating proof identity as unobservable. These are explanations of design consequences, not assertions that one choice is necessary or universally better.

### 13.5 Why elimination rules and equality need their own decisions

Allowing a proposition to mention data is different from allowing a program to extract data by inspecting its proof. Rocq restricts elimination from `Prop` into computational sorts, with defined exceptions for empty and suitable singleton inductives. Its manual relates these restrictions to extraction and consistency. [Rocq, Destructors][R14]

The distinction can be seen in this proposed Ash notation:

```text
exists x: T. P(x)                 // proposition asserting existence
Witness<T, P> = (x: T, proof: P(x))  // informative object containing a witness
```

Whether the first form can produce the second depends on the logic and any choice principle. The two must not be equated merely because both informally say that a witness exists.

Also distinguish **definitional equality**, checked by the core's conversion mechanism, from **propositional equality**, expressed by a proposition with evidence. Rocq documents conversion rules separately; Lean discusses how additional axioms affect computation. A proof of extensional equality need not make two expressions interchangeable by reduction alone. [Rocq conversion][R27]; [Lean, Axioms and Computation][R12]

**Implication for Ash.** Specify proof elimination, erasure, equality, and classical principles together. If runtime behavior must not depend on proof identity, the logical interface must enforce that boundary. “Erase proofs later” is insufficient as a logical specification.

### 13.6 Ordinary proof code: LCF, Lean, and Meta-F*

Edinburgh LCF used ML to program proof procedures while controlling theorem construction through an abstract theorem type. This is the closest historical precedent for the ordinary-code preference: programmers can implement sophisticated search without receiving authority to invent theorems. [Gordon, Milner, and Wadsworth 1979][R09]; [Paulson, discussion of programmable LCF][R28]

Lean's documented architecture instead emphasizes generated proof terms checked by a small kernel. Meta-F* combines metaprograms, tactics, and SMT and supports interpreted or compiled metaprograms. Neither precedent makes the construction language itself the sole criterion of logical validity. [Lean reference, introduction][R29]; [Martínez et al. 2019][R20]

Two architectures should remain distinct:

| Architecture | Acceptance mechanism | Artifact implication |
| --- | --- | --- |
| LCF-style protected theorem interface | Theorem values arise only through trusted inference operations | A separately exportable proof certificate is not automatic |
| Certificate or proof-term validation | A checker validates an explicit derivation | Independent replay is possible if all required context is provided |

**Implication for Ash.** Ordinary sandboxed Ash can orchestrate either architecture. If independent offline certification is a goal, explicitly require exportable evidence and a checker interface; an opaque in-memory theorem value alone does not provide it. Capability restrictions and execution budgets remain separate implementation work.

### 13.7 Static verification, effects, and termination

Hoare logic supplies the foundation for a statement about an effectful program's behavior without identifying that program with its correctness proof. F* demonstrates an integrated language in which effects and divergence are separated from a pure logical core, with weakest-precondition reasoning and SMT-assisted verification. [Hoare 1969][R07]; [Swamy et al. 2016][R19]

This supports an important separation for Ash:

```text
construction program may fail or exhaust a budget
                         !=
logical core may accept arbitrary divergence as evidence
```

**Implication for Ash.** Requiring totality of proof *terms used in the logic* and requiring totality of every proof *search program* are different policies. The latter can be relaxed if failure is controlled and nothing is accepted without valid evidence. Kernel conversion and checking still need a defined computational discipline.

### 13.8 SMT sorts are not dependent-type universes

Z3's foundational system paper describes SMT as satisfiability reasoning over combinations of background theories. SMT-LIB specifies the sort and command language. An SMT sort such as `Int`, `Bool`, or an array sort classifies terms in that logic; it is not automatically a `Type u` universe containing types as ordinary members. [de Moura and Bjørner 2008][R22]; [SMT-LIB 2.7][R21]

For a classical verification condition, the usual reduction is:

```text
Assumptions entails Goal
             reduce to checking
Assumptions AND NOT Goal is unsatisfiable
```

**Implication for Ash.** A dependent or higher-order front end can use an SMT backend, but must justify the encoding and reconstruction. Arithmetic over mathematical integers cannot silently replace machine-word semantics. Classical refutation also does not automatically provide a constructive witness in a constructive core.

SMT-LIB distinguishes `sat`, `unsat`, and `unknown`; it also offers an optional proof command with solver-specific output. Neither an inconclusive result nor an unsatisfiable subset of assumptions is, by itself, a checked derivation in Ash's logic. [SMT-LIB 2.7, command semantics][R21]

### 13.9 SMT certificates and the certification boundary

SMTCoq is a direct precedent for checking external solver results with a checker implemented and proved correct in a proof assistant. It illustrates how automation can remain outside the logical trust boundary while its accepted evidence is checked inside. The 2017 paper describes specific supported theories and solvers; it should not be read as a universal bridge for arbitrary SMT formulas. [Ekici et al. 2017][R23]

cvc5's LFSC documentation describes a proof framework with computational side conditions and a separate checker. The cited version also documents possible **trust steps**, which admit formulas without full justification. Therefore, generating a proof file does not alone establish that every step was independently checked. [cvc5 1.3.2, LFSC proofs][R24]

Proof-carrying code supplies the broader producer/consumer precedent: the consumer checks evidence rather than trusting how it was obtained. It does not remove the need to connect the checked proposition to the actual executable semantics. [Necula 1997][R25]

**Implication for Ash.** Specify the supported certificate calculus, permitted side conditions, treatment of trust steps, and validation of the source-to-solver translation. Distinguish a checked certificate from a trusted solver verdict.

### 13.10 Comparison and reading path

| Tradition or system | Relevant logical or architectural choice | What Ash can learn without copying the whole system |
| --- | --- | --- |
| Martin-Löf type theory | Explicit judgments, dependent types, universes | State formation and evidence rules before designing syntax. [R02][R02] |
| CoC / pure type systems | Dependencies and sorts given by formal rules | Define what “kind” permits, rather than treating it as a label. [R03][R03], [R04][R04] |
| Lean | Distinguished `Prop`, checked proof terms | Shared type-theoretic machinery can retain different data/proof rules. [R10][R10], [R11][R11] |
| Rocq | Multiple sorts with distinct elimination and relevance behavior | Proposition/data separation needs explicit operational rules. [R13][R13]–[R15][R15] |
| Agda | Predicative universes and optional proof-irrelevant propositions | Proof irrelevance and impredicativity are independent choices. [R16][R16], [R17][R17] |
| LCF / Isabelle | Programmable reasoning with a controlled theorem interface; meta/object logic separation | Ordinary proof code and trusted inference need not coincide. [R09][R09], [R18][R18] |
| F* / Meta-F* | Effects, static obligations, SMT, metaprogramming | Integrate proof automation without confusing effects with logical evidence. [R19][R19], [R20][R20] |
| SMT / SMTCoq / LFSC | Theory-specific automation and certificate checking | Define encoding and trust boundaries explicitly. [R21][R21]–[R24][R24] |

For a compact reading path, start with **Howard → Martin-Löf → Barendregt** for the logical vocabulary; **LCF → Lean → Rocq/Agda documentation** for implementation choices; and **Hoare → F*/Meta-F* → SMTCoq** for programming-language integration and automation. Hurkens supplies the warning about inconsistent combinations of universe rules; proof-carrying code supplies the independent-certification model.

The literature gives Ash a set of defensible choices, not a uniquely mandated foundation. The present discussion most strongly supports a separation between ordinary proof construction and controlled acceptance. It does not yet select a particular proposition sort, universe hierarchy, equality theory, or certificate calculus.

## 14. Annotated primary references

**Reference policy.** Links below lead to original papers, authors' or institutions' copies, publisher records, or official manuals. References were checked on 23 October 2026. Manuals describe the cited versions/pages; historical papers establish provenance and should not be treated as specifications of current releases. An annotation identifies where to read and what question it informs. No direct quotations are used.

### Foundational logic and type theory

1. **[R01 — William A. Howard, “The Formulae-as-Types Notion of Construction” (1980; notes circulated in 1969)][R01].** In *To H. B. Curry: Essays on Combinatory Logic, Lambda Calculus and Formalism*, pp. 479–490. The classic source for the proof/type correspondence discussed in §§5 and 13.1. Linked institutional scan was discoverable; full extraction was unavailable in this session.
2. **[R02 — Per Martin-Löf, *Intuitionistic Type Theory* (1984)][R02].** Bibliopolis; notes by Giovanni Sambin. Read the opening chapters on propositions and judgments, the equality chapters, and “Universes.” Supports §§4 and 13.1–13.3.
3. **[R03 — Thierry Coquand and Gérard Huet, “The Calculus of Constructions” (1988)][R03].** *Information and Computation* 76(2–3), pp. 95–120. DOI: `10.1016/0890-5401(88)90005-3`. Foundational dependent-type calculus; linked copy is the 1986 INRIA report version. Relevant to §§4–5 and 13.2.
4. **[R04 — Henk Barendregt, “Introduction to Generalized Type Systems” (1991)][R04].** *Journal of Functional Programming* 1(2), pp. 125–154. DOI: `10.1017/S0956796800020025`. Read for the lambda cube and generalized/pure type systems: a precise language for sorts and dependencies in §13.2.
5. **[R05 — Antonius J. C. Hurkens, “A Simplification of Girard's Paradox” (1995)][R05].** TLCA, LNCS 902, pp. 266–278. DOI: `10.1007/BFb0014058`. A specific inconsistency result, relevant to why universe rules require metatheoretic justification (§13.3).
6. **[R06 — Robert Harper, Furio Honsell, and Gordon Plotkin, “A Framework for Defining Logics” (1993)][R06].** *Journal of the ACM* 40(1), pp. 143–184. DOI: `10.1145/138027.138060`. Read for LF and judgments-as-types; informs the distinction between Ash's object logic and its representation.

### Program specifications and testing

7. **[R07 — C. A. R. Hoare, “An Axiomatic Basis for Computer Programming” (1969)][R07].** *Communications of the ACM* 12(10), pp. 576–580, 583. DOI: `10.1145/363235.363259`. The cornerstone program-logic reference for contracts, invariants, and reasoning about behavior (§§2, 6, 13.7).
8. **[R08 — Koen Claessen and John Hughes, “QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs” (2000)][R08].** ICFP, pp. 268–279. The testing precedent for executable properties and generators (§3), not a justification for promoting sampled tests to universal proofs.

### Proof assistants and their logical cores

9. **[R09 — Michael J. Gordon, Arthur J. Milner, and Christopher P. Wadsworth, *Edinburgh LCF: A Mechanized Logic of Computation* (1979)][R09].** LNCS 78. DOI: `10.1007/3-540-09724-4`. Historical foundation for programmable proof procedures and a controlled theorem interface (§§7, 13.6).
10. **[R10 — Leonardo de Moura, Soonho Kong, Jeremy Avigad, Floris van Doorn, and Jakob von Raumer, “The Lean Theorem Prover (System Description)” (2015)][R10].** CADE-25. Describes Lean's small-kernel architecture and interactive/automated proving goals; use current manuals for Lean 4 details.
11. **[R11 — Lean Language Reference, “Universes”][R11].** Official reference, accessed 23 October 2026. Covers `Sort`, `Prop`, `Type`, predicativity, noncumulativity, and level polymorphism (§13.4).
12. **[R12 — *Theorem Proving in Lean 4*, “Axioms and Computation”][R12].** Official project text. Read for the relationship between axioms, logical strength, and computational content (§§3, 13.5).
13. **[R13 — Rocq Reference Manual 9.0.1, “Sorts”][R13].** Official specification of `SProp`, `Prop`, `Set`, and the `Type` hierarchy. Relevant to the formation and classification questions in §§4 and 13.4.
14. **[R14 — Rocq Reference Manual 9.0.1, “Inductive Types and Recursive Functions”][R14].** Especially “Destructors” and empty/singleton elimination. Explains constraints on moving from propositional proofs to computational values (§13.5).
15. **[R15 — Rocq Reference Manual 9.2, “SProp (Proof Irrelevant Propositions)”][R15].** Official description of strict propositions. Relevant to the distinction between ordinary `Prop` and definitional proof irrelevance; the page marks the feature experimental.
16. **[R16 — Agda User Manual, “Universe Levels”][R16].** Official account of stratification and level polymorphism. Provides an explicit explanation of why a hierarchy is used and why level-polymorphic abstractions need care (§13.3).
17. **[R17 — Agda User Manual, “Prop”][R17].** Official account of optional proof-irrelevant propositions, their hierarchy, and restricted elimination. Useful for separating proof irrelevance from impredicativity (§13.4).
18. **[R18 — Lawrence C. Paulson, “The Foundation of a Generic Theorem Prover” (1989)][R18].** *Journal of Automated Reasoning* 5, pp. 363–397. DOI: `10.1007/BF00248324`. Isabelle's meta-logic/object-logic approach (§13.1).

### Verification languages and metaprogramming

19. **[R19 — Nikhil Swamy et al., “Dependent Types and Multi-Monadic Effects in F*” (2016)][R19].** POPL. Original design account connecting dependent types, effects, termination, weakest preconditions, and SMT (§§6, 13.7).
20. **[R20 — Guido Martínez et al., “Meta-F*: Proof Automation with SMT, Tactics, and Metaprograms” (2019)][R20].** ESOP. A particularly relevant precedent for ordinary-language proof automation and metaprograms interacting with SMT (§§7, 13.6).

### SMT and independent certification

21. **[R21 — Clark Barrett, Pascal Fontaine, and Cesare Tinelli, *The SMT-LIB Standard*, version 2.7, release 7 July 2025][R21].** A fixed release, not a claim about the latest standard. Read the logical-language and command-semantics chapters for sorts, satisfiability responses, and `get-proof` (§13.8).
22. **[R22 — Leonardo de Moura and Nikolaj Bjørner, “Z3: An Efficient SMT Solver” (2008)][R22].** TACAS. Foundational system paper for theory-based automated reasoning; use it for the solver model, not as a present-day certificate-format specification.
23. **[R23 — Burak Ekici, Alain Mebsout, Cesare Tinelli, Chantal Keller, Guy Katz, Andrew Reynolds, and Clark Barrett, “SMTCoq: A Plug-In for Integrating SMT Solvers into Coq” (2017)][R23].** CAV, pp. 126–136. A direct reference for certified external-solver integration (§§8, 13.9).
24. **[R24 — cvc5 1.3.2 documentation, “Proof Format: LFSC”][R24].** Official documentation. Read for LFSC, side conditions, checking, and the explicit limitation concerning trust steps (§13.9).
25. **[R25 — George C. Necula, “Proof-Carrying Code” (1997)][R25].** POPL. DOI: `10.1145/263699.263712`. Linked author's publication page provides the original paper and related reports. Supports the producer/consumer evidence architecture, not a specific Ash logic (§§8, 13.9).

### Supplementary implementation and reference sources

26. **[R26 — Lean Language Reference, “The Type System”][R26].** Official description of the core, including proof irrelevance. For implementation inspection, the GitHub connector also located Lean's [kernel type checker at commit `f37a0276ae8b0c60dd705920c49b9fd0c16279cf`](https://github.com/leanprover/lean4/blob/f37a0276ae8b0c60dd705920c49b9fd0c16279cf/src/kernel/type_checker.cpp). The source is a concrete cross-check, not a metatheoretic soundness proof or a claim that this commit matches every cited manual.
27. **[R27 — Rocq Reference Manual 9.0.1, “Conversion Rules”][R27].** Official explanation of reduction and conversion, relevant to definitional versus propositional equality (§13.5).
28. **[R28 — Lawrence C. Paulson, *Designing a Theorem Prover*, Cambridge technical report 192 (1990), §6.3][R28].** A first-hand explanatory account of programmable LCF, ML, and the abstract theorem type. Supplements the historical LCF book with an accessible architectural description.
29. **[R29 — Lean Language Reference, introduction][R29].** Official account of the division between tactics and kernel-checked proof terms. Complements the historical Lean paper for the current architecture described in §13.6.

[R01]: https://www.inf.ed.ac.uk/teaching/courses/inf1/fp/2009/lectures/lect20.pdf
[R02]: https://archive-pml.github.io/martin-lof/pdfs/Bibliopolis-Book-retypeset-1984.pdf
[R03]: https://www.asc.ohio-state.edu/pollard.4/type/readings/coc88.pdf
[R04]: https://www.cambridge.org/core/journals/journal-of-functional-programming/article/introduction-to-generalized-type-systems/869991BA6A99180BF96A616894C6D710
[R05]: https://link.springer.com/chapter/10.1007/BFb0014058
[R06]: https://era.ed.ac.uk/items/d9cebf2b-52bd-4532-b848-7d0402f32cf5/full
[R07]: https://www.cs.cmu.edu/~crary/819-f09/Hoare69.pdf
[R08]: https://research.chalmers.se/en/publication/237427
[R09]: https://link.springer.com/book/10.1007/3-540-09724-4
[R10]: https://lean-lang.org/papers/system.pdf
[R11]: https://lean-lang.org/doc/reference/latest/The-Type-System/Universes/
[R12]: https://lean-lang.org/theorem_proving_in_lean4/Axioms-and-Computation/
[R13]: https://rocq-prover.org/doc/v9.0/refman/language/core/sorts.html
[R14]: https://rocq-prover.org/doc/v9.0/refman/language/core/inductive.html
[R15]: https://rocq-prover.org/doc/V9.2.0/refman/addendum/sprop.html
[R16]: https://agda.readthedocs.io/en/latest/language/universe-levels.html
[R17]: https://agda.readthedocs.io/en/latest/language/prop.html
[R18]: https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-130.pdf
[R19]: https://fstar-lang.org/papers/mumon/
[R20]: https://www.microsoft.com/en-us/research/uploads/prod/2019/01/2019esop.pdf
[R21]: https://smt-lib.org/papers/smt-lib-reference-v2.7-r2025-07-07.pdf
[R22]: https://www.microsoft.com/en-us/research/project/z3-3/publications/
[R23]: https://theory.stanford.edu/~barrett/pubs/EMT%2B17-abstract.html
[R24]: https://cvc5.github.io/docs/cvc5-1.3.2/proofs/output_lfsc.html
[R25]: https://people.eecs.berkeley.edu/~necula/papers.html
[R26]: https://lean-lang.org/doc/reference/latest/The-Type-System/
[R27]: https://rocq-prover.org/doc/v9.0/refman/language/core/conversion.html
[R28]: https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-192.pdf
[R29]: https://lean-lang.org/doc/reference/latest

---

**Working foundation:** Ash is intended to verify claims during compilation using appropriately sandboxed ordinary Ash code for proof construction. The surrounding logical organization, evidence model, certification boundary, and syntax remain to be specified. The references supply precedents and tradeoffs for making those choices; they do not replace the need to specify and justify Ash's own logical rules.
