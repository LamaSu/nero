# Projector-nav convergence — adopt Jonny's `room_floor` contract (our side)

**Status:** decided 2026-07-18 (Sol, medium effort). Our working plan; the repo/team
moves below still need the team's nod.

## What happened to the repo
- The shared upstream `nimarez/nero` was **deleted or made private** (hard 404, no
  redirect). GitHub **re-rooted the fork network to `LamaSu/nero` (ours)** — we are now
  `isFork:false, parent:null`; `Jonnysol/nero` is now a fork *of us*.
- The old PRs (**ours #3**, Jonny's **#6/#7**) died with the base repo. Both projector-nav
  implementations are currently **unmerged branches on separate forks**; there is no
  shared PR flow until the team re-declares a canonical.

## The two overlapping implementations
| | **Jonny** (`agent/projector-navigation-contract`) | **Ours** (`mrhack/procam/`) |
|---|---|---|
| Frame | `room_floor` **metric** (m, origin floor-centre, +X right, +Y up) | metric floor, origin at tag centre, y-up |
| Geometry | **absolute** `robot_pose/goal_pose/trajectory` | **robot-relative** (bearing/range, goal_rel, path_rel) |
| Transport | HTTP + WebSocket @30 Hz (versioned JSON) | file `/run/nero/nav.json` |
| Data flow | operator-UI goal → **publishes to** Nima's ROS (`goal_pose`,`plan`) | **subscribes to** Nima's ROS → renders |
| Extras | full stack + tests + `docs/projector-navigation-contract.md` | ANSI barrier ring, multi-arrow path, deadman |

Jonny's is more complete and matches the locked team plan's control flow
("drag the circle to move the robot"). Both share: fail-closed on stale Vive,
`control_authority:"none"`, and a one-shot forward-yaw calibration (his
`POST /api/navigation/calibrate-forward` == our `--heading-offset`).

## Decision (Sol, option C)
1. **Jonny's `room_floor` contract is the canonical projector-nav.** Our
   file-transport overlay path is retired as the *primary*.
2. **Port only our unique value into `src/nero/projector/`:** the **ANSI
   safety-barrier ring sizing** (reach + stopping distance + uncertainty; auto-inflate
   on low pose confidence) and the **multi-arrow trajectory styling**.
3. **Keep our robot-relative converter as the documented fallback.**
   `nav_bridge.py` + `follow_circle.nav_rel_to_floor` still work when there is **no
   absolute Vive floor calibration / no SLAM↔floor alignment** — the one case Jonny's
   absolute contract can't cover. `follow_circle.py` also remains our standalone
   swaybg fallback demo.

## `frame_bridge.py` — SUPERSEDED
It bridged our metric floor ↔ Jonny's **normalized `[0,1]²`** frame (from his PR #7).
He has since moved his renderer to the **metric** `room_floor` frame, so the
metric↔normalized seam is no longer needed. Kept only for history / if a normalized
renderer ever returns.

## Repo move (proposed to the team — not yet executed)
`LamaSu/nero` is already the fork-network root. Proposal: make it the **canonical**,
merge Jonny's renderer branch into it, **protect `main`**, and have everyone re-fork
from there so there is one shared PR flow again.
