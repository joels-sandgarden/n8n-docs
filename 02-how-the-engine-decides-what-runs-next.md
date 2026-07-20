# How the engine decides what runs next

The public execution order docs describe the visible rule: workflows created before version 1 run branch nodes in lockstep, while version 1 and later finish one branch before starting another and order siblings from top to bottom, then left to right. `WorkflowExecute` turns that rule into one work list, and this page explains how that list decides branch order, joins, dead ends, loops, and failures.

`nodeExecutionStack` gives the engine one list of pending nodes to manage. `processRunExecutionData` consumes it from the front with `shift()`, and `addNodeToBeExecuted` adds new work to the front with `unshift` or to the back with `push` depending on `workflow.settings.executionOrder`.

That toggle controls scheduling only. `IRunExecutionData.version` in `packages/workflow/src/run-execution-data/run-execution-data.ts` belongs to run data migration and does not change the execution order rule.

## One work list, two enqueue directions

When `workflow.settings.executionOrder` equals `v1`, the scheduler treats the front of the list as the next branch to finish. `processRunExecutionData` collects child nodes, sorts them by canvas position, and hands them back so the topmost child runs first. If two siblings share the same height, the leftmost one wins. Moving a node on the canvas can therefore change the next node that runs. See [The canvas is not the execution](/03-the-canvas-is-not-the-execution.md) for the broader canvas model.

```mermaid
flowchart TD
    "nodeExecutionStack" --> "shift() from the front"
    "shift() from the front" --> "Node runs"
    "Node runs" --> "Has output items?"
    "Has output items?" -- "yes" --> "Sort children by canvas position"
    "Sort children by canvas position" --> "executionOrder = v1"
    "executionOrder = v1" --> "unshift to the front"
    "executionOrder = legacy v0" --> "push to the back"
    "Has output items?" -- "no" --> "Branch ends"
    "nodeExecutionStack" --> "waiting sweep when the stack is empty"
    "waiting sweep when the stack is empty" --> "requiredInputs"
```

## Legacy `v0` keeps FIFO behavior

Workflows created before the execution order change can still run with the legacy `v0` setting. That path keeps FIFO enqueueing with `push`, and it can force upstream parent chains by injecting empty data entries when necessary.

## Joins wait for complete input

Multi input nodes do not run as soon as one wire arrives. The engine parks them in `waitingExecution` and `waitingExecutionSource` until every input arrives, then promotes them back to the stack. At the end of the loop, if the stack runs dry, the scheduler sweeps the waiting set again. In version 1, it checks `requiredInputs` before promotion, so a node runs only when the inputs it truly requires are present. That sweep checks readiness; it does not create a second branch order system.

## Dead branches end when output stays empty

A branch ends when a node returns zero output items. The scheduler does not enqueue downstream nodes in that case unless `alwaysOutputData` or related empty input behavior fabricates an empty item. In practice, a missing branch usually means an upstream node returned zero items.

## Cycles and looping nodes still go through the same list

The engine itself has no loop construct. A cycle on the canvas only causes a node to reenter the work list later with a new run index. `SplitInBatches` owns its loop by storing state in `getContext('node')` and alternating between `loop` and `done`. The engine still stops accidental repetition when the same `node:runIndex` appears twice in a row.

## Failures change what gets scheduled next

Retry on fail gives a node bounded attempts with a wait between tries. `continueOnFail`, `onError`, and error output routing decide whether the run keeps moving and which output the scheduler enqueues next. A failure can therefore steer the work list just like a successful branch can.

## Where to look in the code

- `packages/core/src/execution-engine/workflow-execute.ts` — the main scheduler loop, `shift()` consumption, `unshift` and `push`, sibling sorting, waiting nodes, `requiredInputs`, retries, and error routing.
- `packages/core/src/execution-engine/workflow-execute.ts` — `prepareConnectionInputData` and the node execution path show which input data the next node actually receives.
- `packages/nodes-base/nodes/SplitInBatches/v3/SplitInBatchesV3.node.ts` — the loop node keeps state in node context and switches from `loop` to `done`.
- `packages/workflow/src/run-execution-data/run-execution-data.ts` — run data versioning and migration stay separate from `workflow.settings.executionOrder`.
- `03-the-canvas-is-not-the-execution.md` — the neighboring concept page explains why canvas position influences order without turning the canvas into the executor.