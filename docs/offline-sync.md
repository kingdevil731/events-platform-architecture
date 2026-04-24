## Offline Check-In Flow

Scan ticket
↓
Store event locally
↓
Queue pending sync
↓
Replay when connectivity returns
↓
Server validates
↓
Reconcile final state

## Sync State Handling

Possible outcomes:

- synced
- duplicate
- invalid
- conflict
- failed

## Core Concerns

Designed around:

- replay safety
- duplicate prevention
- idempotent handling
- connectivity interruptions
