# YOLKSBOX

A physics sandbox about Stanley: a man with bones, blood and organs who is trying very hard to
stand up.

**Status: M1 ("he stands up") is incomplete.** The skeleton is physically correct and the motors
drive it, but the body does not yet hold a standing pose. The acceptance suite measures that
failure rather than hiding it.

## What exists

| Area | State |
|---|---|
| Engine, project spine, 60 Hz fixed timestep (Godot 4.7.2 + Jolt) | working |
| Data contracts (`data/balance/*.json`), validated at boot and in CI | working |
| 22-body / 21-joint skeleton, collision, ground probes, support polygon | working |
| Motor system (subtree-inertia velocity servo, torque-limited, gravity feed-forward) | driving, not yet stable |
| Get-up state machine + three-layer balance controller | implemented, not yet standing |
| Medical simulation (blood, consciousness, concussion, fracture, organs) | implemented, feeds the controller |
| Gore, weapons, maps, UI, stores | not started (M2+) |

## Measured, not claimed

`tests/run_tests.gd` (20 trials per statistical test, 5.1 s wall):

| Test | Result |
|---|---|
| `data_schema` | PASS — 22 bones, 21 joints, 11 motors |
| `stand_from_prone` | FAIL — 0/20 stood (target 90%) |
| `stand_from_prone_one_leg` | FAIL — 0/20 |
| `no_jitter` | FAIL — body not standing, so not scoreable |
| `stumble_recovery` | FAIL — 0/20 |
| `shotgun_knockdown` | FAIL — collapsed before the shot |
| `unconscious_limp` | FAIL — motors do not release within one tick |
| `zero_g` | PASS — transforms finite, peak 2.9 m/s |
| `dismember_crawl` | SKIP — needs M2 |

Nothing is claimed as working that is not on that list.

## Requirements

- Godot **4.7.2** (not vendored; `tools/setup_engine.sh` installs it without sudo)
- Python 3.10+ for the data lint

## Commands

```bash
python3 tools/lint_data.py                       # data schema + secrets + hot-path TODOs
python3 tools/make_site.py                       # regenerate ggg.html (single-file build)
godot --path .                                   # run the game windowed
godot --headless --fixed-fps 60 --path . --script tests/run_tests.gd -- --trials=100
tools/build_all.sh                               # lint, test, export every preset
```

`--fixed-fps 60` detaches the simulation from wall-clock time, which is what makes a 100-trial
acceptance run practical. It is a property of the test harness, not a performance claim.

## The bug that blocked M1 for days

Joints were never bound. Two silent engine behaviours:

1. `joint.node_b` resolves relative to the joint's **parent body**, so a sibling bone is
   `../../bone_x`, not `../bone_x`.
2. `joint.node_a` defaults to the **world**, not the parent body — left unset, every child bone is
   nailed to its spawn position while the parent falls.

An unresolved `node_b` logs nothing; the joint is simply inert. Both are now set explicitly and
verified to resolve (`docs/DECISIONS.md`).

## Honest limits

- No device has run this. No frame rate has been measured on hardware.
- Nothing has been rendered in the engine: the build environment has no GPU or display.
- No export has been produced (Godot export templates are not installed there).
- Gore, weapons, maps, UI and stores do not exist yet. Writing their data files before their
  systems would be a lie in JSON form.
- The project licence is **unchosen**; absent one, copyright is all rights reserved.
