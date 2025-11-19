subgraph Analytics["Analytics"]
    ATH[Athena<br/>events_flat, snapshots]
    PBI[Power BI<br/>Athena ODBC]
end

subgraph Tickets["Ticketing / Decisions"]
    RM[Redmine<br/>Issues + Custom Fields]
    HUM[Humans<br/>Analysts / PMs]
    AGENT[Other AI Agents<br/>(future)]
end

%% LambdaWatcher flow
LW -->|fetch_url_text| WEB[Watched Websites<br/>(e.g. GitHub topics/recipes)]
WEB --> LW
LW -->|save_raw_to_s3| S3
LW -->|ensure_baseline_exists<br/>save_snapshot| S3
LW -->|write_change_event_to_s3<br/>events/YYYY/MM/DD/...| S3

%% RAG
LW -->|embed_texts_openai| OA
OA --> VS
LW --> VS

%% Ticket creation
LW -->|on significant change<br/>create_redmine_issue| RM

%% Analytics side
S3 --> ATH
ATH --> PBI
RM -->|issues, status, severity| PBI

%% People / other agents
RM <---> HUM
RM <---> AGENT
