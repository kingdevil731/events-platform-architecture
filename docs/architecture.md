# Architecture

## Monorepo layout

```
apps/
  api             Express + TypeScript + Prisma      :8080
  mobile          React Native, Expo 54, Expo Router
  web-customer    Next.js 15 + Mantine               :3000
  web-organizer   Next.js 15 + Mantine               :3001
  checkin-web     Next.js 15  (V2)                   :3002
  web-links       Next.js 15                         :3003
packages/
  shared          Zod schemas, DTOs, enums, feature resolution
  i18n            i18next, shared across web surfaces
  design-tokens
```

Turborepo with pnpm workspaces. Biome for lint and format.

## Why one shared contract package

Six clients against one API, two of them native. The options were a
hand-maintained SDK, generated clients from an OpenAPI spec, or a shared source
of truth compiled into every surface.

`packages/shared` holds Zod schemas organised by domain — `auth`, `event`,
`order`, `ticket`, `checkin`, `payment`, `organizer`, `notification`, `user`,
`alert`, `common`. The API validates requests against these schemas at runtime;
clients import the inferred TypeScript types. One definition produces both.

The consequence that matters: a breaking contract change fails `pnpm build`
across every dependent surface, at the point of change. There is no window where
the API and a client disagree and nobody notices until runtime. This is worth
more in a monorepo with a React Native app in it than it would be otherwise,
because the mobile release cycle is slow enough that a contract mismatch shipped
to the store is expensive to unwind.

The cost is that `shared` becomes a coordination point — a change there touches
everything, and it needs discipline about what belongs in it. Transport concerns
and surface-specific view models stay out.

## Backend structure

`apps/api/src/api/` is organised by domain, not by layer. Each module owns its
routes, controller, service and repository:

```
api/checkin/
  checkin.routes.ts
  checkin.controller.ts
  checkin.service.ts
  checkin.repository.ts
```

Roughly twenty such modules — `event`, `ticket`, `ticket-type`, `order`,
`payment`, `organizer`, `event-seller`, `event-content`, `event-interaction`,
`notification`, `community-alert`, `analytics`, `audit`, `capability`, `admin`,
`auth`, `banner`, `tag`, `upload`.

Route → controller → service → repository is not novel. It is chosen here for a
specific reason: the repository layer is the single place where database access
happens, which is where scoping and soft-delete rules can be enforced once rather
than per call site. That property only pays off if the layering is actually
respected, so it is one of the guardrails in the contribution rules.

## Data

PostgreSQL via Prisma. 24 forward-only migrations, starting March 2026. The
migration history is readable as a product history — tags, event sellers,
organizer verification states, admin invites, user locale, community alerts.

Redis is present for caching and session concerns.

## Infrastructure

Docker Compose in three configurations:

- **Local dev** — Postgres, Redis, API, four web apps. Mobile runs on the host
  under Expo.
- **Staging, shared infra** — joins an existing host network and shared Postgres.
- **Staging, standalone** — fully self-contained, brings its own everything.

Two staging variants exist because the same stack has to deploy both onto a box
that already runs other services and onto a clean one. Maintaining both is cheap;
discovering at deploy time that the compose file assumes infrastructure that is
not there is not.

`checkin-web` is behind a Compose profile and off by default, consistent with its
V2 status.

## CI

Two workflows. `pr-reminders.yml` posts the contribution rules on every pull
request. `repo-standards.yml` runs on pull requests and on pushes to main, and
fails if any required document is missing — the architecture guardrails, the
design doc, the mobile and API guides, the check-in spec, the copy guide.

Testing that documentation exists is a weak check. It is there because in a
repository where the V1/V2 boundary is a policy document rather than a code
property, the policy document going missing is a real failure mode.

Documentation content is not verified, and coverage across the monorepo is 26
test files — thin, and the honest weak point of the current setup.
