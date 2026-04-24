# Events Platform

Architecture showcase for a private event commerce platform.

Focus areas:

- Ticketing workflows
- Offline-first sync systems
- Reconciliation logic
- Distributed state handling
- Operational tooling

## Live Application

Add staging or live URL here.

## Overview

Platform supporting:

- Event discovery
- Ticket purchases
- Ticket wallet and QR tickets
- Organizer reporting
- Offline venue check-in
- Sync replay and reconciliation

## Standout Engineering Problem

How can venue check-in continue working when connectivity fails,
without corrupting ticket validity or introducing inconsistent state?

This led to an offline-first replay architecture.

## Core Architecture

Customer App
↓
Ticketing Platform
↓
QR Validation / Check-In App
↓
Offline Queue
↓
Sync Replay Engine
↓
Reconciliation Layer

## Ticket Lifecycle

Discover Event
↓
Purchase Ticket
↓
Ticket Confirmation
↓
QR Ticket Issued
↓
Venue Check-In
↓
Offline Queue (if needed)
↓
Replay / Validation
↓
Final Reconciliation

## Offline Sync State Model

Check-in events move through states:

PENDING_SYNC
→ SYNCED
→ DUPLICATE
→ INVALID
→ CONFLICT
→ FAILED

## Engineering Challenges Solved

- Offline-first operation
- Replay queue processing
- Duplicate scan handling
- Conflict detection
- Eventual consistency
- Operational reconciliation

## Stack

Backend

- Node.js
- TypeScript
- PostgreSQL
- Prisma
- Docker

Frontend

- React
- Next.js

Operational Concepts

- Offline queue
- Replay engine
- Reconciliation workflows

## Architecture Metrics

Capability → Approach

Offline Sync → Queued Replay
Validation → Conflict-Aware
Ticket State → Server Reconciliation
Operations → Organizer Reporting

## Architecture Decisions

### Why offline queueing

Allows venue operations to continue without connectivity.

### Why replay model

Supports resilient sync after reconnect.

### Why reconciliation layer

Separates sync from authoritative ticket state validation.

## Future Scaling Considerations

Potential evolution:

- Distributed replay workers
- More advanced idempotency controls
- Analytics instrumentation
- Multi-venue operational scaling

## Screenshots

Add:

- Event discovery
- Checkout flow
- Ticket wallet
- Check-in app
- Reconciliation dashboard

## Repository Note

This repository intentionally contains architecture documentation and showcase material only.

Core implementation remains private.
