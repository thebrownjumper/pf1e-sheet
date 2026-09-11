# ADR-0001: Record architecture decisions

**Status:** Accepted — 2026-09-11

## Context
This project will run over roughly a year of part-time work, with gaps. Decisions made in
month two will be invisible by month nine, and the reasoning behind them will be gone.

## Decision
Every significant decision gets a short file in `docs/adr/`, numbered sequentially, with
Context / Decision / Consequences. Significant means: hard to reverse, or surprising to a
reader of the code.

## Consequences
Slight overhead per decision. In exchange, returning to the project after a break starts with
reading, not archaeology.
