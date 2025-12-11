Scorers convert model output plus the target(s) into `Score` objects and emit metrics.

- **Score structure:** `value` (e.g., "C"/"I"/numeric), optional `answer` extraction, optional `explanation`, and `metadata`. Scores are stored per-scorer in logs. Reducers combine epochs.
- **Built-ins:** `includes`, `match`, `pattern`, `answer`, `exact`, `f1`, `choice` (MC), `model_graded_qa`, `model_graded_fact`. Each provides metrics `accuracy` + `stderr`; additional metrics include `mean`, `var/std/stderr/bootstrap_stderr`, `grouped`, reducers (`mean_score`, `median_score`, `max_score`, `at_least`, `pass_at`, etc.).
- **Model-graded scorers:** prompt a grader model (default role `grader`; falls back to eval model). Support majority vote across multiple grader models, partial credit, custom templates, and history inclusion.
- **Custom scorers:** Decorate with `@scorer(metrics=[...])` and implement `async def score(state: TaskState, target: Target) -> Score`. Use `multi_scorer` and reducers for ensembles. Scorers see full `TaskState` including metadata, messages, and model outputs.
- **Workflow:** Scorers run after solver completion per sample. Metrics aggregate across samples (and epochs when used). Results are written into `EvalLog.results` with scorer params and metric metadata.
