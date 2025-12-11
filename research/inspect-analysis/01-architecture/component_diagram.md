```mermaid
flowchart LR
  Entry[CLI / Python API<br/>inspect eval / eval_set / eval()] --> Runner[Eval runner]
  Runner --> Task[Task (dataset + solver + scorer)]
  Task --> Dataset[Dataset<br/>CSV/JSON/JSONL/HF/Mem]
  Task --> SolverChain[Solver / Agent chain]
  Task --> Scorer[Scorer + metrics]
  SolverChain -->|generate()| ModelAPI[Model abstraction<br/>provider/model]
  ModelAPI --> Providers[Providers<br/>OpenAI/Anthropic/Google/Mistral/...]
  SolverChain --> Tools[Tools & MCP]
  Tools --> Sandbox[Sandbox env<br/>local/docker/custom]
  Scorer --> Metrics[Metrics / reducers]
  {Scorer,SolverChain,ModelAPI,Tools} --> Events[Events & token usage]
  Events --> Logs[EvalLog (.eval/.json)]
  Metrics --> Logs
  Logs --> Exports[CLI/API export<br/>JSON/CSV/S3/Azure]
  Runner --> Hooks[Hooks & storage extensions]
  Hooks --> Logs
```

- Tasks bind datasets, solver pipelines, and scorer/metric definitions.
- Solvers operate on `TaskState` and call the provider-agnostic `ModelAPI`; tools run inside sandboxes with optional approval policies.
- Scorers turn model output + targets into `Score` objects and metrics, which are aggregated into EvalLogs.
- Eval logs capture plan, samples, transcripts, scores, usage, and can be streamed, viewed, or exported.
