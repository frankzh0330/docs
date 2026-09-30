# Source: https://github.com/frankzh0330/query-agent/blob/master/docs/diagrams/flowchart.md

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# flowchart.md

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

[History](https://github.com/frankzh0330/query-agent/commits/master/docs/diagrams/flowchart.md)

Open commit details

History

68 lines (58 loc) · 2.26 KB

## FilesExpand file tree

master

/

# flowchart.md

Copy path

Top

## File metadata and controls

- Preview

- Code

- Blame

68 lines (58 loc) · 2.26 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/docs/diagrams/flowchart.md)

Copy raw file

Download raw file

Edit and raw actions

# End-To-End Flowchart

Loading

**Unable to render rich display**

Could not find a suitable point for the given distance 

For more information, see https://docs.github.com/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams#creating-mermaid-diagrams 

```
flowchart TD
    Start(("User sends query")) --> Channel{"Channel?"}

    Channel -->|Telegram| Poll["TelegramGateway<br/>Long polling"]
    Poll --> Adapt["TelegramAdapter<br/>Convert to StandardMessage"]
    Adapt --> Clean["MessageCleaner<br/>Clean text"]
    Clean --> Dup{"Duplicate?"}
    Dup -->|Yes| DropDup(("Discard"))
    Dup -->|No| Empty{"Empty after cleaning?"}
    Empty -->|Yes| DropEmpty(("Discard"))
    Empty -->|No| Session

    Channel -->|HTTP| HTTP["FastAPI POST /nl2dsl"]
    HTTP --> Session

    Session["SessionManager<br/>create_or_get"]
    Session --> Context["Enhanced Context<br/>Session + Project Memory"]
    Context --> Extract

    subgraph Extract["Layer 1: LLM Extraction"]
        Prompt["Build context-aware prompt"]
        LLM["Call LLM"]
        Parsed["Parse ExtractionsJson"]
        Prompt --> LLM --> Parsed
    end

    Parsed --> Followup["Follow-up Detection"]
    Followup --> Turn{"Turn mode?"}
    Turn -->|new_query| Resolve
    Turn -->|followup_patch| Patch["Extract patch"]
    Patch --> Merge["Merge with last_query_state"]
    Merge --> Resolve
    Turn -->|confirmation| Confirm["Restore pending task"]
    Confirm --> Resolve

    subgraph Resolve["Layer 2: Matcher Resolution"]
        Metric["MetricMatcher"]
        Event["EventMatcher"]
        Dimension["DimensionMatcher"]
        Time["TimeMatcher"]
        Metric --> Event --> Dimension --> Time
    end

    Time --> Ambiguous{"Needs confirmation?"}
    Ambiguous -->|Yes| Task["Create TaskContext<br/>Return candidates"]
    Task --> EndConfirm(("Wait for user reply"))
    Ambiguous -->|No| BuildDSL

    subgraph BuildDSL["Layer 3-4: DSL Build And Render"]
        Semantic["Build Semantic DSL"]
        Render["Render Exec DSL"]
        Validate["Validate region and consistency"]
        Semantic --> Render --> Validate
    end

    Validate --> Persist["Persist session state<br/>Record preferences"]
    Persist --> Learn["Async MemoryWriter"]
    Persist --> Caller{"Caller?"}
    Caller -->|Telegram| Execute["BearerService execute_query"]
    Execute --> Format["Format response"]
    Format --> Send["Send Telegram response"]
    Send --> EndTG(("End"))
    Caller -->|HTTP| Return["Return NL2DSLResponse"]
    Return --> EndHTTP(("End"))
```