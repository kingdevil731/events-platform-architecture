# Product modes: shipping V2 code in V1

## The decision

MozCultura is designed as a transactional events platform — tickets, orders,
payments, wallet, door check-in. It launched without any of that exposed.

The reasoning is commercial. Organizers in Mozambique already collect payment
through M-Pesa, e-Mola and bank transfer, and they have no reason to route money
through an unknown platform. Becoming a payment processor first would mean
carrying settlement and compliance risk in order to serve organizers who did not
yet trust the product. Discovery first, transactions once there is an audience.

That left an engineering question with three bad-looking answers:

1. **Build V1 only, add V2 later.** Cheapest now, and it means designing the data
   model twice — the second time under commercial pressure, against live data.
2. **Build everything, ship everything.** Ships a checkout nobody uses and a
   payment integration that has to be maintained and secured from day one.
3. **Build everything, ship the V1 surface.** Carries dead code.

Option 3 was chosen, and the whole design problem is making the carried code not
rot.

## Mechanism

Feature resolution is layered: a coarse product mode, then granular per-feature
overrides, resolved once at module load.

```ts
const v2 = v2OverridesFromMode(process.env.NEXT_PUBLIC_PRODUCT_MODE);

const granular = pickFeatureOverrides({
  paymentsEnabled:  parseFeatureBool(process.env.NEXT_PUBLIC_PRODUCT_PAYMENTS_ENABLED),
  ordersEnabled:    parseFeatureBool(process.env.NEXT_PUBLIC_PRODUCT_ORDERS_ENABLED),
  ticketsEnabled:   parseFeatureBool(process.env.NEXT_PUBLIC_PRODUCT_TICKETS_ENABLED),
  checkinEnabled:   parseFeatureBool(process.env.NEXT_PUBLIC_PRODUCT_CHECKIN_ENABLED),
  // ...
});

export const features: Features = resolveFeatures({ ...v2, ...granular });
```

The mode sets a baseline; individual flags override it. That matters for staging,
where the useful configuration is V1 everywhere except one feature under test.

`resolveFeatures` and the flag types live in `packages/shared`, so all six
surfaces resolve flags identically rather than each re-implementing the
precedence rules.

### One constraint worth knowing

Next inlines `NEXT_PUBLIC_*` at build time by textual substitution, so each
variable must appear as a literal property access. `process.env[name]` does not
get replaced and silently yields `undefined` — which `parseFeatureBool` reads as
false, quietly disabling a feature. The verbose literal block above is deliberate
and is commented at the call site, because the concise version fails silently at
runtime in production only.

## The policy, not just the switch

A flag system alone does not stop rot. `docs/versioning/SCOPE.md` is checked into
the repository and enforced in CI, and it defines per surface what is shown, what
is hidden-but-kept, and what the replacement is.

The rule that does the most work:

> Never show a dead "Buy" button. Where a buy-ticket CTA used to live, replace it
> with organizer contact, an external link, or a "How to attend" block.

Hiding a feature is not deleting a call to action. A greyed-out or non-functional
purchase button reads as a broken product, which costs more trust than the
missing feature does.

The document also specifies hide-versus-delete per surface, so the decision is
made once in review rather than argued in every pull request. Mobile hides the
wallet tab, order screens, payment screens and the ticket QR flow, and keeps all
of the code. Organizer keeps ticket-type *pricing fields* visible, marked "not
yet sold via platform", because organizers want to publish prices even when the
platform is not collecting them.

## Cost

Honest accounting of what this choice costs:

- **Dead code paths are not exercised**, so they drift. There are 26 test files
  across the monorepo, which is not enough coverage to catch V2 regressions
  automatically. The V2 flip will need a deliberate re-verification pass, not just
  an environment variable change.
- **Two states to reason about** in any surface that touches orders or tickets.
  Every change to those areas has to be considered in both modes.
- **The offline check-in system has never run at a gate**, which is a direct
  consequence — it is V2 code, so it does not get real-world exposure until the
  flip.

What it buys is that V2 is a configuration change and a verification pass rather
than a build. Whether that trade was correct depends on how long V1 runs before
the flip; past roughly a year, option 1 probably wins.
