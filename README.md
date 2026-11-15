**English** · [中文](README.zh-CN.md)

# Jayden Wang

<img src="assets/guided-run.jpg" align="right" width="34%" alt="At a guide-running training in a park, a runner with covered eyes runs beside a guide on a brick path, with more people running behind them">

I'm Jayden, a high school student at SCIE in Shenzhen. I built an app that matches blind runners
with volunteer guides, and I'm the first author of a paper on why AI agents fail at multi-step
plans. I also compete in algorithms and math: USACO Gold, AIME qualifier.

[Portfolio: jaydenw.me](https://www.jaydenw.me) · [Résumé (PDF)](https://www.jaydenw.me/Jayden_Wang_Resume.pdf) · zw6963470@gmail.com

| Project and my part | Why I started, and where it is now |
|---|---|
| [GuideRun · 助盲跑](#guiderun--助盲跑)<br>I designed and built the backend on my own, took over the iOS app from a teammate and ported it to Android | I guide blind runners twice a month. They told me how rarely they get to run.<br>**Now:** 30+ blind runners and 30+ guides finished internal testing. The iOS app is in App Store review |
| [AI planning paper](#ai-agents-that-plan-in-minecraft)<br>First author | I wanted a Minecraft assistant, and it kept losing its plans.<br>**Now:** accepted at a NeurIPS 2026 workshop, and in review at AAAI-27 |
| [Contrast-free CT](#contrast-free-ct)<br>All of the research and code are mine. West China Hospital provided the scans | Some patients cannot safely receive the dye a contrast CT needs.<br>**Now:** ongoing. The model failed on patients it had never seen |
| [SafePath](#safepath)<br>Built it alone | My first time alone in a foreign city, I chose walking routes by instinct.<br>**Now:** [live demo](https://safepath-black.vercel.app). It earned college credit |

---

## GuideRun · 助盲跑

I run. At a road race in Shenzhen I met blind runners doing the same, and I became a volunteer
guide. Twice a month I run tethered to a blind runner. On those runs I heard how rarely they
get to run. Their clubs had runners and had volunteers, but finding a guide still took phone
calls and group chats. So I offered to build the app. I started in October 2025. It was first
called BlindRun.

<a href="assets/guiderun-demo.mp4"><img src="assets/guiderun-demo.gif" width="640" alt="Screen recording of the app with English subtitles: booking a run, the system finding a guide, the phone call, and the run itself"></a>

This is part of the 92-second demo I showed at COSCon'26. I added English subtitles. [Watch all of it](assets/guiderun-demo.mp4).

<img src="assets/guiderun-screens.jpg" width="820" alt="Four screens of the app with English labels: booking by place, date and time slot; a guide has accepted and the runner is asked to call first; during a run, with a button that reads the numbers aloud; the help screen with location, emergency contact, 120 and 110">

I got several things wrong before the testers put me right. An order-grabbing mode would
have left runners in remote areas waiting forever, so I replaced it with system dispatch. A
booking form seemed the obvious choice. Only when I closed my eyes and tried it did I see how
hard a form is without the screen, and one blind tester got stuck in it. Our first voice
version misheard a runner and booked anyway, without saying what it had heard. Now the runner
picks three things and the rest is settled in a phone call. The testers' top complaint was one
I had never thought of: they struggled to find each other at the meeting spot. That is why
the call comes before every run.

Three designs I threw out:

- Matching. My first design offered a run to one guide at a time, best match first, with 30 seconds to answer. I had ruled out a broadcast to every nearby guide, because the fastest thumb would beat the best match. It didn't survive the way people book. Runs get booked a day ahead, so the answer window had to grow to as long as an hour, and at an hour each, asking ten guides takes ten hours. Now the system invites the three best-matched guides together.
- Two guides, one run. With three guides invited, two accepts can land milliseconds apart. My first guard was a version column with a loop that retried a rejected write three times. A test showed the loop had never once worked: a retry inside one transaction rereads the same stale copy of the row. Now each accept takes a row lock on the order, and the second guide is told the run is taken.
- Emergencies. My first version notified the guide, waited 30 seconds for a confirmation, and only then reached the runner's emergency contacts. A runner who had actually fallen would have a family that heard nothing for those 30 seconds. Now the text to the emergency contacts waits for no one, and the guide is told in parallel.

<details>
<summary>More engineering notes</summary>

- Every control has a spoken label for VoiceOver. Touch targets are 64 pt, and Apple's minimum is 44.
- SOS sends only with a fresh GPS fix. The app never says "your family has been notified", because the SMS goes out later and can still fail.
- A runner's phone number is shown masked, because VoiceOver reads the screen aloud in public.
- SOS messages have their own thread pool, so a slow SMS provider cannot make them wait behind normal traffic.

</details>

<table>
<tr><td><b>Status</b></td><td>30+ blind runners and 30+ volunteer guides have finished internal testing. The iOS app has been submitted for App Store review</td></tr>
<tr><td><b>Partner</b></td><td>Shenzhen Association of the Blind</td></tr>
<tr><td><b>My role</b></td><td>I designed and built the backend on my own, from the first commit. A teammate started the iOS app and I took it over in July 2026. I ported it to Android</td></tr>
<tr><td><b>How I keep it correct</b></td><td>2,596 automated tests on the backend. One API spec for all three apps, and CI fails if an app calls an endpoint that is not in it. UI tests run Apple's accessibility audit on the main screens</td></tr>
<tr><td><b>Stack</b></td><td>Backend in Java, Spring Boot, MySQL and Redis. <a href="https://github.com/Jayden23018/guiderun-ios">iOS</a> in Swift and SwiftUI. Android in Kotlin and Compose. I work with Claude Code as a coding assistant, and I read every change before it merges</td></tr>
<tr><td><b>Read more</b></td><td><a href="https://www.jaydenw.me/work/blindrun">Case study</a> · <a href="https://jayden23018.github.io/blindrun-legal/">Privacy policy and user agreement</a>. The backend and Android code are private</td></tr>
</table>

---

## AI agents that plan in Minecraft

I wanted an assistant to spare me the boring parts of Minecraft, like collecting the same
resources again and again. A small model on my own computer kept losing its plan halfway.
I wanted to know why.

We asked 7 language models to write plans for a Minecraft agent. A rule-based checker walked
every plan step by step. We compared plans written with and without worked examples.

- Tasks that need a two-step chain of materials passed only 10–30% of the time without examples. Single-material tasks passed 45–95%.
- Any example of the right kind closed most of the gap (+31 percentage points). Picking the best example added less (+12), and our planned statistical test did not confirm it.
- Models copy mistakes from their examples. One invalid command showed up in 71.5% of the plans that saw it, and in none of the 505 plans that didn't.

I had expected a smart example picker to let a small model match a large one. The data said no
(28% against 50%), so we rebuilt the paper around what the data did support. Midway we found a
one-line bug that had switched off three of the picker's four signals. We fixed it and re-ran
those experiments.

<table>
<tr><td><b>Status</b></td><td>Accepted at the NEmo workshop, NeurIPS 2026. In review at AAAI-27</td></tr>
<tr><td><b>My role</b></td><td>First author, advised by Dr. Li Ning (Stellaris AI). I designed the method and built the evaluation. Ziming Dai built the part where the bots work together</td></tr>
<tr><td><b>Read more</b></td><td><a href="https://github.com/Jayden23018/existence-before-relevance">Code and data</a> · <a href="https://www.jaydenw.me/work/llm-planning">Case study</a></td></tr>
</table>

---

## Contrast-free CT

Contrast-enhanced CT normally requires a dye injection that some patients cannot safely
receive. With scans from West China Hospital, I tried to generate these images from plain
scans instead, as an aid for the doctor reading them.

Most papers I read used structural similarity to judge such a model, so I did the same. After
one experiment the score was high. When I opened the generated image, the highlights were in
all the wrong places. A plain scan and a contrast scan already look alike in structure, so the
score could only confirm that. I wrote a new measure that checks whether the highlights are in
the right place, and I tested the model patient by patient.

On patients the model had never seen, the correlation between generated and real contrast was
r = 0.31 for the chest and 0.18 for the abdomen. My target was 0.5. No patient reached it.
All 49 patients came from one hospital, so I still do not know how it would do elsewhere.

<details>
<summary>Engineering notes</summary>

- An early score of r = 0.61 came from 5 patients. Changing only which slices were sampled moved the same model from 0.61 to 0.40. I now version every metric.
- The two scans are taken at different moments and do not line up. A reference with perfect anatomy but average brightness reaches only r ≈ 0.38 on the chest, so part of the gap is misalignment and not the model.
- Data that leaves the hospital machine is pseudonymized and date-shifted. No patient images are published.

</details>

<table>
<tr><td><b>Status</b></td><td>Ongoing research collaboration with West China Hospital, Sichuan University. No paper yet</td></tr>
<tr><td><b>My role</b></td><td>The hospital provided the scans and the clinical question. The research and all of the code are mine. I started in August 2025 with a GAN and later moved to a diffusion model</td></tr>
<tr><td><b>Code</b></td><td>Private, because of patient data. Python, PyTorch, MONAI</td></tr>
<tr><td><b>Read more</b></td><td><a href="https://www.jaydenw.me/work/virtual-cect">Case study</a></td></tr>
</table>

---

## SafePath

<img src="assets/safepath-routes.jpg" width="640" alt="Map near Penn. The shortest route is blue, the safest route is green, and high-risk streets are red">

In summer 2026 I was in a foreign city on my own for the first time. I picked my walking routes
by instinct, and I had no way to tell whether the instinct was any good. Map apps answer
"shortest". I wanted one that also answers "safest".

Pick a start and an end near Penn. SafePath draws both routes on one map and tells you what the
safer one costs. On the walk from Penn Park to Mill Creek, the shortest route crosses 10
high-risk blocks. The safe route crosses none and is 107 m longer.

<details>
<summary>How it works</summary>

- The street network comes from OpenStreetMap. The risk score comes from 106,881 incidents recorded by the city of Philadelphia.
- Dijkstra's algorithm runs twice, once on distance and once on distance weighted by risk.
- Raw crime counts made short crosswalks look dangerous just for being short. Ranking by percentile fixed that.
- The README lists 13 limits. The first is "historical crime is not a prediction".

</details>

<table>
<tr><td><b>Status</b></td><td>Live. Anyone can open the <a href="https://safepath-black.vercel.app">demo</a> and plan a walk</td></tr>
<tr><td><b>My role</b></td><td>Solo final project at Penn's Engineering Summer Academy. It earned college credit and a certificate</td></tr>
<tr><td><b>Read more</b></td><td><a href="https://github.com/Jayden23018/safepath">Code</a> · <a href="https://www.jaydenw.me/work/safepath">Case study</a></td></tr>
</table>

---

## Honors

- USACO Gold, 2026. USA Computing Olympiad, Gold division.
- AIME qualifier, 2025. American Invitational Mathematics Examination, by invitation, for roughly the top 5% of AMC scorers.
- AMC 12, top 5% nationally, 2024.
- Silver, Senior Physics Challenge, 2025. Run by the British Physics Olympiad.

## Talks

- COSCon'26, the 11th China Open Source Conference, Hangzhou, November 2026. I gave a talk of about 30 minutes on GuideRun at the Youth Open Source Forum. I also sat on the main-forum roundtable on the next generation of open-source talent ([announcement, in Chinese](https://www.sohu.com/a/1100311889_115128)).
- COSCon'25, the 10th China Open Source Conference, Beijing. I was invited to the Youth Open Source Forum with Ziming Dai, and presented the method from my paper. [Talk page](https://www.jaydenw.me/work/coscon25)

## Also

- [Physics demo](https://github.com/Jayden23018/Physic-demo), a double-pendulum simulation you can play with, with an AI tutor. React and TypeScript.
- [jaydenw.me](https://www.jaydenw.me), my portfolio site. I wrote it by hand in Astro.
