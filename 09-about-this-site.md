# About this site

This site is a concept-level field guide to the n8n codebase. It explains runtime behavior, the purpose of the main subsystems, and the relationships between the pieces that cooperate during execution. The goal is a shared mental model, not a product walkthrough or an API reference.

The intended audience includes engineers who run, debug, extend, or embed n8n. It also speaks to node authors and self-hosters who need to reason about execution order, partial execution, expressions, webhooks, or workflows that keep running across process boundaries. Each page focuses on the ideas that matter when reading, discussing, or changing the system.

Doc Holiday wrote this guide by exploring the n8n source repository directly. Every page stays grounded in actual code, with real file paths and symbol names, as of `[GENERATED_FROM: commit SHORT_SHA, DATE]`. The guide presents one dated snapshot of an actively developed codebase, so migration-state details appear as observations at that point in time.

## Scope

This guide complements the official n8n documentation and does not replace it. The authoritative product documentation remains at https://docs.n8n.io.

n8n is fair-code licensed under the Sustainable Use License, with enterprise-licensed components in `.ee` areas of the repository. This guide is an independent companion to n8n and to any commercial offering tied to it.

The codebase changes quickly, so this guide reflects a snapshot rather than a permanent description. It can help readers understand the current shape of the system, but it may lag behind active development. Corrections and updates are welcome at `[CONTACT_OR_REPO_LINK]`.

## Table of contents

- [/00-the-big-picture.md](/00-the-big-picture.md) — A map of the whole platform and its major moving parts.
- [/01-anatomy-of-an-execution.md](/01-anatomy-of-an-execution.md) — An end-to-end trace of one workflow execution.
- [/02-how-the-engine-decides-what-runs-next.md](/02-how-the-engine-decides-what-runs-next.md) — The scheduling algorithm, including the node stack, v0 and v1 ordering, joins, and dead branches.
- [/03-the-canvas-is-not-the-execution.md](/03-the-canvas-is-not-the-execution.md) — Where the visual model and the engine model disagree.
- [/04-partial-executions-and-dirty-nodes.md](/04-partial-executions-and-dirty-nodes.md) — What execute step really does through the DirectedGraph machinery.
- [/05-items-runs-and-paireditem.md](/05-items-runs-and-paireditem.md) — The data model of a run: items, runs, and item lineage.
- [/06-expressions-and-user-code.md](/06-expressions-and-user-code.md) — How `{{ }}` expressions and Code nodes evaluate in practice.
- [/07-triggers-webhooks-and-activation.md](/07-triggers-webhooks-and-activation.md) — How workflows start, from editor tests to production activation.
- [/08-one-execution-many-processes.md](/08-one-execution-many-processes.md) — Queue mode, wait and resume behavior, and executions that outlive a process.