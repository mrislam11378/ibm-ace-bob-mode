# ACE Developer Mode — Test Generation & Migration Analysis

> **⚡ ace-developer mode** is a specialized fork and enhancement of the open-source [ibm-ace-bob-mode](https://github.com/ibm-self-serve-assets/ibm-ace-bob-mode), which builds directly on the [ace-bob skill](https://github.com/ot4i/ace-bob) — IBM's authoritative ACE Toolkit reference for node types, xmi namespaces, and project structures. While that foundation provides the core ACE vocabulary, this work extends it with enterprise migration patterns, test automation intelligence, and coverage analysis capabilities encoded into a 14-file structured XML rule system.

---

## Why Testing ACE Flows Is Critical — and Why It Rarely Happens

Integration solutions are the nervous system of the enterprise, handling mission-critical transactions across App Connect Enterprise, legacy IIB estates, and hybrid cloud environments. Yet almost none of these flows have automated tests. Three compounding reasons explain why.

**1. The IIB legacy.**
IBM Integration Bus had no test framework. Testing meant deploying to an environment and sending real messages — automated testing simply wasn't viable. ACE v12 introduced `ibmint generate tests`, JUnit 5, NodeSpy, and NodeStub APIs, carried forward into v13. But the culture and habits formed over years of IIB never caught up. Most teams are still working the same way: deploy and hope.

**2. Migration is almost always the context.**
The most common engagement is migration — IIB → ACE, on-premises → OpenShift/cloud. Without automated tests, validating that a migration hasn't broken anything is slow, manual, and low-confidence. Legacy codebases are often a decade or more old. The architects who designed them have retired or moved on. When IBM is brought in to migrate, there is frequently no documentation, no test data, and no institutional memory of what the flows are supposed to do. Understanding the codebase from first principles — tracing every path, inferring every business rule — is slow, expensive, and almost always incomplete.

**3. Writing tests takes time and skills that aren't free.**
Even with ACE v13 tooling, a complete test suite requires Java/JUnit knowledge, fluency with the Integration Test Library API, and hard-won experience with failure patterns like BIP2331E, fan-out propagation counts, and SSL config. Done manually, one flow takes 4–8 hours. Across a 50+ flow estate, that is weeks of work before a single assertion runs. The consequence is predictable: testing is deferred, migrations ship without a regression baseline, and production incidents become the de facto quality gate.

This is the gap the ACE Developer mode is designed to close.

---

## The Problem with Native Tooling Alone

### `ibmint generate tests` in isolation

Produces a raw baseline, not a finished test suite:

- Missing `.project`, `.classpath`, `.settings/` — cannot import into the Toolkit without creating them manually.
- Time-varying fields (`createdDate`, HTTP `Date`, CDN headers) cause immediate failures — tests never pass on first run.
- No README, no coverage analysis, no guidance on untestable nodes.
- Fan-out nodes generate tests that fail with `propagateCount=0` — silent and confusing without context.
- **Node-level tests only** — `ibmint` tests each node in isolation, producing an average of **1,200 lines of fragile, tightly-coupled boilerplate** per flow. Fan-out, fan-in, aggregation, routing decisions, and sequential propagation patterns are invisible at the node level. A flow can pass all node-level tests and still behave incorrectly end-to-end. When customers realize they must maintain 1,200–3,000 lines of JUnit code per flow just to support a future migration, the maintenance shock is real.

**Cost: 1–3 hours** to make it compile and understand why tests fail before writing anything new.

### Writing WholeFlow tests manually

- Requires knowing which of 6+ patterns to apply (routing, content, error path, directory scan, expected failure, stub bypass) — none consistently documented in one place.
- Common mistakes (BIP2331E path notation, missing `setStopAtInputTerminal` causing real MQ writes, `@ParameterizedTest` vs `@Test`) only surface at runtime.

**Cost: half a day to a full day** per flow for an experienced ACE developer.

---

## The ace-developer Mode: Architecture-Aware Runtime Analysis

The ace-developer mode is implemented as a **custom Bob subagent mode** backed by a 14-file structured XML rule system. It attacks both the knowledge gap and the test generation cost through two core capabilities.

### Pillar 1 — Execution Path Discovery via Recorded Message Analysis

To resolve the legacy knowledge gap typical of decades-old integration estates, the agent drives discovery through empirical runtime analysis. The pipeline programmatically ingests raw recorded message assemblies (`.mxml`) and correlates their runtime data payloads directly against the logical geometry of target message flow (`.msgflow`) XML documents.

By parsing the output of `ibmint generate msgindex`, the agent extracts and maps `messageId-propagationCount` pairs directly to flow nodes. This analysis instantly:

- Maps active execution routes across nested subflows, ESQL statements, Java Compute code, and graphical mapping nodes
- Uncovers undocumented conditional logic and hidden edge-case paths
- Flags untested path gaps — nodes with no recorded checkpoint are surfaced explicitly

Instead of spending weeks reverse-engineering complex multi-technology flow configurations from first principles, IBM consultants and customer teams get an immediate, data-driven blueprint of actual system behaviour within minutes. Gaps are explicit, prioritised, and actionable. This is a direct replacement for weeks of manual code archaeology.

### Pillar 2 — Automated Whole-Flow JUnit Test Generation

From those same recorded messages, the agent generates a complete, runnable JUnit 5 test project. The generation pipeline shifts the paradigm from micro-assertions to **parameterized whole-flow integration testing**, compressing output from an average of 1,200 lines to a clean **200-line test suite** — a verified **83% reduction in code volume** — through five automated phases:

1. **Scaffold Generation** — provisions the complete test project structure: `.project` with correct Eclipse natures (testProjectNature, javanature, jcnnature, barnature), `.classpath` with JUnit 5 and the Integration Test Library, `testproject.descriptor`, `build.gradle` using `MQSI_BASE_FILEPATH`, `settings.gradle`. The output imports into the ACE Toolkit with zero manual steps and compiles without errors.

2. **Dynamic Mocking & Stub-Injection** — evaluates heterogeneous flow boundaries and leverages native NodeSpy and NodeStub APIs to inject strategic `.withStub()` hooks across Compute, Java Compute, and Mapping nodes. This completely detaches external physical backends (mainframe MQ queues, live databases) so tests run locally inside containerized CI/CD pipelines in milliseconds — including nodes buried several levels deep inside nested subflows.

3. **Environment Isolation** — automatically embeds mandatory `@AfterEach` teardown blocks (`TestSetup.restoreAllMocks()`) to ensure clean, fully isolated execution state between test runs.

4. **Intelligent Assertions & Masking** — detects volatile elements (timestamps, HTTP Date headers, CDN tokens, `WrittenDestination` paths) via pattern matching and automatically chains `.ignorePath()` suppressions. Tests that would fail immediately due to timestamp drift are fixed before they are ever run. The notation distinction (`messagePath()` uses dot notation; `ignorePath()` inside `equalsMessage()` uses XPath slash notation) is enforced proactively, eliminating BIP2331E errors entirely.

5. **Empirical Documentation** — outputs a README with a generated Mermaid flow diagram of the subflow topology, a recorded sessions table mapping messageIds to checkpoints, and a path coverage analysis table showing nodes covered vs untestable with explicit reasons.

---

## Correct Pattern Selection, Automatically

Bob knows which test pattern to apply for each node type, eliminating the research cost that would otherwise require reading multiple IBM documentation pages:

| Node type | Pattern Bob selects |
|---|---|
| Compute / Mapping | Node-level unit test, `equalsMessage` |
| HTTP Request (outbound) | `equalsMessage` + `ignorePath` for live fields, or `setStopAtInputTerminal` for offline |
| Fan-out (Flow Order) | Parameterized whole-flow test; README explains propagation count behaviour with Mermaid diagram |
| Routing flow | `@CsvSource` routing test with `terminalReceiveCountIs` on NodeStubs |
| Error path | `error_path_catch_terminal` variant with catch terminal assertion |
| Expected flow failure | `assertThrows(TestException.class)` + `ex.getMessageNumber()` |

The distinctions between `NodeSpy.evaluate()` and `NodeSpy.propagate()`, `terminalReceiveCountIs` vs `terminalPropagateCountIs`, and `NodeStub` vs `setStopAtInputTerminal` are not documented in a single place anywhere in IBM's official documentation. Bob knows all of them.

---

## Empirical Benchmark

Validated in a head-to-head comparison on a live enterprise application using identical recorded message assemblies:

| Approach | Lines of Code |
|---|---|
| Traditional node-level — native `ibmint generate tests` | **1,200 lines** |
| ace-developer mode — parameterized whole-flow suite | **200 lines** |

**83% reduction in code volume. 10× improvement in overall test authoring velocity.**

Bob also correctly mocked a database-dependent node buried several levels deep inside nested subflows — without being told which nodes to mock. No existing tool does this automatically.

---

## Time Savings Summary

| Task | Without Bob | With Bob |
|---|---|---|
| Scaffold test project (post `ibmint`) | 1–2 hours | ~5 minutes |
| Fix timestamp / `ignorePath` failures | 1–2 hours | Automatic |
| Coverage gap analysis | 30–60 min per flow | ~2 minutes |
| Select correct test pattern | 30–60 min research | Immediate |
| Write routing test class | 1–2 hours | ~10 minutes |
| Debug path notation (BIP2331E) | 20–60 min | Prevented |
| Write README + diagrams | 30–60 min | ~5 minutes |
| **Total per flow** | **4–8 hours** | **~30 minutes** |

For a project with 5 applications, this is the difference between a week of test work and a single afternoon. Every flow ships with a regression baseline. Migrations become verifiable, not just hopeful.

---

## Future Roadmap

To scale this analysis and JUnit generation pipeline across thousands of enterprise interfaces without exhausting active LLM context windows, the roadmap establishes a split-system design using the **Model Context Protocol (MCP)**. Core behavioural guardrails remain locked in the structured rule files; resource-intensive programmatic generation tasks offload to decoupled, on-demand MCP workers.

This infrastructure enables four upcoming platform enhancements:

- **Deep Intra-Node Coverage Scoring** — shifting from basic node-level visibility to a semantic parser that computes exact component, terminal, and logic path coverage percentages. Measures line-by-line execution inside ESQL Compute blocks, compiled Java logic within Java Compute nodes, mapping rules within Graphical Data Maps, and localized topologies within nested subflows. Delivered as a numeric CI gate, not just a README entry.

- **Multi-Flow Dependency Scaffolding (Contract Testing)** — when shared libraries or subflows are reused across multiple distributed applications, the pipeline automatically generates dedicated standalone scaffolding test applications. Isolating the shared asset into an independent test project validates cross-application contract constraints centrally, preventing redundant test compilation and cascading failures when shared logic shifts.

- **Automated Mutation Testing & Integration Chaos Engineering** — a programmatic "Chaos Monkey" framework purpose-built for enterprise integration. The engine injects malicious runtime faults, payload corruptions, and simulated exception states directly into execution message assemblies and node states to validate that the generated JUnit 5 test suites accurately detect, trap, and alert on any behavioural regressions.

- **Enterprise Pattern Validation** — an automated compliance and auditing engine that continuously evaluates all message flow configurations against organizational design patterns, security baselines, and architectural standards prior to artefact compilation.

---

*Rules maintained in `.bob/rules-ace-developer/`. Built on [ibm-ace-bob-mode](https://github.com/ibm-self-serve-assets/ibm-ace-bob-mode) and the [ace-bob skill](https://github.com/ot4i/ace-bob). To contribute improvements, follow the pattern established in the existing rule files: templated examples, no customer data, cross-references between files using the section/file name convention.*
