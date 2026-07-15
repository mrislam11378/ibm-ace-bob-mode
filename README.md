# ACE Developer Mode — Test Generation Rules

Rules powering the **ACE Developer** Bob mode for IBM App Connect Enterprise v13.

---

## Why Testing ACE Flows Is Important — and Why It Rarely Happens

Every ACE application should have a test project. In practice, almost none do.
Three reasons compound to make this the norm rather than the exception.

**1. The IIB legacy.**
IBM Integration Bus had no test framework. Testing meant deploying to an environment
and sending real messages — automated testing simply wasn't viable. ACE v13 changed
that, but the culture and habits formed over years of IIB never caught up. Most teams
are still working the same way: deploy and hope.

**2. We are almost always migrating legacy code.**
The most common engagement is migration — IIB → ACE, on-premises → OpenShift/cloud.
Without automated tests, validating that a migration hasn't broken anything is slow,
manual, and low-confidence. Recorded-message tests solve this directly: they capture
real production behaviour and run in seconds, turning a nerve-wracking go-live into a
verifiable one.

**3. Writing tests takes time and skills that aren't free.**
Even with ACE v13 tooling, a complete test suite requires Java/JUnit knowledge, fluency
with the Integration Test Library API, and hard-won experience with failure patterns
like BIP2331E, fan-out propagation counts, and SSL config. Getting there takes days —
days that most engagement teams would rather spend delivering business value.

This is the gap the ACE Developer Bob mode is designed to close.

---

## The Problem with the Alternatives

### `ibmint generate tests` alone

Produces a raw baseline, not a finished test suite:

- Missing `.project`, `.classpath`, `.settings/` — cannot import into the Toolkit without creating them manually.
- Time-varying fields (`createdDate`, HTTP `Date`, CDN headers) cause immediate failures — tests never pass on first run.
- No README, no coverage analysis, no guidance on untestable nodes.
- Fan-out nodes generate tests that fail with `propagateCount=0` — silent and confusing without context.
- **Node-level tests only** — `ibmint` tests each node in isolation. They do not test message flow-level behaviour: fan-out, fan-in, aggregation, routing decisions, and sequential propagation patterns are invisible at the node level. A flow can pass all node-level tests and still behave incorrectly end-to-end.

**Cost: 1–3 hours** to make it compile and understand why tests fail before writing anything new.

### Writing WholeFlow tests manually

- Requires knowing which of 6+ patterns to apply (routing, content, error path, directory scan, expected failure, stub bypass) — none consistently documented in one place.
- Common mistakes (BIP2331E path notation, missing `setStopAtInputTerminal` causing real MQ writes, `@ParameterizedTest` vs `@Test`) only surface at runtime.

**Cost: half a day to a full day** per flow for an experienced ACE developer.

---

## What Bob Adds

The ACE Developer mode wraps the full test lifecycle — from recording through running
tests — in a single guided workflow. The rules encode hard-won knowledge that would
otherwise require reading multiple IBM documentation pages, making mistakes, and
iterating.

### 1. Immediate, runnable output

Bob generates the complete project — not just the Java class. It creates `.project`,
`.classpath`, `.settings/`, `testproject.descriptor`, `build.gradle`, `settings.gradle`,
and `README.md` in a single pass. The output imports into the ACE Toolkit with zero
manual steps and compiles without errors.

**Time saved: 30 min** per project.

### 2. Tests that actually pass on first run

Bob automatically identifies time-varying fields from the recorded messages and adds
`ignorePath()` calls for common offenders (`createdDate`, `modifiedDate`, HTTP `Date`,
CDN headers, `WrittenDestination`). Generated tests that would fail immediately due to
timestamp drift are fixed before they are ever run.

**Time saved: 1 - 2 Hours** of debugging per flow.

### 3. Coverage analysis built in

Bob reads the `.msgflow` files alongside the `ibmint generate msgindex` output and
produces a coverage report as part of the README:

- Every node in the flow is listed and matched against recorded checkpoints.
- Nodes with no recorded output are flagged with the reason (error terminal never
  triggered, node on unrecorded path, input/terminal node not directly testable).
