# Case Study 9: Woozy VTuber Character — Directed Build, Verified Live

**Roles:** Hunter directed scope, acceptance criteria, and approval decisions. Implementation and technical checks were performed through ChatGPT and the agent/tool workflow described below; they are not attributed to Hunter as personally executed modeling, code, or commands.

**Evidence:** This account summarizes the review package `Woozy-Muse-Portfolio-Handoff-2026-10-17` (21 files, integrity-verified by Archer on receipt — all matched) and the v1-vs-v2 demo video Hunter supplied on 2026-10-17. Recorded verification results below are from the October builds, not re-run for this write-up, except where noted.

**Version 1 → Version 2** (frames captured from Hunter's v1-vs-v2 comparison video, 2026-10-17):

![Woozy version 1 — flat matte green slime](screenshots/v1.jpg)
*Version 1: flat matte shading, face pasted on.*

![Woozy version 2 — glossy volumetric slime](screenshots/v2.jpg)
*Version 2: glossy specular response, integrated features, rounded form.*

**Recorded outcome:** a live webcam-driven slime VTuber character — glossy green body, live eyes and mouth, integrated goo brows, timed slime-flow deformation with exact recovery — accepted by Hunter on October 11, 2026 and streaming live with four independently selectable looks.

## Problem

Integrate recognizable live facial features into a stylized slime character while preserving its approved silhouette and movement — on a body that deforms, under webcam tracking that drops and reacquires, framed for OBS streaming. The character had to stay a slime first and a face second: every facial and motion decision served the creature, not the other way around.

## Decisions I made

| Options reviewed | My choice and reasoning | AI execution under that direction |
| --- | --- | --- |
| Separate falling slime ball and crown impact effects, or slow left-side body flow with a lowering anatomical left eye. | The flow. Detached droplet effects read as particle tricks; the slow deforming flow reads as the slime itself moving. Rejected both effects outright. | Remove the droplet/crown effects; implement the heavy, slow left-side flow tied to the left eye lowering, with the 6.6-second return-to-neutral cycle. |
| Brows as separate attachments, or integrated goo brows. | Integrated. Separate brow meshes would fight the deforming body every time the slime moves; brows made of the same goo move with it. | Build the brows as part of the goo body with their own morph behavior. |
| Visible nose or steering control, or an invisible steering nose. | Invisible. A visible nose breaks the clean slime face; the steering function stays, the geometry doesn't show. | Implement nose steering without visible nose geometry. |
| Whole-body head following, or upper-head following with a planted base. | Upper-head follows, base stays planted. A slime that slides around its scene looks unanchored; the planted base keeps it grounded while the head tracks the performer. | Separate upper-head motion from the base transform; verify the base holds through head turns. |
| Flat matte v1 look, or glossy volumetric v2 rebuild. | The v2 rebuild. The demo video shows why: v1 reads as a flat sticker, v2 as a physical little creature with specular response and integrated features. | Rebuild the body shading and form; preserve the approved silhouette and proportions. |

My contribution was the character direction: what Woozy is, what it is not, and which version of it ships. Modeling, runtime code, and verification were performed through the AI workflow; I reviewed options live, rejected what didn't serve the character, and accepted the October 11 result.

## What I directed and decided

I set the character contract — glossy green slime, rounded head, live webcam eyes and mouth, integrated goo brows, invisible nose steering, upper-head following with a planted base — and the behavior contract: heavy slow flow lowers the anatomical left eye, no separate falling droplet, the cycle returns to neutral at 6.6 seconds, tracking loss cancels with a 0.85-second return, reacquisition does not resume the cycle, and automatic cycles wait 45–90 valid tracked seconds after recovery. Four stream looks stay independently selectable (Clean, Helmet, Goggles, Full Tactical) with Clean startup and tap-to-recenter. My acceptance on October 11, in my own words: "looks really good lets make sure to presevre this as the new character document eveything and place it eith the other woozy work."

## How it was verified

- **Acceptance on a hash-verified payload:** the October 11 checkpoint's 70 files matched their SHA-256 manifest with zero mismatches, and the current runtime matched its recorded finalSave digest.
- **Live observations recorded:** the four look controls, helmet and goggles following head tilt, OBS rendering the combined look with controls excluded, and saved scene readback retaining state.
- **Fitted accessories, measured:** the goggles and strap were fitted against the deforming body across 25 sampled morph combinations and 975 contact samples, with approximately 0.55 cm minimum sampled strap clearance.
- **Package integrity re-verified by Archer on 2026-10-17:** all 21 top-level review-package files matched; every nested archive CRC passed.
- **Side-by-side screenshots** (frames from Hunter's v1-vs-v2 comparison video, 2026-10-17) show the evolution directly: flat matte v1 to glossy volumetric v2.

## Honest limits

- Recorded verification results are historical; they were not re-run for this write-up, except the package-integrity re-verification noted above.
- There is no end-to-end FPS benchmark. A 13 ms figure in the build records is a Node VM calculation, not an OBS/Lens frame time — it is not presented as one here.
- Strap clearance is sampled across poses, not a proof for every possible pose; extreme camera angles are unobserved.
- Code and geometry were AI-assisted. Hunter's contribution is direction, decisions, live review, and acceptance — not hand-written code or hand-modeled geometry.
- Two original attachments (the standalone V2 handoff ZIP and a standalone runtime JS) were lost to failed downloads and are not claimed as recovered; later archives contain the usable material.

## Evidence references (local or private unless linked)

- Review package: `Woozy-Muse-Portfolio-Handoff-2026-10-17` (Hunter's files; brief, evidence map, baseline readings, original project files)
- Accepted character: `Woozy-Character-Accepted-2026-10-11.zip` (frozen source authority)
- Screenshots: `screenshots/v1.jpg` and `screenshots/v2.jpg` (frames captured from Hunter's v1-vs-v2 comparison video, 2026-10-17)
