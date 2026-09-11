# cloud-itonami-isco-7321

Open Occupation Blueprint for **ISCO-08 7321**: Pre-press Technicians.

This repository designs a forkable OSS business for a pre-press shop scheduling and logistics coordination practice: a workshop scheduling and supply-coordination robot manages crew/task records under a governor-gated actor, so a pre-press crew keeps its own operating records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/prepresstech/` implements the
`PrepressTechActor` as a `langgraph.graph/state-graph`
(`prepresstech.actor`) wired to a `Pre-press Technician Advisor`
(`prepresstech.advisor`) and an independent `PrepressTechGovernor`
(`prepresstech.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok?) +-> :request-approval (:escalate?, human-in-the-loop interrupt)
+-> :hold (:hard?)`. 22 tests / 47 assertions green (`kbb -M:test`).
HARD invariants (always hold, never overridable):
worker provenance, workshop provenance, no-actuation (`:effect` must
be `:propose`), a closed op-allowlist (`:log-work-record`,
`:schedule-crew-operation`, `:flag-safety-concern`,
`:coordinate-supply-order` — nothing else may ever be proposed), and a
permanent, unconditional block on any proposal that would directly
finalize a plate-preparation-execution decision (e.g. deciding to
proceed with a specific plate-preparation or plate-processing
procedure) or override a shop safety officer's judgment. Always-escalate paths (human sign-off regardless
of confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)):
`:flag-safety-concern` (always) and `:coordinate-supply-order` above
the registered cost threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a workshop scheduling/logistics coordination robot performs crew scheduling, job/materials-usage/progress-record logging and plate/imaging-materials supply-order coordination for a pre-press crew, under an actor that proposes actions and an independent **Pre-press Technician Governor** that gates them. The governor never
dispatches hardware itself, never performs plate-preparation or plate-processing work on the shop floor, and never finalizes a plate-preparation-execution decision or overrides a shop safety officer's judgment; `:high`/`:safety-critical` actions (such as a flagged chemical-exposure/equipment-condition concern, or an above-threshold supply order) require human sign-off. **This actor coordinates shop scheduling/logistics only — it never performs plate-preparation or plate-processing work itself.**

## Core Contract

```text
crew roster + workshop registration + safety-reporting policy
        |
        v
Pre-press Technician Advisor -> Pre-press Technician Governor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, finalize
a plate-preparation-execution decision, override a shop safety officer's judgment, suppress an operating record, or disclose sensitive data without governor
approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7321`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
