# Polyglot Persistence for a Next-Gen MMORPG

This repository contains the PDF of my Data Science
Technology homework at the University of Naples Parthenope.

The homework designs a **data storage architecture** for a fictitious
global-scale MMORPG called *Loca Lucha*, using **polyglot persistence**:
different database technologies are selected and combined, each assigned to the
subset of data and workload it is natively best suited for.


## Overview

The report focuses exclusively on the **data storage** aspects of the game
backend. It does not cover implementation or deployment details.

The backend must support:

- Account and billing management (accounts, authentication, wallets,
  cash-shop transactions).
- Real-time gameplay state (sessions, matchmaking, PvP leaderboards).
- Persistent game state (character profiles, inventories, quest progress,
  in-world coordinates).
- Telemetry and analytics (gameplay events, errors, anti-cheat signals,
  store events, chat).

Because these tasks have heterogeneous and partially conflicting requirements,
the design adopts **polyglot persistence** instead of a single monolithic DBMS.

## Data Classification

Following the literature on MMORPG storage, game data is classified into four
main classes:

1. **Account Data** – user credentials, email, wallet balances, recharge
   history, subscription status.
2. **Game Data** – item and NPC metadata, class definitions, map geometry,
   quest scripts, balancing parameters.
3. **State Data** – character level and stats, inventory contents, quest
   progress, position, temporary buffs/debuffs.
4. **Log Data** – gameplay events (kills, deaths, joins/leaves), errors,
   anti-cheat signals, chat events, store events.

The report estimates volumes and access patterns for each class (NU, DAU, CCU,
GB/TB per year) and uses them to motivate different storage choices.

## Data Flow Diagram

The Data Flow Diagram models four main processes and five data stores:

**Processes**

- `P1 – Login & Account`  
  Authenticates the player and reads account data from the relational store.

- `P2 – Game Service (State & Matchmaking)`  
  Manages sessions, matchmaking and PvP arenas, reading and writing character
  state.

- `P3 – Store Service (Economic Transactions)`  
  Processes cash-shop purchases and premium currency operations.

- `P4 – Telemetry (Game & Economic Events)`  
  Collects and normalises gameplay and financial events before sending them to
  the analytics pipeline.

**Data Stores**

- `D1 – Account DB (PostgreSQL)`  
  Account records, wallet balances, transaction logs.

- `D2 – Session Store (Redis)`  
  Hot session state, matchmaking queues, leaderboards.

- `D3 – Profile & Inventory DB (Apache Cassandra)`  
  Persistent character profiles and inventories.

- `D5 – Telemetry Bus (Kafka/Kinesis)`  
  Normalised events buffered for ingestion.

- `D4 – Telemetry Lake (Data Lake + Apache Hive)`  
  Long-term storage for telemetry and log data, queried via Hive for analytics.

The DFD makes explicit that there is no single “central” database: different
stores are used in parallel, each serving the subset of data and operations for
which it is best suited.

## Storage Technologies

The report discusses and motivates the following technologies:

- **PostgreSQL (Relational OLTP)**  
  Used for account and financial data, where ACID properties and strong
  consistency are non-negotiable.  
  Also includes a short “bad ideas” section explaining why Cassandra or generic
  key–value stores are inappropriate for real-money transactions.

- **Redis (In-Memory Key–Value)**  
  Used for session tokens, matchmaking state and PvP leaderboards.  
  Focus on sub-millisecond latency, specialised data structures (Sorted Sets),
  and the ephemeral nature of this data.

- **Apache Cassandra (Wide-Column NoSQL)**  
  Used for persistent character profiles and inventories in a
  geographically-distributed environment.  
  Highlights flexible schema, masterless architecture and tunable
  consistency (e.g. `LOCAL_QUORUM`).

- **Data Lake + Apache Hive (OLAP)**  
  Used for high-volume append-only telemetry logs (tens of terabytes per year).  
  Emphasises separation between OLTP and OLAP workloads, columnar storage, and
  SQL-like analytics over large tables.

Each subsection also briefly considers **alternative options and “bad ideas”**
to show why certain technologies are not appropriate for given data classes.

## Record Examples

To mirror the official homework example, the report includes simple record
examples:

- JSON document for a character profile stored in Cassandra.
- CSV line for a gameplay telemetry event stored in the Data Lake.
- JSON object for an account record in PostgreSQL.
- Example keys and Sorted Set entries for Redis (session tokens and PvP
  leaderboard).

These examples illustrate the different data formats and further motivate the
choice of storage technologies.

## References

The report references:

- Diao & Schallehn, *Towards Cloud Data Management for MMORPGs* and *Achieving
  Consistent Storage for Scalable MMORPG Environments*.
- Riot Games, *Big Data at Riot Games* (Hadoop Summit 2012).

These works support the decision to use a mix of relational, NoSQL and
data-lake technologies for large online games.

