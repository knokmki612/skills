# Principle & Pattern Definitions

A lookup for **Step 4** of the design review. Open this to **verify the meaning of a
principle or pattern you are already considering** — after a real problem and its
concrete pain are fixed. Do **not** read it top-to-bottom to shop for a principle to
apply; that is the exact failure mode the skill guards against.

Each entry is a pointer, not a transcription: the one-line accurate definition, the
point it is most often misread on, and where to confirm it. Comprehensiveness is
delegated to the sources below — when in doubt, read the primary source, not memory.

## Trusted sources

| Source | Good for | Note |
| --- | --- | --- |
| **Refactoring.Guru** — <https://refactoring.guru> (JP: <https://refactoring.guru/ja>) | GoF patterns, refactorings, code smells | Each pattern page has a **"Applicability" / "Problems? / when not to use"** section — read *that* part, it aligns with this skill's discipline. |
| **martinfowler.com/bliki** — <https://martinfowler.com/bliki> | Nuanced takes on YAGNI, coupling/cohesion, refactoring, "is this a pattern worth it" | Fowler writes about *tradeoffs*, not rules. Best for "should I even do this". |
| **Robert C. Martin (Uncle Bob)** — <https://blog.cleancoder.com> | SOLID, the original intent behind each letter | Go here before quoting SOLID — the popular one-liners distort several of them (see below). |
| **GoF book** — *Design Patterns*, Gamma/Helm/Johnson/Vlissides (1994) | The authoritative pattern catalog + each pattern's stated *intent* and *consequences* | Every pattern lists **Consequences** (costs). A pattern chosen without reading its Consequences is a red flag. |

For a *solution approach* you don't already have a name for, web search is fine.
For a *definition*, prefer the sources above over recollection.

## Principles

### SOLID — five separate principles, often over-applied as one

Coined by Michael Feathers; articulated by Robert C. Martin. The frequent misuse is
treating each as "always split more / abstract more". They are about *managing
change*, not maximizing decomposition.

- **SRP — Single Responsibility.** "A module should have one reason to change,"
  where a *reason* means **one group of stakeholders / one axis of change** — *not*
  "a class should do one thing." Misread: splitting by verb count. Correct lens:
  would two different stakeholders demand changes here for unrelated reasons? See
  also Fowler, <https://martinfowler.com/bliki/BeckDesignRules.html>.
- **OCP — Open/Closed.** Open for extension, closed for modification — you can add
  behavior without editing existing, tested code. Misread: "add an interface/plugin
  point everywhere just in case." OCP pays off only where a *real* axis of variation
  is already changing (Step 3). Speculative OCP is YAGNI.
- **LSP — Liskov Substitution.** A subtype must be usable anywhere its base type is,
  without surprising the caller (no strengthened preconditions, weakened
  postconditions, or new exceptions). Misread: reduced to "inheritance good." It is a
  *constraint that flags bad inheritance* — a rectangle/square that breaks callers is
  an LSP violation, i.e. a signal to prefer composition.
- **ISP — Interface Segregation.** No client should be forced to depend on methods it
  does not use. Misread: "make every interface tiny." The driver is a *real* client
  being coupled to irrelevant methods, not interface size for its own sake.
- **DIP — Dependency Inversion.** High-level policy should not depend on low-level
  detail; both depend on an abstraction owned by the policy side. Misread: "wrap
  every dependency in an interface / add a DI container." DIP matters across
  *volatile* boundaries; over the whole codebase it is ceremony.

### DRY — Don't Repeat Yourself

*The Pragmatic Programmer* (Hunt & Thomas): "Every piece of **knowledge** must have a
single, authoritative representation in the system." Misread as "no two lines of code
may look alike." It is about **duplicated knowledge/decisions**, not coincidentally
similar text. Two rules that happen to compute the same today but change for different
reasons are **not** a DRY violation — merging them creates false coupling. The
counter-force is WET / "rule of three": tolerate duplication until the shared *concept*
is proven. See Fowler on <https://martinfowler.com/bliki/BeckDesignRules.html> and DRY
critiques on the same site.

### YAGNI — You Aren't Gonna Need It

Fowler, <https://martinfowler.com/bliki/Yagni.html>: build capability when a real,
present need exists, not on speculation. Key subtlety Fowler stresses: YAGNI is about
**presumptive features / speculative generality**, not an excuse to skip good design
or write careless code. Refactoring later is cheap *because* the code was clean, not
in spite of it. Use YAGNI in Step 2 to demote hypothetical problems.

### KISS — Keep It Simple

No canonical primary text (attributed to Kelly Johnson). Treat as a tie-breaker, not
an argument: between two options that solve the same Step 3 pain, prefer the one a
reader can hold in their head. "Simple" = fewer moving concepts for the *reader*, not
fewer characters.

### Law of Demeter — principle of least knowledge

Lieberherr et al. (Northeastern). A method should only talk to its immediate
collaborators (its own fields, its parameters, objects it creates) — not reach through
them (`a.getB().getC().doThing()`). Misread as a mechanical "count the dots" rule;
the real target is **not depending on another object's internal structure**. A fluent
builder with many dots is not a violation. Reference:
<https://www2.ccs.neu.edu/research/demeter/>.

### Coupling & Cohesion — the vocabulary under most "structure" arguments

Prefer **low coupling** (few, narrow, stable dependencies between modules) and **high
cohesion** (a module's parts belong together, serving one purpose). Misread as
absolutes — *zero* coupling means nothing talks to anything. The goal is to put the
coupling where change *doesn't* cross module lines. Fowler, "Reducing Coupling":
<https://martinfowler.com/ieeeSoftware/coupling.pdf>. This is usually the most honest
way to name a Step 3 pain ("these two change together but live apart") without
invoking a heavier principle.

### Composition over inheritance

Prefer assembling behavior from parts over deep type hierarchies. Not "inheritance is
bad" — inheritance is right for genuine *is-a* with substitutability (see LSP). The
guidance bites when inheritance is used for *code reuse* between things that are not
truly substitutable. GoF states this as a core design tenet.

## Patterns (GoF and friends)

Do not transcribe pattern mechanics here — read the source. When considering a
pattern, the discipline is:

1. Open its page on **Refactoring.Guru** or its entry in the **GoF book**.
2. Read the **Intent** — does it match your Step 3 pain, in your words?
3. Read **Applicability / "when not to use"** (Refactoring.Guru) or **Consequences**
   (GoF). Every pattern trades simplicity for some flexibility. If you can't name the
   cost, you're not ready to adopt it.
4. Compare against the simplest alternative (per Step 4) — a function, a map/table, a
   conditional, a parameter. Many pattern needs dissolve into "just pass a function."

Most-misapplied, with the tell:

- **Singleton** — global state with a nicer name; makes testing and reasoning harder.
  Usually the wrong answer to "I need one of these"; pass the instance instead.
- **Factory / Abstract Factory** — justified when construction genuinely varies across
  a real axis. Tell of misuse: a factory that only ever builds one concrete type.
- **Strategy** — swapping an algorithm at runtime. In languages with first-class
  functions, a plain function parameter is often the same thing with less ceremony.
- **Observer** — decoupled event notification; watch for hard-to-trace control flow
  and lifetime/leak issues. A direct call is simpler when there is exactly one
  listener.
- **Decorator / Adapter / Facade** — legitimate and cheap when the wrapped boundary is
  real; misused as pre-emptive wrapping of code you own and can just change.

The pattern is the tool, the Step 3 pain is the job. If the job is unclear, no pattern
is the right one.
