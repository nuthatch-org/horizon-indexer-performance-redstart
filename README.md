# horizon-indexer-performance-redstart

A subgraph for **The Graph Horizon** — indexing staking, operator authorizations,
allocations, delegations, indexing rewards, and query fees across three contracts
on **Arbitrum One**, with timeseries + daily/hourly aggregations.

It is authored in **[Redstart](https://github.com/lodestar-team/redstart)** — a
single-language subgraph compiler — and ejects to a standard AssemblyScript
subgraph that the canonical Graph CLI compiles and deploys **unmodified**.

🟢 **Live on Subgraph Studio:** [`horizon-indexer-performance-redstart`](https://thegraph.com/studio/subgraph/horizon-indexer-performance-redstart)
· Query endpoint: `https://api.studio.thegraph.com/query/1754959/horizon-indexer-performance-redstart/v0.0.1`

---

## Credit

This subgraph is a faithful, behaviour-for-behaviour port of
**[PaulieB14/horizon-indexer-subgraph](https://github.com/PaulieB14/horizon-indexer-subgraph)**.
The schema design, contract selection, event-handling logic, and the
timeseries/aggregation model are the work of
**[PaulieB14](https://github.com/PaulieB14)**. All credit for the indexing design
belongs to the original author; this repo only re-expresses it in Redstart and
ships the generated subgraph.

---

## Repository layout

This repo holds the subgraph in **two forms**:

```
redstart/      # ── Source of truth ── the Redstart (.red) authoring layer
  redstart.toml
  src/
    main.red             # ABIs + the 3 data sources
    schema.red           # entities, timeseries, aggregations
    helpers.red          # getOrCreateIndexer / getOrCreateGlobalStats / …
    staking.red          # HorizonStaking handler
    delegation.red       # StakingExtension handlers
    subgraph_service.red # SubgraphService handlers
    tests.red            # native handler tests
    abis/*.json

subgraph/      # ── Generated ── the standard graph-cli project (ejected from Redstart)
  schema.graphql
  subgraph.yaml
  src/mappings.ts
  abis/*.json
  package.json
```

The `subgraph/` directory is a normal, self-contained Graph CLI project: you can
build and deploy it **without installing Redstart**. The `redstart/` directory is
how it's authored and maintained — edit the `.red` modules and regenerate.

## Deploy the generated subgraph (no Redstart required)

```sh
cd subgraph
npm install
graph auth <YOUR_STUDIO_DEPLOY_KEY>
npm run codegen
npm run build
graph deploy horizon-indexer-performance-redstart --version-label v0.0.1
```

## Or work from the Redstart source

```sh
# with the redstart CLI (https://github.com/lodestar-team/redstart):
cd redstart
redstart check .
redstart test .          # native handler tests — no Docker, no graph-node
redstart build .         # regenerates ../subgraph artifacts
redstart deploy horizon-indexer-performance-redstart . --version-label v0.0.1
```

## What it indexes

| Contract | Address (Arbitrum One) | Events |
|---|---|---|
| HorizonStaking | `0x00669A4CF01450B64E8A2A20E9b1FCB71E61eF03` | `OperatorSet` |
| SubgraphService | `0xb2Bb92d0DE618878E438b55D5846cfecD9301105` | `AllocationCreated` / `Closed` / `Resized`, `IndexingRewardsCollected`, `QueryFeesCollected` |
| StakingExtension | `0x3bE385576d7C282070Ad91BF94366de9f9ba3571` | `StakeDelegated` / `Locked` / `Withdrawn` |

Entities: `Indexer`, `Operator`, `Allocation`, `Delegation`, immutable
`RewardEvent` / `QueryFeeEvent` logs, a `GlobalStats` singleton, and three
timeseries → aggregation pairs (`RewardData` → `RewardDailyAgg`, etc.).

## License

MIT. Original indexing design © [PaulieB14](https://github.com/PaulieB14).
