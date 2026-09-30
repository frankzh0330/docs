# Source: https://github.com/frankzh0330/query-agent/blob/master/docs/diagrams/matcher-sequence.md

[frankzh0330](https://github.com/frankzh0330) / **[query-agent](https://github.com/frankzh0330/query-agent)** Public

- [Notifications](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent) You must be signed in to change notification settings
- [Fork 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)
- [Star 0](https://github.com/login?return_to=%2Ffrankzh0330%2Fquery-agent)

 ## FilesExpand file tree

master

/

# matcher-sequence.md

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

[History](https://github.com/frankzh0330/query-agent/commits/master/docs/diagrams/matcher-sequence.md)

Open commit details

History

53 lines (49 loc) · 1.95 KB

## FilesExpand file tree

master

/

# matcher-sequence.md

Copy path

Top

## File metadata and controls

- Preview

- Code

- Blame

53 lines (49 loc) · 1.95 KB

[Raw](https://github.com/frankzh0330/query-agent/raw/refs/heads/master/docs/diagrams/matcher-sequence.md)

Copy raw file

Download raw file

Edit and raw actions

# Matcher Sequence

Loading

```
sequenceDiagram
    participant API as FastAPI /nl2dsl
    participant MS as MatcherService
    participant Matcher as Event/Metric/Dimension Matcher
    participant Pipeline as BaseMatcher Pipeline
    participant Index as Inverted Index
    participant Fuzz as RapidFuzz
    participant Catalog as Catalog YAML
    participant Pref as UserPreferenceStore

    rect rgb(227, 242, 253)
    Note over API,Catalog: Initialization - build matcher indexes
    API->>MS: MatcherService(catalog_path)
    MS->>Catalog: Load metrics, events, dimensions
    Catalog-->>MS: Catalog object with entities and aliases
    MS->>Matcher: Build indexes
    Matcher->>Pipeline: Build exact alias table and inverted index
    Pipeline->>Index: Store token -> document IDs
    end

    rect rgb(232, 245, 233)
    Note over API,Fuzz: Runtime - resolve one extracted field
    API->>MS: resolve_from_extractions(type, extractions, default)
    MS->>MS: Pick first extraction text
    MS->>Matcher: match(query)
    Matcher->>Pipeline: Stage 1 exact alias lookup
    alt Exact match
        Pipeline-->>Matcher: score=100, method=exact_alias_match
    else No exact match
        Pipeline->>Pipeline: Stage 2 tokenize + synonym expansion
        Pipeline->>Index: Stage 3 inverted-index recall
        Index-->>Pipeline: candidate document IDs
        Pipeline->>Fuzz: Stage 4 fuzzy rerank
        Fuzz-->>Pipeline: sorted candidates
        alt Score passes threshold
            Pipeline-->>Matcher: best candidate + score
        else No candidate passes threshold
            Pipeline-->>Matcher: no_match
        end
    end
    Matcher-->>MS: ResolvedResult with candidates and explain
    end

    rect rgb(255, 249, 196)
    Note over MS,Pref: User preference is applied only after recall
    MS->>Pref: rerank_candidates(project_id, user_id, field, candidates)
    Pref-->>MS: weakly biased candidates + explain
    MS-->>API: resolved value + resolver_explain
    end
```