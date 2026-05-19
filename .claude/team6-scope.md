# Sub-team 6 — Desktop GUI & Client Shell — scratch context

This file is for Claude's working notes about our sub-team's slice. Long-lived team facts belong in the root `CLAUDE.md`. Use this for evolving decisions, ADR seeds, and meeting outcomes that aren't ready to publish yet.

## Decision log (working)

| Date | Decision | Status | Rationale |
|---|---|---|---|
| 2026-05-19 | Adopt MVVM split (View / ViewModel / Service Gateway) for client shell | accepted (per Section 6.6) | Required by spec; gives us a testable ViewModel layer with no Unity dependency. |
| 2026-05-19 | Primary worked example = File tab (native-call → ViewModel command via gateway) | accepted (per Section 6.6) | Highest visible impact; touches direct native-plugin coupling and file I/O that "belongs server-side". |
| 2026-05-19 | Second worked example = Debug tab (Observer of structured log stream) | accepted (per Section 6.6) | Demonstrates Observer pattern + push-style data flow; small surface so we can finish it. |

## Open questions to resolve with Sub-team 1

- Service Gateway transport: JSON-RPC over named pipes locally. **What is the contract surface for the File-load use case?** (e.g. `OpenCube(path, hdu, subsetBounds) -> CubeHandle` shape and error model.)
- Lifetime model: who owns `CubeHandle` server-side? How does the client release it?
- Versioning: is the gateway versioned independently of the plug-in ABI?

## Open questions to resolve with Sub-team 4

- Where is the menu-state contract for the **VR quick-menu ↔ Desktop panel state** sync? Today `CanvassDesktop` reaches into `QuickMenuController` and `PaintMenuController` directly — both need an interface seam.

## CanvassDesktop.cs — first-pass smell inventory

(Snapshot of the current file — verify before quoting in the deliverable.)

- **God class:** ~1900 lines, single MonoBehaviour, 30+ public/serialized fields.
- **Hardwired scene paths:** dozens of `transform.Find("A/B/C/...").GetComponent<...>()` chains in `Start()`. Untestable; every prefab rename breaks the script.
- **Service location via singletons:** `FindObjectOfType<VolumeInputController>()`, `FindObjectOfType<VolumeCommandController>()`, `FindObjectOfType<HistogramHelper>()`.
- **Mixed concerns:** subset-bounds maths, FITS axis dropdown wiring, popup state, threshold sliders, file dialog, coroutine lifecycle — all in one class. SRP violation by inspection.
- **Direct native-plugin coupling:** `using System.Runtime.InteropServices;` in a UI script is a smell — file/mask I/O is reaching native code from the View layer.

These are the raw inputs for the **Day 2 baseline benchmark** and the **before/after CK delta** on the File tab worked example.

## Day-by-day plan (Sprint 1: 18–22 May)

| Day | Date | Sub-team 6 focus |
|---|---|---|
| 1 | Mon 18 May | (DONE) Codebase running. Read assignment + Section 6.6 brief. Sub-team kick-off. |
| 2 | Tue 19 May | **Sprint 1 plan.** Baseline benchmark of `Assets/Scripts/UI/` + `Assets/Scripts/Menu/` (CK + SonarQube). CI green. Draft requirements doc. |
| 3 | Wed 20 May | Concern mapping of `CanvassDesktop`. Initial scope agreed. Seed first ADRs (MVVM split; gateway transport choice). |
| 4 | Thu 21 May | Requirements doc drafted (NFRs traced to ISO 25010 sub-characteristics). First interface contracts proposed to Sub-team 1. |
| 5 | Fri 22 May | Sprint review + retro. Sprint 2 plan committed. |

## Risks / watch-list

- **Bus factor on the gateway contract.** If Sub-team 1's gateway interface slips, our File-tab worked example slips with it. Mitigation: stub the gateway interface ourselves on Day 3 with a placeholder; iterate when Sub-team 1 publishes theirs.
- **Unity 6 UI Toolkit unfamiliarity.** None of the existing GUI uses UI Toolkit — it's all `UnityEngine.UI` Canvas. Our "after" diagrams must show the UI Toolkit equivalents. Set aside time in Sprint 2 for a UI Toolkit mini-spike.
- **Scope creep.** It is tempting to refactor the whole `Assets/Scripts/UI/` tree. Spec only requires **two** worked examples. Stop at two; everything else lives in the architecture doc as "applies analogously".
