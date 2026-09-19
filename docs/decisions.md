# Decisions

Each entry records what was chosen, what was rejected, and what it cost.

## Discovery-only V1 with V2 code retained

**Chosen:** build the full transactional platform, ship only discovery, gate the
rest behind build-time flags.

**Rejected:** building V1 standalone and adding transactions later — which means
designing the order and ticket model twice, the second time against live data.
Also rejected: shipping payments immediately, which carries settlement and
compliance risk before there is an audience to justify it.

**Cost:** unexercised code drifts, and 26 test files is not enough coverage to
catch that automatically. The V2 flip needs a deliberate verification pass.
Detail in [product-modes.md](product-modes.md).

## Queue-and-review rather than automatic reconciliation for offline check-in

**Chosen:** flag every server disagreement to a human supervisor. Five of six
terminal queue states require review.

**Rejected:** last-write-wins and server-receipt-time ordering. Both resolve
conflicts silently, and a silent wrong answer at a door either admits a duplicate
or turns away a paying guest, with no one finding out until reconciliation.

**Why it works here:** a supervisor is physically present at the gate and has
information the system does not. This reasoning does not transfer to a system
without a human in the loop.

**Cost:** false positives are manual work. Without server-side idempotency, a
lost response on a successful scan produces a `DUPLICATE` flag that a person has
to clear. Detail in [offline-checkin.md](offline-checkin.md).

## Zod schemas in a shared package as the API contract

**Chosen:** one schema definition producing runtime validation server-side and
static types client-side.

**Rejected:** OpenAPI-generated clients, which add a generation step and a window
where the spec and the implementation disagree. Also rejected: a hand-maintained
SDK, which is a second thing to keep in sync.

**Cost:** `shared` is a coordination point. A change there rebuilds everything,
and it needs discipline to keep surface-specific concerns out of it.

## Domain-organised backend modules with a repository layer

**Chosen:** each domain owns routes, controller, service and repository together.

**Why:** the repository layer concentrates database access in one place per
domain, which is where scoping and soft-delete rules can be enforced once. The
value is entirely in that property, so the layering has to be enforced in review
— a service reaching past its repository silently removes the benefit.

## Two staging Compose variants

**Chosen:** maintain both a shared-infrastructure and a fully standalone staging
configuration.

**Why:** the same stack deploys onto a host that already runs other services and
onto a clean box. Discovering at deploy time that a compose file assumes
infrastructure that is not present is a bad time to find out.

**Cost:** two files to keep in step.
