```mermaid
flowchart LR

%% --- Subgraph: Analytics ---
subgraph Analytics["Analytics"]
    ATH["Athena\nevents_flat, snapshots"]
    PBI["Power BI\nAthena ODBC"]
end

%% --- Subgraph: Ticketing / Decisions ---
subgraph Tickets["Ticketing / Decisions"]
    RM["Redmine\nIssues + Custom Fields"]
    HUM["Humans\nAnalysts / PMs"]
    AGENT["Other AI Agents\n(future)"]
end

%% LambdaWatcher flow
LW -->|fetch_url_text| WEB["Watched Websites\ne.g. GitHub topics/recipes"]
WEB --> LW
LW -->|save_raw_to_s3| S3
LW -->|ensure_baseline_exists\nsave_snapshot| S3
LW -->|write_change_event_to_s3\nevents/YYYY/MM/DD/...| S3

%% RAG
LW -->|embed_texts_openai| OA
OA --> VS
LW --> VS

%% Ticket creation
LW -->|on significant change\ncreate_redmine_issue| RM

%% Analytics side
S3 --> ATH
ATH --> PBI
RM -->|issues, status, severity| PBI

%% People / other agents
RM <--> HUM
RM <--> AGENT
