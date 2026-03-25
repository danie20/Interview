
```mermaid
flowchart TB
  subgraph clients["Clients"]
    D["Desktop app"]
    M["Mobile app"]
  end

  G["GraphQL service<br/>(API / BFF)"]

  subgraph data["Data layer"]
    C[("In-memory cache<br/>(e.g. Caffeine, Redis local)")]
    DB[("MongoDB")]
  end

  D -->|"HTTPS / GraphQL"| G
  M -->|"HTTPS / GraphQL"| G
  G <-->|"read / write"| C
  G <-->|"source of truth / persistence"| DB
```

## Cache-aside (typical read path)

```mermaid
sequenceDiagram
  participant C as Client (Desktop / Mobile)
  participant G as GraphQL service
  participant K as In-memory cache
  participant M as MongoDB

  C->>G: GraphQL query
  G->>K: get(key)
  alt cache hit
    K-->>G: value
    G-->>C: response
  else cache miss
    K-->>G: miss
    G->>M: query / aggregate
    M-->>G: documents
    G->>K: put(key, value)
    G-->>C: response
  end
```

## Writes / cache invalidation (typical pattern)

```mermaid
flowchart LR
  subgraph clients["Clients"]
    D[Desktop]
    M[Mobile]
  end

  G["GraphQL service"]

  D -->|mutation| G
  M -->|mutation| G
  G -->|persist| DB[("MongoDB")]
  G -->|invalidate or update keys| K[("In-memory cache")]
```
