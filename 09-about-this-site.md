# About this site

This site is a concept-level field guide to the n8n codebase. It explains how the execution engine behaves, why the main subsystems exist, and how the pieces fit together at runtime.

The guide exists for engineers who run, debug, extend, or embed n8n. It is written for node authors, self-hosters, and anyone investigating execution order, partial execution, expressions, webhooks, or workflows that survive across process boundaries. It aims to give a working mental model of the system, not a product walkthrough or an API reference.

Doc Holiday wrote this page from direct exploration of the n8n source repository. The guide records the codebase as it existed at the time of generation and keeps the placeholder token `[GENERATED_FROM: commit SHORT_SHA, DATE]` exactly as shown.

## Scope

This guide complements the official n8n documentation and does not replace it. The authoritative product documentation remains at https://docs.n8n.io.

n8n is fair-code licensed under the Sustainable Use License, with enterprise-licensed components in `.ee` areas of the repository. This guide stays independent from n8n and from any commercial offering tied to it.

The codebase changes quickly, so this guide reflects a snapshot rather than a permanent description. It should help readers understand the current shape of the system, but it may lag behind active development. For project-specific contact or repository details, use `[CONTACT_OR_REPO_LINK]`.

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