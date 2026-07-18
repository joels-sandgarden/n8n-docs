# Partial Executions and Dirty Nodes

## Overview

Partial execution lets the workflow engine run only the portion of a workflow that changed. It matters because the engine can reuse valid results from earlier runs instead of replaying every node every time.

Dirty nodes support that behavior. A dirty node marks a part of the workflow whose stored result no longer matches the current upstream state, so the engine knows which downstream work it must rebuild.

## Mental model

The execution engine treats a workflow as a graph of dependent nodes. When the engine receives a partial execution request, it starts from the selected entry point, follows the dependency graph, and identifies the smallest set of nodes that must run.

Nodes outside that path remain clean. Their stored output stays usable because nothing on the path to those nodes changed.

Nodes on the affected path become dirty when upstream data changes, when a branch stops early, or when the selected execution slice does not include the work that originally produced them. Dirty status does not mean the node failed. It means the stored result no longer represents the current workflow state.

## Why dirty nodes exist

Dirty nodes prevent partial execution from mixing fresh and stale results. Without that distinction, a rerun could reuse output that depends on data the engine has already replaced.

The dirty marker gives the engine a simple rule: reuse clean results, recompute dirty ones, and keep the boundary between them explicit. That rule keeps partial runs predictable even when a workflow contains long chains or multiple branches.

## How partial execution flows through the engine

`WorkflowExecute.runPartialWorkflow2` drives the current partial execution path. The method coordinates the work of the partial execution helpers in `packages/core/src/execution-engine/partial-execution-utils/` and uses them to decide what to execute next.

The helpers identify the affected portion of the workflow, track the nodes that depend on changed data, and prepare the execution data that the engine needs for the rerun. The main engine then executes only that slice and leaves the rest of the graph intact.

This design keeps partial execution inside the normal workflow engine. The same execution model still applies; partial execution only narrows the part of the graph that the engine considers active.

## Design decisions

### Reuse the main execution engine

The partial execution path stays inside `WorkflowExecute` instead of branching into a separate runtime. That choice keeps the partial run aligned with the same workflow semantics that full executions use.

The consequence is consistency. Partial runs and full runs share the same rules for node order, data flow, and execution data.

### Mark state instead of guessing

The engine marks dirty nodes rather than trying to infer freshness from timing or cache presence alone. That choice makes dependency changes visible and keeps the reuse boundary explicit.

The consequence is safer reruns. The engine can preserve valid output while still forcing any dependent work to run again.

### Limit work to the affected slice

The partial execution utilities focus on the part of the graph that changed. They do not rebuild the entire workflow when a smaller slice is enough.

The consequence is lower runtime cost and faster feedback for workflows that only need a narrow rerun.

## Relationship to the rest of the system

Partial execution sits between workflow state and the execution engine. It consumes the workflow graph, the current execution data, and the selected entry point, then returns the subset of work that still needs to run.

The rest of the engine consumes that result as normal execution input. Nothing else in the workflow path needs to know whether the run started as a full execution or a partial one; dirty node tracking already records the boundary.

## Where to look in the code

- `packages/core/src/execution-engine/workflow-execute.ts` — the main execution entry point and the `runPartialWorkflow2` flow.
- `packages/core/src/execution-engine/partial-execution-utils/` — helpers that identify the affected slice and prepare partial execution state.
- `packages/core/src/execution-engine/` — related execution engine code that shows how partial runs fit into the broader runtime.
