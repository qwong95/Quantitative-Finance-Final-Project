# Quantitative-Finance-Final-Project

```mermaid
flowchart TB

  %% Collection
  A1[CDC NWSS SARS-CoV-2 CSV/API]:::coll
  A2[Simulated COVID Stream\n(JSON)]:::coll

  %% Ingestion
  K[(Kafka/Redpanda\ncovid_wastewater_stream)]:::ing

  %% Streaming pipeline
  subgraph SP[Spark Structured Streaming Pipeline]
    direction TB
    S1[Parse JSON → rows]:::proc
    S2[Dedup & QC]:::proc
    S3[Daily rollup\nsite→county]:::proc
    S4[Window features\nmean7/14, std14, z, wow]:::proc
    S5a[Rule-based severity]:::proc
    S5b[ML Inference\n• Spike classifier\n• 1-day forecaster]:::ml
  end

  %% Delta Layers
  B[(Delta Bronze)]:::store
  S[(Delta Silver)]:::store
  G[(Delta Gold)]:::store
  MLP[(Delta ML Predictions)]:::mlstore

  %% Alerts
  AL[(Kafka: severity_alerts)]:::ing
  MLAL[(Kafka: predictive_alerts)]:::ing

  %% Offline ML training
  subgraph MLTrain[Offline ML Training (Batch)]
    direction TB
    T1[Label generation\n(spike in next 7d?)]:::train
    T2[Feature assembly]:::train
    T3[GBTClassifier / GBTRegressor]:::train
    T4[Model Registry\n(GCS/S3/Local Delta)]:::train
  end

  %% Viz
  V[Tableau / Notebooks]:::viz

  %% Edges
  A1 -->|Backfill| B
  A2 --> K

  K --> S1 --> S2 --> S3 --> S4 --> S5a --> S5b

  S1 --> B
  S2 --> S
  S4 --> G
  S5b --> MLP

  S5a -->|severity>=τ| AL
  S5b -->|p>=θ or Δforecast| MLAL

  G --> T1 --> T2 --> T3 --> T4 --> S5b
  G --> V
  MLP --> V

  %% Styles
  classDef coll fill:#eef8ff,stroke:#6aa9ff;
  classDef ing fill:#fff5e6,stroke:#ffb347;
  classDef proc fill:#e8f7e8,stroke:#62b267;
  classDef train fill:#f7f2ff,stroke:#8e44ad;
  classDef store fill:#f3eefc,stroke:#9b59b6;
  classDef mlstore fill:#e6e0fa,stroke:#7e57c2;
  classDef viz fill:#fdecec,stroke:#e57373;
```