- Missing scenarios (failure paths, alternate branches) are called out explicitly with
  instructions for capturing them.

This is coverage analysis that `ibmint` does not provide at all and that a developer
would otherwise have to derive manually by cross-referencing the `.msgflow` XML against
the msgindex output — a task that takes **30–60 minutes** per flow and is easy to get wrong.

**Time saved: 30–60 minutes** per application.

### 4. Correct pattern selection, automatically

Bob knows which test pattern to apply for each node type:

| Node type | Pattern Bob selects |
|---|---|
| Compute / Mapping | Node-level unit test, `equalsMessage` |
| HTTP Request (outbound) | `equalsMessage` + `ignorePath` for live fields, or `setStopAtInputTerminal` for offline |
| Fan-out (Flow Order) | Comment block explaining the propagation count issue + Mermaid diagram in README |
| Routing flow | `@CsvSource` routing test with `terminalReceiveCountIs` on NodeStubs |
| Error path | `error_path_catch_terminal` variant with catch terminal assertion |
| Expected flow failure | `assertThrows(TestException.class)` + `ex.getMessageNumber()` |

Without Bob, selecting the right pattern requires reading multiple documentation pages
and understanding the difference between `NodeSpy.evaluate()` and `NodeSpy.propagate()`,
`terminalReceiveCountIs` vs `terminalPropagateCountIs`, and when to use `NodeStub` vs
`setStopAtInputTerminal`. These distinctions are not documented in a single place anywhere
in IBM's official documentation.

### 5. Path notation errors eliminated

BIP2331E (`Field '/message/JSON/Data/...' does not exist`) is one of the most common
and confusing test failures. It is caused by using XPath slash notation with `messagePath()`
instead of dot notation. Bob knows the rule: `messagePath()` uses dot notation;
`ignorePath()` inside `equalsMessage()` uses XPath slash notation. This distinction
is not documented in IBM's Knowledge Center.

**Time saved: 20–60 minutes** of debugging.

### 6. Living README with Mermaid diagrams

Every test project gets a README that documents the flow structure (Mermaid diagram
from the actual `.msgflow`), recorded sessions, coverage gaps, known limitations, and
re-generation instructions. Fan-out propagation failures are explained visually in the
README rather than as cryptic inline comments.

---

## Time Savings Summary

| Task | Without Bob | With Bob |
|---|---|---|
| Scaffold test project (post `ibmint`) | 1–2 hours | ~30 minutes |
| Fix timestamp/ignorePath failures | 1–2 hours | Automatic |
| Coverage gap analysis | 30–60 min per flow | ~2 minutes |
| Select correct test pattern | 30–60 min research | Immediate |
| Write routing test class | 1–2 hours | ~10 minutes |
| Debug path notation | 20–60 min | Prevented |
| Write README + diagrams | 30–60 min | ~5 minutes |
| **Total per flow** | **4–8 hours** | **~30 minutes** |

For a project with 5 applications, this is the difference between a week of test work and
a single afternoon.

---

## Future Direction

- **Coverage scoring** — numeric score (nodes covered / testable nodes) as a CI gate, not just a README entry.
- **Test quality analysis** — flag tests using `equalsMessage` without `ignorePath` on volatile fields, `@Disabled` without a linked issue, or flows with no error-path coverage.
- **CI/CD pipeline generation** — Bob already knows `ibmint deploy` and `IntegrationServer --test-project`; extend to generate a full pipeline script (GitHub Actions / Azure DevOps / Jenkins).
- **Multi-flow dependency awareness** — map Callable flow dependencies and generate cross-flow integration tests with NodeStub isolation.
- **Mutation testing guidance** — once a baseline exists, identify high-risk ESQL nodes and suggest mutations to verify the tests would catch a regression. Particularly valuable for IIB → ACE migrations.

---

*Rules maintained in `.bob/rules-ace-developer/`. To contribute improvements, follow
the pattern established in the existing rule files: templated examples, no customer
data, cross-references between files using the section/file name convention.*
