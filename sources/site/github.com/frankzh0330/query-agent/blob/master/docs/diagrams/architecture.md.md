# Source: https://github.com/frankzh0330/query-agent/blob/master/docs/diagrams/architecture.md

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# architecture.md

Copy path

Blame

More file actions

Blame

More file actions

## Latest commit

[![frankzh0330](https://avatars.githubusercontent.com/u/205485759?v=4&size=40)](https://github.com/frankzh0330) [frankzh0330](https://github.com/frankzh0330/query-agent/commits?author=frankzh0330)

[Remove duplicate Chinese diagram docs](https://github.com/frankzh0330/query-agent/commit/38f7c137f64e6ea8a42d6cf6599b57017750433b)

Jul 28, 2026

[38f7c13](https://github.com/frankzh0330/query-agent/commit/38f7c137f64e6ea8a42d6cf6599b57017750433b) · Jul 28, 2026

## History

[History](https://github.com/frankzh0330/query-agent/commits/master/docs/diagrams/architecture.md)

Open commit details

History

118 lines (103 loc) · 3.55 KB

## FilesExpand file tree

master

/

# architecture.md

Copy path

Top

## File metadata and controls

- Preview

- Code

- Blame

118 lines (103 loc) · 3.55 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/docs/diagrams/architecture.md)

Copy raw file

Download raw file

Edit and raw actions

# Architecture Diagram

Loading

```
graph TB
    subgraph Client["Client Layer"]
        TelegramUser["Telegram User"]
        HttpClient["HTTP Client"]
    end

    subgraph Gateway["Gateway Layer"]
        TelegramGateway["TelegramGateway<br/>Long Polling"]
        GatewayManager["GatewayManager<br/>Channel Management"]
        BaseGateway["BaseGateway"]
    end

    subgraph Ingress["Ingress Layer"]
        TelegramAdapter["TelegramAdapter"]
        Cleaner["MessageCleaner<br/>Text Cleaning"]
        Dedup["MessageDeduplicator<br/>Deduplication"]
        StandardMessage["StandardMessage<br/>Unified Message Model"]
    end

    subgraph API["API Layer"]
        FastAPI["FastAPI App<br/>app.py"]
        Models["NL2DSLRequest<br/>NL2DSLResponse"]
        TelegramNotify["Telegram Notification Helper"]
    end

    subgraph Service["Service Layer"]
        Orchestrator["QueryOrchestrator<br/>Turn Routing + Business Flow"]
        LLM["LLM Extractions"]
        SessionManager["SessionManager<br/>Session State"]
        TaskManager["TaskManager<br/>Confirmation Tasks"]
        BearerService["BearerService<br/>Query Execution"]
        CatalogSync["CatalogSync<br/>Metadata Refresh"]
    end

    subgraph Memory["Memory Layer"]
        LongTermMemory["LongTermMemory<br/>Project Memory"]
        MemoryWriter["MemoryWriter<br/>Async Learning"]
        UserPreference["UserPreferenceStore<br/>Weak Rerank Signal"]
        Storage["JSONL / Markdown / JSON Storage"]
    end

    subgraph Matcher["Matcher Layer"]
        MatcherService["MatcherService"]
        EventMatcher["EventMatcher"]
        MetricMatcher["MetricMatcher"]
        DimensionMatcher["DimensionMatcher"]
        TimeMatcher["TimeMatcher"]
    end

    subgraph DSL["DSL Layer"]
        SemanticDSL["Semantic Models"]
        Renderer["Exec DSL Renderer"]
        Validators["Validators"]
    end

    subgraph Catalog["Catalog Layer"]
        Metrics["metrics.yaml"]
        Events["events.yaml"]
        Dimensions["dimensions.yaml"]
        Regions["region_groups.yaml"]
        CatalogLoader["Catalog Loader"]
    end

    subgraph External["External Services"]
        TelegramAPI["Telegram Bot API"]
        LLMBackend["LLM Backend"]
        BearerAPI["Bearer Query Engine"]
        CatalogAPI["Catalog API"]
    end

    TelegramUser --> TelegramAPI
    TelegramAPI --> TelegramGateway
    HttpClient --> FastAPI
    GatewayManager --> TelegramGateway
    TelegramGateway -.-> BaseGateway
    TelegramGateway --> TelegramAdapter
    TelegramAdapter --> Cleaner
    TelegramAdapter --> Dedup
    TelegramAdapter --> StandardMessage
    StandardMessage --> Orchestrator
    FastAPI --> Orchestrator

    Orchestrator --> SessionManager
    Orchestrator --> TaskManager
    Orchestrator --> LLM
    Orchestrator --> MatcherService
    Orchestrator --> SemanticDSL
    Orchestrator --> Renderer
    Orchestrator --> Validators
    Orchestrator --> MemoryWriter
    Orchestrator --> UserPreference

    SessionManager --> LongTermMemory
    LongTermMemory --> Storage
    MemoryWriter --> Storage
    UserPreference --> Storage

    MatcherService --> EventMatcher
    MatcherService --> MetricMatcher
    MatcherService --> DimensionMatcher
    MatcherService --> TimeMatcher
    EventMatcher --> Events
    MetricMatcher --> Metrics
    DimensionMatcher --> Dimensions
    CatalogLoader --> Metrics
    CatalogLoader --> Events
    CatalogLoader --> Dimensions
    CatalogLoader --> Regions

    LLM --> LLMBackend
    BearerService --> BearerAPI
    CatalogSync --> CatalogAPI
    Renderer --> BearerService
    TelegramGateway --> TelegramAPI
```