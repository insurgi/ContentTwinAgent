1) KPI Relationship Map (from Velocity → Epic Progress → Automation → Engagement)
flowchart LR
  %% --- Inputs & Planning ---
  subgraph Planning[Planning & Capacity]
    CAP[Team Capacity]
    CAL[Calendar Quality & Scope]
    DEP[Dependencies & Risks]
  end

  %% --- Delivery Metrics ---
  subgraph Delivery[Delivery Metrics]
    CP[Committed Points]
    WIP[Work In Progress]
    BD[Burndown Trend]
    CMP[Completed Points]
    VEL[Velocity]
    PRED[Predictability (±15%)]
  end

  %% --- Quality & Stability ---
  subgraph Quality[Quality & Stability]
    DEF[Defect Density]
    REO[Reopen Rate]
    BDD[BDD Coverage %]
    STAB[Release Stability]
  end

  %% --- Automation Layer ---
  subgraph Automation[Automation & Ops]
    AUTO[Automation Coverage %]
    PUBL[Publishing Reliability %]
    API[API Success Rate]
    UPT[System Uptime %]
    REND[Rendering Throughput]
  end

  %% --- Outcomes & Adaptation ---
  subgraph Outcome[Outcomes & Adaptation]
    EPIC[Epic Progress %]
    KPI[Weekly Output (Videos/Week)]
    ENG[Engagement Lift %]
    ADAPT[Strategy Adaptation Accuracy %]
  end

  %% Relationships
  CAP --> CP
  CAL --> CP
  DEP --> CP

  CP --> WIP --> BD --> CMP --> VEL --> PRED
  VEL --> EPIC

  %% Quality influences velocity & stability
  BDD --> STAB
  DEF -->|negative| STAB
  REO -->|negative| STAB
  STAB --> VEL

  %% Automation chain
  AUTO --> PUBL
  API --> PUBL
  UPT --> PUBL
  REND --> KPI
  PUBL --> KPI

  %% Outcomes
  EPIC --> KPI
  KPI --> ENG
  PUBL --> ENG

  %% Feedback loop to planning
  ENG --> ADAPT --> CAL

2) Alert & Escalation Map (what triggers action)
flowchart TB
  subgraph Monitors[Monitored KPIs]
    VEL[Velocity < 85% of Committed]
    DEF[Defect Density > 0.25]
    AUTO[Automation Coverage < 80%]
    REL[Publishing Reliability < 98%]
    UPT[Uptime < 99.5%]
    ENG[Engagement ↓ ≥20% vs prior week]
  end

  subgraph Alerts[Alerts & Owners]
    SM[Scrum Master]
    QA[QA Lead]
    AI[AI/ML Engineer]
    BE[Backend Lead]
    DO[DevOps]
    ROC[Rocco]
  end

  VEL --> SM
  DEF --> QA
  AUTO --> AI
  REL --> BE
  UPT --> DO
  ENG --> ROC

  %% Actions
  SM --> RETRO[Sprint Retro & Replan]
  QA --> RCA[Root Cause Analysis & Test Gap]
  AI --> TUNE[Agent Logic Tuning]
  BE --> RETRY[Queue/Retry & API Hardening]
  DO --> INCIDENT[Incident Response & Postmortem]
  ROC --> STRAT[Trigger Strategy Update]

  %% Close the loop
  RETRO -->|adjust scope/capacity| VEL
  RCA -->|raise BDD coverage| DEF
  TUNE -->|raise automation| AUTO
  RETRY -->|raise reliability| REL
  INCIDENT -->|stabilize| UPT
  STRAT -->|new topics/timing| ENG
