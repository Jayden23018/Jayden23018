**English** · [中文](README.zh-CN.md)

# Jayden

Prospective undergraduate, interested in information systems and in computer science where
software meets hardware. I build software for people who can't afford it failing:
an app that pairs blind runners with volunteer guides, LLM agents that plan in Minecraft,
and a walking-safety router for Philadelphia.

Most of my work lives in private repositories (a production service and unpublished research).
This page describes what is in them; links point only to public code.

---

## AidRun · 助盲跑 — running companions for blind runners

<p>
  <img src="assets/aidrun-home.jpg" width="190" alt="Runner home screen: the next booked run and a button to book a new one">
  <img src="assets/aidrun-voice-booking.jpg" width="190" alt="Voice booking screen: one large button to start speaking">
  <img src="assets/aidrun-running.jpg" width="190" alt="During a run: distance, time and pace, with buttons to tell the guide to slow down">
  <img src="assets/aidrun-safety.jpg" width="190" alt="Help and safety screen: announce location, call emergency contact, 120, 110">
</p>

A blind runner books a time and place — by voice or by form. The system matches a nearby,
identity-verified volunteer. Both sides then go through one order flow: an optional intro call,
departure, meeting up, the run, and a post-run record. During the run the runner has an SOS
button — confirmed or held for 3 seconds, so it can't fire by accident — that sends their live location.

**What I built** — three apps against one API contract, April – October 2026:

| Part | Stack | My commits | Merged PRs | Automated tests |
|---|---|---|---|---|
| iOS app | Swift, SwiftUI, AMap, AVSpeech / Speech, ActivityKit | 533 of 611 | 285 | 1,884 |
| Backend | Java 17, Spring Boot 3.5, MySQL, Redis, WebSocket | 969 of 988 | 447 | 2,082 |
| Android app | Kotlin, Jetpack Compose, Retrofit | 93 of 98 | 88 | 3,458 |

The backend is mine from the first commit. The iOS app started as a teammate's prototype;
I took it over in July 2026. The backend runs on Alibaba Cloud. The
[privacy policy and user agreement](https://jayden23018.github.io/blindrun-legal/) are public.
The app is not on the App Store yet.

**Engineering decisions I'm proud of**

- **Accessibility first, not last.** Every runner-side screen is designed for VoiceOver:
  64 pt touch targets (Apple's minimum is 44), Dynamic Type support, and spoken announcements
  when the order changes state. UI tests run Apple's accessibility audit on the main screens on a real iPhone.
- **Voice booking with an LLM, but not trusting it.** A runner says
  "tomorrow 7 am at the park's south gate". The backend extracts the slots with Qwen;
  the app reads them back and asks for confirmation before anything is booked.
- **A 12-state order machine** with explicit rules for who may cancel in which state,
  including an intro phone call before a runner's first run with a new volunteer.
- **SOS that never lies.** It only fires with a fresh GPS fix, and it never tells a blind user
  "your family has been notified" — the SMS is sent asynchronously and could still fail.
- **One contract, checked by machines.** The OpenAPI spec lives in the backend repo. The iOS
  API client is generated from it. CI fails if the app calls an endpoint that isn't in the spec,
  or if a backend change would break existing clients.

Code: [iOS (public)](https://github.com/Jayden23018/blind-run-ios) · backend and Android are private.

---

## ImplicitEnhancer — helping LLM agents plan in Minecraft

Research project, **first author**. LLMs fail at long crafting chains in Minecraft because they lack
step-by-step procedural knowledge. We add a training-free retrieval layer (State-Aware Example
Retrieval, SAER) that picks examples based on the agent's current inventory.

- Evaluated on **7 models × 60 held-out tasks × 3 seeds**. Without retrieval, 6 of 7 models
  pass only 0–20% of tasks. With SAER, the improvement is statistically significant for
  5 of 7 models (McNemar exact test, Bonferroni-corrected).
- My hands-on work includes the evaluation pipeline — cross-validation scripts,
  the McNemar significance tables, and a replication panel on an open-weight model.
- Accepted as a poster at the NEmo workshop, NeurIPS 2026.

Stack: JavaScript (Node.js, Mineflayer), Python. Code is private.

---

## SafePath — safer walking routes around Penn

Solo final project for a University of Pennsylvania summer program. It overlays a crime-risk
score from about 107,000 historical incidents (OpenDataPhilly) and user-submitted sidewalk
reports on a walking graph of the 3.5 km around campus. It then runs Dijkstra twice — once
for the shortest route, once for the safest — and shows both.

Stack: Python, Flask, Pandas, OpenStreetMap, Leaflet.
[Code](https://github.com/Jayden23018/safepath) · [Live demo](https://safepath-black.vercel.app)

---

## Smaller projects

- **Physics demo** — interactive double-pendulum chaos simulation with an AI chat tutor.
  React, TypeScript, Vite.
- **Chinese slide generator** (private) — produces single-file, offline HTML decks with
  embedded subset fonts and vector PDF export, because generic tools break Chinese typesetting.
- **Personal site** — Astro, MDX, GSAP; my résumé is typeset in Typst.

---

## Tools I use day to day

Swift / SwiftUI · Java / Spring Boot · Kotlin / Compose · Python / PyTorch · TypeScript / React ·
MySQL · Redis · Git & GitHub Actions

## Contact

zw6963470@gmail.com
