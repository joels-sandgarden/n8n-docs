# How the engine decides what runs next

The public execution order docs describe the visible rule: workflows created before version 1 run branch nodes in lockstep, while version 1 and later finish one branch before starting another and order siblings from top to bottom, then left to right. `WorkflowExecute` turns that rule into one work list, and this page explains how that list decides branch order, joins, dead ends, loops, and failures.

`nodeExecutionStack` gives the engine one work list to manage. `processRunExecutionData` consumes it from the front with `shift()`, and `addNodeToBeExecuted` adds new work to the front with `unshift` or to the back with `push` depending on `workflow.settings.executionOrder`.

That toggle controls scheduling only. `IRunExecutionData.version` in `packages/workflow/src/run-execution-data/run-execution-data.ts` belongs to run data migration and does not change the execution order rule.

## One work list, two enqueue directions

When `workflow.settings.executionOrder` equals `v1`, `nodeExecutionStack` behaves like a LIFO stack: `processRunExecutionData` always takes from the front, and `addNodeToBeExecuted` uses `unshift` so the most recently discovered child is next. When the setting is legacy `v0`, the same front-of-list consumption combines with `push`, so the work list behaves like a FIFO queue and advances all branches one node at a time. That single choice, `unshift` versus `push`, is the branch-ordering switch.

Because v1 inserts children in canvas order before they reach the front of the list, the topmost sibling wins, then the leftmost one. Moving a node on the canvas can therefore change execution order. See [The canvas is not the execution](03-the-canvas-is-not-the-execution.md) for the broader canvas model.

```mermaid
flowchart TD
    "Take front entry" --> "Run node"
    "Run node" --> "Output empty?"
    "Output empty?" -- "yes" --> "End branch"
    "Output empty?" -- "no" --> "Has multiple inputs?"
    "Has multiple inputs?" -- "yes" --> "Park in waitingExecution"
    "Has multiple inputs?" -- "no" --> "Order children by canvas position"
    "Order children by canvas position" --> "enqueue with unshift in v1"
    "Order children by canvas position" --> "enqueue with push in v0"
    "Park in waitingExecution" --> "waiting sweep when stack is empty"
    "waiting sweep when stack is empty" --> "requiredInputs"

## Legacy `v0` keeps FIFO behavior

Workflows created before the execution order change can still run with the legacy `v0` setting. That path keeps FIFO enqueueing with `push`, and it can force upstream parent chains by injecting empty data entries when necessary.

## Joins wait for complete input

Multi input nodes do not run as soon as one wire arrives. The engine parks them in `waitingExecution` and `waitingExecutionSource` until every input arrives, then promotes them back to the stack. At the end of the loop, if the stack runs dry, the scheduler sweeps the waiting set again. In version 1, it checks `requiredInputs` before promotion, so a node runs only when the inputs it truly requires are present. That sweep checks readiness; it does not create a second branch order system.

## Dead branches end when output stays empty

A branch ends when a node returns zero output items. The scheduler does not enqueue downstream nodes in that case unless `alwaysOutputData` or related empty input behavior fabricates an empty item. In practice, a missing branch usually means an upstream node returned zero items.

## Cycles and looping nodes still go through the same list

The engine itself has no loop construct. A cycle on the canvas only causes a node to reenter the work list later with a new run index. `SplitInBatches` owns its loop by storing state in `getContext('node')` and alternating between `loop` and `done`. The engine still stops accidental repetition when the same `node:runIndex` appears twice in a row.

## Failures change what gets scheduled next

Retry on fail gives a node a bounded number of attempts with a wait between tries. `continueOnFail`, `onError`, and error output routing decide whether the run keeps moving and which output the scheduler enqueues next. A failure can therefore steer the work list just like a successful branch can.

## Where to look in the code

- `packages/core/src/execution-engine/workflow-execute.ts` — `processRunExecutionData` consumes `nodeExecutionStack`, `addNodeToBeExecuted` chooses `unshift` or `push`, and the waiting sweep rechecks `requiredInputs`.
- `packages/core/src/execution-engine/workflow-execute.ts` — `prepareConnectionInputData` and `runNode` show how the next node gets its inputs, retries, and error routing.
- `packages/nodes-base/nodes/SplitInBatches/v3/SplitInBatchesV3.node.ts` — the loop node keeps state in node context and switches between `loop` and `done`.
- `packages/workflow/src/run-execution-data/run-execution-data.ts` — run data versioning and migration stay separate from `workflow.settings.executionOrder`.