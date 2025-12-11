```mermaid
graph LR
  subgraph Frontend
    FE[Clients (React + TypeScript + MUI)]
  end

  subgraph Backend API
    BE[Servers (Node/Express API)]
    Worker[Servers Worker Jobs]
  end

  subgraph Eval Layer
    EvalServer[EvalServer (FastAPI + DeepEval runner)]
    BiasAPI[BiasAndFairnessServers (FastAPI)]
    BiasModule[BiasAndFairnessModule (Python evaluation engine & metrics)]
  end

  subgraph Data
    PG[(PostgreSQL)]
    Redis[(Redis)]
    Artifacts[(Artifacts: reports/results CSV/JSON)]
  end

  FE -->|REST/Web| BE
  BE -->|DB| PG
  BE -->|Caching/queues| Redis
  BE -->|Delegates eval jobs| EvalServer
  EvalServer -->|Writes logs/metrics| PG
  EvalServer -->|Model inference & metrics| BiasModule
  EvalServer -->|Fairness tasks| BiasAPI
  BiasAPI -->|DB| PG
  BiasAPI -->|Caching| Redis
  EvalServer --> Artifacts
  Worker -->|Async jobs| PG
  Worker -->|Cache| Redis
  FE -->|Fetch artifacts/reports| BE
```
