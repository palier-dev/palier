# ADR 0001: Start with a monorepo

- **Status:** Accepted
- **Date:** 2026-10-23

## Context
The project is public and educational. Readers and coding agents both need to see domain docs, data, evals and code together. No service boundaries exist yet.

## Decision
Use a single monorepo (`palier`) containing docs, synthetic data, evals, services and infrastructure.

## Consequences
- One place for full context, easier for agents and readers.
- Split later if bounded contexts or deployment cadence justify it.
