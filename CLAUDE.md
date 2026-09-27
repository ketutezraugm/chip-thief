# Chain Jam Vol. 1 — project context

Handoff notes for a fresh session. Read this before touching anything; a lot of
what looks like an obvious improvement below has already been tried and rejected
for a stated reason.

## START HERE — status as of 2026-09-27: SUBMITTED

**The entry is in.** User submitted at jam.chain.wtf on 2026-09-27, deadline day. Nothing left to build
against a deadline — treat further work as ordinary polish, not a race.

**Live and public.** Deployed on Vercel at **https://chip-thief.vercel.app** (game at `/chip-thief.html`,
`/` redirects; `prototypes/vercel.json` adds the redirect + CORS on the manifest). Repo is **public**, at
`github.com/ketutezraugm/chip-thief` (see "Repo" below). Manifest has `assets.iconUrl/coverUrl`, the page
has `og:image`/`twitter:card`, manifest validates. Submitted URL: `https://chip-thief.vercel.app/chip-thief.html`,
declared RTP 96.0%.

**Deploy:** `cd prototypes && npx vercel deploy --prod --yes` (already logged in and linked; `.vercel/` is
untracked). No build step. The jam page says "same URL, newest build counts" — a redeploy after submission
still updates the live entry, no resubmission needed, but check with the user before pushing more changes:
the entry is now final in their mind unless they ask for something.

**Still open, not urgent:**
1. Ask the Chain team (discord.gg/3kpZHvvTq) whether they deploy/whitelist the contract themselves: it has
   only run on the local simulator chain, is unaudited and not on Base.

**Chrome/animation/packaging rebuild (2026-09-27, Claude Design "Chip Thief Chrome v3" + "Chip Thief Logo",
`Assets New 2/`, gitignored).** The brown cabinet + `<dialog>` Info from the previous pass is gone, replaced
wholesale by the vendored **`CT` chrome module** (`chip-thief-chrome.js`, inlined as a classic `<script>`
right before the game's module script) plus `chip-thief-chrome.css` (inlined in `<style>`). This is a real,
substantial rewrite of the DOM/CSS layer — read this whole section before touching the chrome again.

- **Everything diegetic, nothing boxed.** Balance is a mechanical drum counter (`#counter`/`.drums`), stake
  is a stack of clay chips on a brass tray with thumb levers (`#tray`/`.chip`), Release is a lacquer cap in a
  brass collar (`#go`). All of it sits in a `#floorStrip` over the carpet, no card/panel background anywhere.
  Sizing is one `--u` unit (`viewport height / 620`, JS-synced by `CT.syncUnit()`), so it scales exactly with
  the world art from 1280x720 to 2560x1080; portrait gets its own `--u` formula and the rail stacks into rows.
- **`CT` owns the chrome state machine.** Public API: `CT.init(opts)`, `setBalance`, `setStake`/`stepStake`,
  `beak(mult)` (the "in beak" figure + pop animation), `result(tier, {mult,stake,payout,net,applyBalance})`,
  `wallet(state, detail)`, `screen(id|null)`, `mute(bool)`. It also owns **every keyboard shortcut**
  (Space/Enter release-or-confirm, Up/Down stake, M mute, Escape close) and the three full-screen `.screen`
  overlays: **title** (shown once per `sessionStorage`, skipped entirely for a returning player who already
  has `ct.hinted` in `localStorage` — exactly the "no friction for returners" ask), **info** ("How to play":
  three art panels, keys, a stats strip CT tracks itself, then our own `#dbg` self-check and `#log` run log
  bolted into its `<details>`), and **broke** (balance < 50, "Reset balance"). `chip-thief.html`'s own script
  now only feeds `CT` numbers and gets a stake back via `onRelease(stake)` — it does not touch `#balance`,
  `#stakeVal`, the Release button, or any keydown handler directly any more.
- **Tier names changed:** CT's tiers are `caught`/`short`/`win`/`big`/`jackpot` (was `caught`/`partial`/`win`
  in the old `fillVerdict`). `settle()`/`presentBigWin()` now call `CT.result(tier, {...})` with those names.
- **One behavior change worth knowing:** CT's `release()` calls `ready()` (re-enables Release, un-dims the
  rail) as soon as `onRelease` is invoked, and `result()` never auto-hides the slab on a timer — it only
  hides on the *next* release (`hideSlab()`). So a fast player COULD mash Space through a big win's 1.5s hold
  before it's fully admired; accepted as-is (a real slot lets you spin again during a win too), not a bug.
  The canvas-side freeze (`enterHold`/`clockHold`, the ENHANCE inset, the jackpot-chip lift) is unaffected —
  those still hold the frame until the next `launch()` clears them.
- **Chain-mode balance never gets locally simulated**, same rule as before: `CT.result()`'s own optimistic
  balance tick is skipped with `applyBalance: !chainMode` (a small vendored addition to `result()` in
  `chip-thief-chrome.js` — search `applyBalance` there); the host's own snapshot via `CT.setBalance()` in
  `onHostSnapshot` stays authoritative. Wallet states surface through `CT.wallet('waiting'|'settling'|'failed', detail)` — a bottom-center inline line, never a modal, Release stays clickable throughout (a click while
  `!canBetOnChain()` is a silent no-op with `CT.wallet('waiting')` shown, matching the old silent-no-op guard).
- **`#statusLine` is CT's wallet line now** — the old incident-progress flavor text ("INCIDENT ACTIVE ·
  439m TO FIRE EXIT", "NO INCIDENT · 800m", "FRAME HOLD · INCIDENT 0413") is gone, on purpose: it was
  dev/operator readout, the same category of thing the first chrome pass already stripped from the sidebar.
- **The wordmark is real art now**, not text: a brass marquee sign (`#sign`, inline SVG, from
  `Assets New 2/export/logo/logo-compact.svg`) with a goose-head-on-a-chip emblem, top-left, swaying gently
  (`signSway`). The title screen gets the large version (`logo-title.svg`) with a bulb-chase + glint-sweep
  entrance (added by hand, not in the export — chrome.css shipped the `bulbChase` keyframe but no rule using
  it). **Both SVGs came with an ~8KB embedded C2PA provenance blob per `<metadata>` — stripped before use,
  same as the favicon before it.** Both files reuse the same internal ids (`brassUp`/`cream`/`chipG`/`w`/`t`)
  since they're the same artwork at two sizes; they're prefixed `lc-`/`lt-` before inlining so having both in
  one document doesn't collide (duplicate SVG ids resolve to whichever the browser finds first, silently
  breaking one of the two gradients — verify this again if a third instance of the logo is ever inlined).
- **Store images regenerated** from the new logo art (`Assets New 2/export/logo/*.png` re-encoded with ffmpeg
  to the same `prototypes/assets/` filenames: icon-512.webp ~12KB, cover-1600x900.webp ~17KB,
  social-1200x630.jpg ~41KB). Manifest/`og:image` already pointed at these filenames, no URL change needed.
- **Extra spectacle kept, retargeted:** the ENHANCE inset and the jackpot-chip digital-zoom lift
  (`#bwEnh`/`#lift`, `showEnhance()`/`startLift()`/`renderLift()`) are ours, not CT's — CT only ships the
  `#alarm` red border and the result slab. Both still work; sized in raw px against `--u` where it mattered.
  `#bwTop` is reused for both the jackpot's "Alarm · severity 1" text and the lift's "Digital zoom" caption.
- **A rendering bug from mixing the two systems, fixed:** `idleFrame()` used to gate the idle redraw on
  `!busy` alone; since `busy` now clears immediately in `settle()` (CT owns the slab's lifetime, not a
  `setTimeout`), the idle scene's own canvas-drawn reticle started redrawing underneath the still-visible
  slab. Fixed with one extra clause: `!busy && !heldFrame && !verdict.classList.contains('show')`. If the
  chrome changes again, re-check this — it's the seam between the two systems.
- **How-to-play thumbnails** (`.panel canvas`) call `CTW.backdrop()` at three scroll offsets (idle/mid-floor/
  near-exit) — deliberately NOT the full `world.js` scene compositor (goose/staff/chip drawing), which was
  trimmed out of the inlined `CTW` back when the art pass landed. Room-only thumbnails, no actors; a fine
  trade for three small "how to play" panels, not worth re-inlining a second copy of the actor-drawing code.
- **Verified:** idle, title (auto-shows once per session, skipped for returners), a normal run through win/
  short/caught, jackpot (alarm + enhance inset + lift, credit drum ticks correctly), How to play (panels,
  keys, live stats, self-check text, run log), broke screen + reset, at 1440x800 / 2560x1080 / 390x844, zero
  console errors. **Not verified:** chain mode against the local simulator (code changes were narrow —
  `CT.wallet(...)`, `CT.setBalance/setStake`, `applyBalance` — but not exercised live this session).

**Art pass (2026-09-26, Claude Design "Chip Thief World" + `Assets New/world.js`, gitignored).** The green CCTV
grade is GONE: the room is a warm casino (oxblood walls, brass, emerald felt, sodium lamp light) and the camera
survives only as light treatment (scanlines, vignette, REC dot, timecode, the dashed reticles/labels in paper
cream instead of green). `world.js` is inlined in `chip-thief.html` as `const CTW = (function(){...})()`
(only the layer/actor code the game uses). `drawWorld` calls `CTW.backdrop` (shell, far: coffers/chandeliers/
balcony/marquee signs, mid: slot banks/roulette/craps/palms/pillars, light shafts + dust, lane scrim, carpet),
`CTW.near` (rope stanchions + foreground table edge, drawn after staff/pickups and before the goose),
`CTW.exitSign`/`exitDoor`, and the world's `GOOSE`/`STAFF` poses. The game keeps its own legs (`gooseLegs`,
feet plant), chips, plaques, scenery, particles and every gameplay hook. Goose draw scale is `GS=.78`. New
'panic' pose while `pursuit<190`. Staff with no lunge pose lean forward instead. Exit sign hangs at y=118 so
it clears the marquee lettering.
**Performance rule:** the mid layer is hundreds of small draws (46 fps at 1440x800 on a real GPU when drawn
live, vs 120 before the art pass). `midCached` renders it into an offscreen strip ~8x/s and blits it with the
scroll offset every frame: back to ~100 fps. Measure with headless Chrome WITH the GPU (do not pass
`--disable-gpu`; software raster is 3-5x slower and misleading) and skip layers one at a time to bisect. Far,
shafts, floor are still drawn live. If it gets tight again, cache those the same way. Not verified on a real
low-end machine.
Portrait phones work but are cramped (hint text sits over the goose); not a priority.

**UI chrome (2026-09-26, "Chip Thief Chrome").** Full-bleed: the feed is `position:absolute; inset:0` (world
height fixed at 620, width follows the viewport), topbar and cabinet float over it, the cabinet sits in the
floor strip below the goose's feet, bottom strips are offset 104px above it. Cabinet strip: balance | stake
steppers + 50/100/500 presets | Release. Removed from the player's view: sidebar, camera/tape/lock readouts,
boxed first-run notes, the "CONTAINMENT FAILED" stamp. Operator stuff lives in the **Info** `<dialog>` (how
it works, keys, staff cuts, declared RTP, Maths and fairness = the self-check, Run log). First run shows one
line of guidance plus a "Press Space" pointer. Results are a bottom-left slab (tag, Bodoni figure, one deadpan
line, Stake/Payout/Net): gold only when net > 0, a <1x escape is cream + "Escaped · short". Normal results
fade after ~3.6s, BIG/JACKPOT stay until the next release. Tokens are the `:root` block in `chip-thief.html`.
Verified by screenshot at 1440x800, 2560x1080, 390x844 and via forced BIG/JACKPOT copies. Not re-verified:
chain mode inside the simulator after the restyle (bridge code untouched, only DOM ids/markup changed), and
the caught/partial slabs by eye.

**Environment gotchas.** jam.chain.wtf needs a VPN from the user's country. The local test stack
(`casino-sdk/`: `npm start` = chain :8545 + VRF + simulator :3300) is gitignored and re-downloadable.
Headless Chrome (`C:\Program Files\Google\Chrome\Application\chrome.exe`) is driven over raw CDP (Node's
global WebSocket; no Playwright). Windows + Git Bash: `pkill` doesn't stop processes; use `netstat -ano` +
`Stop-Process -Id`. gh is at `C:\Program Files\GitHub CLI\gh.exe` (not on PATH).

**How the user likes to work:** blunt honesty over agreeableness, no over-hedging; they catch real mistakes
by playing the game. Verify by playing/looking at screenshots, not just statistics.

## The jam

Build an original casino game on the free Chain casino SDK. Site: `jam.chain.wtf`,
SDK: `sdk.chain.wtf/casino`. Prizes 1,000 USDC (500/350/150) plus **25% lifetime
revenue share** on any entry integrated into chain.wtf, winner or not — the revenue
share is the real prize, so bets-per-session matters as much as placing.

**Deadline: extended to ~27 September 2026** (originally 20 September 23:59 UTC;
user confirmed the extension in session on 2026-09-21). Mention it once if
relevant; the user has explicitly asked not to have it drive every answer.

### Eligibility — pass/fail gates, not scoring
An entry is eligible only when it:
- Implements the Chain casino SDK exactly (**contract, bridge, manifest**)
- Runs correctly in the local simulator and loads near-instantly
- Has theoretical **RTP between 93–98%**, declared math matching the actual paytable
- Is recognisably a casino game and a novel concept — **no blackjack or classics,
  no plinko / dice / limbo / crash clones, no copies**
- Runs standalone as a playable demo outside the chain.wtf iframe
- Carries the jam widget and was submitted through jam.chain.wtf with source access

### Judging — four unweighted criteria
Novelty · Fun (still playing after 10 hours?) · Simplicity (no manual needed) ·
Visual & sound ("does it feel like a real game? No AI slop").

AI-generated code and assets are explicitly allowed.

## Repo

- `github.com/ketutezraugm/chip-thief` — **public**, branch `main`. Renamed 2026-09-27
  from `chain-jam-vol-1` (it was the multi-prototype repo before Chip Thief was the
  sole entry); GitHub redirects the old URL, but use the current one for the jam form.
- Remote is **HTTPS** (SSH failed host-key verification; `gh auth setup-git` is configured)
- `gh` CLI is at `C:\Program Files\GitHub CLI\gh.exe` — **not on PATH**, call it by full path
- Authenticated as `ketutezraugm`
- Local git identity is `AkuTampanTay <ezradarkwing@gmail.com>`, which does **not**
  match the GitHub account, so commits won't link to the profile

## Layout

```
prototypes/chip-thief.html        the game — single HTML file, no build step
prototypes/game.manifest.json     Chain casino SDK manifest
prototypes/vendor/                vendored penpal + keccak256 + JetBrains Mono, same-origin, no CDN
prototypes/assets/                store images: icon-512.webp, cover-1600x900.webp, social-1200x630.jpg,
                                   favicon.svg (web-sized: 15KB/48KB/103KB; the designer's PNGs were
                                   200/800/540KB). NOT yet referenced by the manifest or og:image —
                                   both need the final absolute URL, i.e. after deploy.
contract/ChipThiefGame.sol        on-chain game contract (ICasinoGameV2)
contract/ICasinoGameV2.sol        vendored copy of the SDK's canonical interface
casino-sdk/                       local SDK dev/test stack — gitignored, not part of
                                   the submission; re-download from sdk.chain.wtf if
                                   missing (see "SDK integration" below)
README.md                         written for judges/outside readers
```

The other five prototypes (`insider.html`, `closing-time.html`, `underwriter.html`,
`smuggler.html`, `ghost-auction.html`) and the `Asset/` Claude Design export were
deleted on 2026-09-21 once Chip Thief was confirmed as the sole entry — the user
still has the source Claude Design project if any of that is ever needed again.
Don't re-create references to them.

---

# Chip Thief — current state

Single-file canvas game. Art direction is **"CAM 04"**: the entire game is framed
as casino CCTV footage.

## Core loop
One button. Stake, release the goose, watch a ~10s run down the casino floor toward
a fire exit 800m away. It grabs chips, gets tackled by staff, and either makes the
door or gets detained. No decisions during the run.

## Maths — exact, do not break this
RTP **96.0%** (≈95.9% after the on-chain 100× cap), bust rate **22.3%**, **median escape
0.84×**, ~38% of escapes pay ≥1×, and a real top end: **1 in ~90 runs reach 5×, 1 in
~190 reach 10×, 1 in ~930 reach 20×**. Everything is closed form and verified against
200k simulated runs in the self-check panel (which prints the median-escape figure too).

```
chips = n·p·(1−λ)·v̄·(1−pλ)^(n−1)
mults = (1−q + q(1−λ)·m̄)^k
staff = (1−s·f̄)^j
EV    = chips · mults · staff
```

Current paytable (`RUN` in chip-thief.html, mirrored constant-for-constant in
`contract/ChipThiefGame.sol`): λ=0.024; 15 chip slots, each lands with p=0.68; chip
face values 0.5/1/2.5/5/**25 (plaque)**/**350 (jackpot)** at weights
.2800/.3500/.2700/.0894/.0100/.0006 (a chip pays face×0.2×scale: the plaque ≈1.3×
the stake, the jackpot ≈18×; the labels are honest); **2** multiplier slots, p=0.11,
tiers 2×/3×/4× at .70/.27/.03; 3 staff slots, p=0.15, cuts 20/30/40% at .45/.35/.20.

- `λ = 0.024` — per-leap chance the pursuer catches you. Every chip you reach for is
  another roll, so **greed is what gets you caught**. This is the *only* way to bust.
- Getting caught forfeits the **entire haul** (payout 0).
- Survival is shared across all leaps, but the indicators stay independent, which is
  why it still factorises exactly.

`RUN.scale` is solved from `RTP/rawEV(RUN)` at load — if you change any table, the
scalar re-solves and RTP holds automatically. Verify with the self-check panel. **But
the contract does not re-solve**: `SCALE_WAD`, `LAM_WAD`, and every 64-bit threshold in
`ChipThiefGame.sol` and in `chainOutcomeFromRandomness()` are baked literals derived
from these tables via exact integer bps math (`floor(bps·2^64/10000)`). Change a table
and you must regenerate them, redeploy, and re-run the bit-for-bit JS-vs-contract check.

### Why the paytable was rebalanced (2026-09-24) — two passes, the first overshot
Playtesters called the game "rigged". The RTP was never the problem — the *shape* was.
The original paytable had a median escape of **0.65×** (66% of escapes still lost
money), and the screen said "SUBJECT LEFT THE BUILDING" — an apparent win — while
taking the player's money; the end-screen label was also inverted (`won>=STAKE?'LOSS'
:'NET'` showed "LOSS" on a doubled stake).

*Pass 1* fixed the median (0.99×) by removing the flat μ=0.07 "final approach" gauntlet,
cutting to 2 gentle multiplier slots, and thinning the jackpot to nothing. **It made
the game feel MORE rigged.** Only 1 in ~1,660 runs reached 10× and the "jackpot" chip
paid ~1.5× — there was no reason to keep playing. Lesson: a median-only fix trades away
the tail, and the tail is what makes people accept losing. Never tune the median without
looking at P(≥5×) and P(≥10×).

*Pass 2* (current) gave the chip table a real jackpot (350-face chip ≈18×, 0.06% per
landed chip) and a plaque (≈1.3×), at the cost of median 0.99→0.84×. A jackpot has to be
paid for out of the fixed 96% RTP budget, so the two trade off directly — the scan of
candidates is what picked this point. `settle()` distinguishes caught / partial escape
(<1×, red "PARTIAL RECOVERY", "LOSS n") / real win (green, "NET +n"). **Do not re-add a
flat hazard, do not let an escape that pays <1× look like a win, and do not flatten the
tail again.**

*The guard* (pass-1 regression, fixed in pass 2): every near-miss and evade hop counted
as a "leap" that pulled the pursuer in, though those hops carry no catch roll, so the
guard hovered on the goose's heels all run (avg gap ~170–250px vs ~300px before, pinned
at its closest 10% of frames vs 0%) — reading as scripted doom. Now the gap closes only
on real pickups (each is a catch roll, so it tracks the actual risk), drifts back when
running level, and the scripted lunge before a capture is 420px, not 900px.

Known feel issue, deliberately not changed: a capture forfeits the whole haul, and half
of all captures land with ≥0.5× already in the beak (avg 0.65×) — each loss lands when
the haul looks best. There is NO hidden ramp (flat 2.4% per pickup; of hauls that pass 1×,
7% get caught later), but it feels like one. Option if it keeps coming up: keep a share
of the haul on capture (changes the whole paytable again).

The remaining complaint — "I can't control the goose" — is inherent: zero decisions is
the design (and any cash-out/steering mechanic risks the banned crash pattern). Don't
try to fix it with fake agency; see "Rejected".

## Architecture rule that cannot change
**The outcome is drawn from the seed first; the animation replays a decided result.**
The flight path is generated *after* the rolls, to pass through whatever was actually
won. This is the only way it works against an on-chain VRF, and it's how real slot
reels work. Never compute a result from animation state.

## Decisions already made — with reasons

**Bust is tied to an action, never to elapsed time.** No climbing multiplier, no
cash-out button, no memoryless per-second tick. This is deliberate and load-bearing:
it is what keeps the game clear of the banned crash-clone pattern while still having
real jeopardy. **Do not add a cash-out or a time-based multiplier** — either one makes
the entry ineligible.

**One speed only (TROT, 10s, ~8 leaps).** Four selectable speeds existed and were
removed. BERSERK (5.2s, ~2.9 leaps) was unreadable — pickups and staff resolved faster
than the telegraph system could show them, which kills excitement rather than raising
it. Simplicity is 25% of the score and a first-time judge had no basis to choose a mode.

**Session RTP is off the HUD**, in the self-check panel instead. Over a handful of runs
it is pure noise; showing a player "58%" tells them they're losing, and a judge seeing
187% or 60% will doubt the maths. HUD shows cumulative RECOVERED instead.

**The 93–98% RTP band legalises real skill.** Used in the now-deleted `insider.html`
and `underwriter.html` prototypes, where sloppy play returned ~93.5% and sharp play
~97.8%. Chip Thief has zero decisions, so its declared RTP holds for every player
with no caveat.

**Art direction is Cam 04 surveillance inside the feed, a warm cabinet around it, never neon.** A "NEON HEIST" asset sheet exists
in `Asset/Chip Thief Assets.dc.html` and was implemented then replaced. Reasons: the
magenta/cyan/yellow-on-purple palette is the default of every crypto game (bad for
Novelty and for "no AI slop"), it fought the comedy (neon is *cool*, the joke is a
goose), and the 0.5 chip at `#ff2d87` was invisible against the `#c11d5e` carpet.
Surveillance framing also *explains the architecture* — CCTV means you're watching
something that already happened, which is literally how the VRF works.

**Palette rule: the goose and the chips are the only warm objects in frame.** The room
is desaturated green-grey. This is what makes the eye track the subject with no effort;
don't add colour to the environment.

**Copy is deadpan incident-report language.** "CONTAINMENT FAILED", "SUBJECT LEFT THE
BUILDING", "ASSET RECOVERY: 0.00x". The system never acknowledges what it's reporting.
The gap between the flat language and the chaos on screen is the joke — no exclamation
marks, no wordplay, no winking.

## Dopamine work already done
Excitement comes from presentation, not variance — cranking variance just makes players
bust faster and the RTP display swing. These all raise perceived tension at identical EV:

- **Near-miss reach** — missed chips generate a partial leap so the goose visibly
  stretches and comes up short, with a beak-snap. Strongest single lever. **Every**
  miss gets one: a visible coin the goose ignores outright reads as a bug. (An old
  "skip the reach if a picked coin is within 200px" dedup caused exactly that; it was
  removed together with adding a 160px minimum spacing between all items, because the
  cluster jitter alone put >60% of items within 160px of a neighbour.)
- **The pursuer** — a staff member permanently in frame behind you; the gap closes on
  every *pickup* (not on near-miss/evade hops — those carry no risk) and drifts back
  when running level. Makes greed *visible*.
- **Threat-last staging** — one staff member is always dragged to within ~600–880px of
  the exit, so the closing seconds always put the full haul at risk. Costs nothing
  mathematically (the `got` roll happens before the reposition).
- **Time dilation** — eases to 0.42× when a jackpot, plaque, live multiplier or a
  connecting staff member is within 230px ahead.
  (There used to be a "FINAL APPROACH" banner for the removed μ gauntlet — gone.)
- **Clustered pickups** — 4–6 pockets rather than an even sprinkle, so runs have quiet
  stretches then a flurry (and bunched leaps collapse the gap fast). Items keep a
  160px minimum gap (`MIN_GAP` in `drawOutcome`) so no two ever overlap.
- **Staff telegraphed from 850px** plus edge-of-frame markers for threats off camera.
  Radio squelch fires ~260px before every contact as the only audio warning.

## Animation notes
The goose's legs are **drawn procedurally each frame**, not baked into the SVG. Feet
plant on the floor and slide back during stance, lift and reach forward during swing.
This was a real bug fix: trailing baked-in legs read as *flying* no matter what the
body did. Shelf heights are 440/384/316 against a floor at 520 and an approach of
`70 + lift×0.30`, so a grab is a ~200px hop, not a 600px glide.

Goose poses: `idle`, `sprint`, `beakFull`, `tackled`, `escape`. `sprint` and `beakFull`
carry `legs:true` and omit leg ops.

## Layout
Full-viewport grid, feed locked to **16:9** inside a centering stage capped at
`(100vh − 128px) × 16/9`, with a 268px system sidebar. World *height* is fixed at 620
and visible *width* follows the aspect ratio. Getting this wrong doesn't just overflow —
a too-tall feed **zooms in** and shows less floor.

---

# Eligibility checklist — status

All of it is pass/fail.

- [x] **SDK integration** — contract, bridge, manifest. Done and verified live
      against the local simulator (not just written — see below).
- [x] **Runs in the local simulator** — confirmed: opened a real session against
      the deployed contract through `?game=…&gameAddress=…`, watched it settle,
      balance updated correctly, no console errors from our code.
- [x] **Jam widget** embedded — `<script async src="https://jam.chain.wtf/widget.js">`
      in `chip-thief.html`'s `<head>`. Zero config (no game ID — it reports
      `location.pathname` and self-identifies from the submitted Game URL). It
      no-ops when embedded in an iframe (`window.top !== window.self`), so it
      only pings/heartbeats from the standalone page, never from inside
      chain.wtf's own iframe. Renders a small fixed bottom-right badge —
      confirmed by screenshot it doesn't overlap the RELEASE button or sidebar.
- [ ] **Hosted on own domain** — standalone mode works (confirmed in a real
      browser), but it isn't deployed anywhere yet; still local only. When it is:
      add `assets.iconUrl`/`coverUrl` (absolute URLs to `prototypes/assets/`) to
      `game.manifest.json`, and `og:image` + `twitter:card` (summary_large_image,
      social-1200x630.jpg) to the page head — none can be added before the domain is known.
      The host origin needs to serve `game.manifest.json` with CORS (the simulator fetches
      it cross-origin; most static hosts do by default).
- [ ] **Repo public** with source access — still private.
- [ ] **Submitted** via jam.chain.wtf — not started.
- [x] Self-host the JetBrains Mono woff2 — done: one 40KB variable woff2 in
      `prototypes/vendor/fonts/` (+ `OFL.txt`), preloaded, no Google Fonts request left.

## Load time — "loads near-instantly" is an eligibility gate (measured 2026-09-25)
Before: the RELEASE button appeared **~23s** after navigation, because all init lived in
`window.addEventListener('load')`, which waits for every third-party request (Google
Fonts, and the jam widget — jam.chain.wtf needed a VPN from the user's region). Plus the
200k-run self-check ran inline and froze the main thread ~1.6s. After: button in
**~130ms** (68ms with fonts+widget blocked), worst frame stall ~70ms, identical with the
network blocked. Rules that keep it that way:
- **Never gate init on `load`.** It's a module script, the DOM is parsed when it runs:
  init straight away.
- **No third-party request may block first paint** (font is self-hosted; the widget is
  `async` and nothing depends on it).
- Heavy work (the self-check) runs in 1,500-iteration slices via `setTimeout` after first
  paint; the panel shows "running… n%".
- To re-measure: headless Chrome + CDP, poll for `#controls button`, and block
  `*fonts.googleapis.com*`/`*jam.chain.wtf*` with `Network.setBlockedURLs`.

## Polish already in (2026-09-25)
Space/Enter releases the goose (more bets per minute), ↑/↓ or +/- change the stake, `M`
toggles sound (persisted in localStorage; while muted no AudioContext is even created);
`prefers-reduced-motion` disables the shake, scanline roll, glitch flashes and blinking
(the media block must stay LAST in the stylesheet or the earlier `animation:` rules win);
favicon (inline SVG placeholder — replace with the designed one), meta description + Open
Graph title/description (no `og:image` until the site has a real URL), aria labels on the
canvas / verdict / sound toggle. Mobile/portrait layout is deliberately NOT a priority —
nothing in the jam's rules or eligibility list requires it.

## Claude Design kit — what was built from it (2026-09-25)
The kit lives in the user's Claude Design project; exports go in `Asset/` (gitignored, so
they never reach the public repo). Built from it, all verified in a real browser:
- **Idle first frame**: static room with loose chips, two patrolling staff (±48px sine,
  7s), lit EXIT sign, "SPACE TO RELEASE" (or "TAP RELEASE" on coarse pointers).
  **Loose chips and ambient staff are idle-ONLY** (`drawIdleDressing`, gated on
  `run===idleRun`) and cut at release. The kit had them scrolling off with the run, but the
  goose runs straight through that stretch, so they'd read as coins it ignores — the
  same bug fixed earlier. Furniture and the EXIT sign (`DECOR`, `EXIT_SIGN_X`) do persist.
- **First-run operator notes** (3 callouts, `localStorage["chipthief.briefed"]`): Space
  dismisses AND releases, Enter/Esc/click dismiss only. Leaders are measured from the real
  HUD elements, so they stay attached at any feed size.
- **Big-win tiers**: BIG ≥5×, JACKPOT ≥10× (`presentBigWin`, hard cuts, ≤1.5s, overlay stays
  until the next release, controls re-enable at 1.5s, `revealOutcome` fires then). JACKPOT
  adds the alarm border, ENHANCE inset and redaction rows. **Jackpot-chip lift** (600ms
  digital-zoom hold) plays only when the pre-drawn payout is already ≥10× — never a tease.
  Design consequence to know about: that makes the lift a spoiler (seeing it means you have
  escaped), so the last ~3s of those runs carry no suspense. Deliberate, per the designer.
- Wordmark dot in the topbar; favicon inlined (the designer's SVG carried an ~8KB embedded
  provenance blob — stripped to 429 bytes); store images re-encoded to WebP/JPEG.
- Not built: auto-bet, run history, leaderboard (never in the kit, and not needed for
  eligibility).

**Testing the rare paths** (BIG/JACKPOT are ~1-in-90 / 1-in-190 runs, so don't wait for
them): make throwaway copies of the page in `prototypes/` (imports are relative) with the
demo payout forced — replace `Math.min(CAP_MULTIPLIER, t.scale*cs*mp*sp)` with `7.3`
(BIG) or `18.4` plus jackpot weight `w:0.0006`→`w:0.35` (JACKPOT + lift) — drive them with
headless Chrome over CDP, and **delete the copies**. Also learned here: `reticle()` used to
measure label text before setting its font, so every label box was ~150px wide regardless
of content (fixed); and the frozen big-win frame is drawn with `reticleLabels=false` so
the canvas's "SUBJECT · 0m TO EXIT" label can't collide with the verdict text.

## SDK integration — what exists and where

- `contract/ChipThiefGame.sol` (+ vendored `contract/ICasinoGameV2.sol`) —
  implements `ICasinoGameV2` as an instant game. Every client-side
  `Math.random()` roll (chip land/value, multiplier land/tier, staff
  land/tier, leap-catch) is reproduced on-chain via
  `keccak256(randomness, idx)` threshold draws — same lam/tables/RTP as
  `RUN` in chip-thief.html. Payout is hard-capped at **100× wager**: in a 40M-run
  sim ~1 in 52,000 runs (two jackpot chips + a multiplier) exceeds it, costing
  ~0.05% of EV (96.0% uncapped → ~95.9% effective — disclose it if asked). It lets
  the vault accept larger bets (reserve 99× wager) and sits at the facet's default
  heavy-tail threshold (>100×) instead of over it. `probabilityWad` (8e-4, the
  ≥30× tier) / `bodyVarianceScaled` (σ≈1.5×) for the SDK's risk model come from
  that same simulation, rounded conservative (safe direction for vault solvency).
  Re-verified after the jackpot pass against **100,000 live on-chain
  `onRandomness` calls**: 0 mismatches between the contract and the replica code
  extracted straight out of chip-thief.html (bit-for-bit), 95.82% RTP / 22.40% bust,
  median escape 0.84×. Earlier tail sims / call counts were for older paytables.
- **Guest bridge**: `chip-thief.html`'s `<script>` is now `type="module"` and
  talks to the host via a hand-vendored `connectGameToHost` (mirrors
  `@chain/casino-sdk/guest` exactly). Zero new *build-time* deps — `penpal`
  and `@noble/hashes`' keccak are vendored as static files in
  `prototypes/vendor/` (same-origin, no runtime CDN call — same reasoning as
  the "self-host the font" item above, just not yet applied to the font
  itself). The page still runs standalone with zero changes to that path:
  outside an iframe `connection.promise` never resolves (documented SDK
  behavior) and the original local-`Math.random()` demo loop is what runs.
  Inside the iframe, `chainOutcomeFromRandomness()` in the page replays the
  contract's exact draw sequence from the session's on-chain VRF word — the
  animation is never a locally-rolled guess, it's the verified-identical
  reconstruction of what the contract already paid. Confirmed with **zero**
  `local replay disagreed with the chain payout` warnings across every live
  test run.
- `prototypes/game.manifest.json` — instant-game shape (same as coinflip's:
  `full-iframe`, `openSession` only), validated against the SDK's own
  `validateCasinoGameManifest`.
- Local dev/test stack lives in `casino-sdk/` at the repo root — the
  unzipped SDK package (simulator, VRF node, docs, coinflip example),
  gitignored, **not part of the submission**. Re-download from
  `sdk.chain.wtf/sdk/casino-sdk.zip` if missing. `npm install && npm start`
  from there brings up chain+VRF+simulator(:3300)+coinflip(:3100); the
  local-node watcher auto-compiles/deploys anything dropped into
  `casino-sdk/simulator/contracts/` — that's where the canonical
  `contract/ChipThiefGame.sol` gets copied for testing (copy again after any
  edit, the two aren't symlinked).
- To test chip-thief.html against the simulator: serve `prototypes/` with a
  **CORS-enabled** static server (plain `python -m http.server` fails — no
  `Access-Control-Allow-Origin` header, and the simulator fetches
  `game.manifest.json` cross-origin), then open
  `http://localhost:3300/?game=http://localhost:<port>/chip-thief.html&gameAddress=<ChipThiefGame address from casino-sdk/simulator/local-node/deployed.json>`.
  Forgetting `gameAddress` silently reuses whatever contract the simulator
  had loaded before (bit me once — it tried to open a Chip Thief session
  against CoinflipGame and reverted).

## Known gaps
- None outstanding. (The old plaque/jackpot label mismatch — "500" worth 12, "25" worth
  25 — is fixed: chip labels are their face value and pay face×0.2×scale.)

## Rejected — don't re-propose
- **Cash-out / climbing multiplier** — makes the entry ineligible.
- **Slider-driven bets** — the decision is identical every round; set once and spam.
  Replaced with dealt choices (Insider/Underwriter) or no choice at all (Chip Thief).
- **Fake agency** — choices that don't affect anything. Players smell it and it fails
  "fun after 10 hours". Real agency inside the 93–98% band is the better trick.
- **Fracture / Net concepts** — required elaborate mouse work *every round*, which is
  fatiguing and suppresses bet count.

## Working style
The user wants brutal honesty over agreeableness, especially on design. They have
pushed back on: letting the deadline dominate answers, over-hedging, and shipping
things that look finished but aren't fun. Flag real problems directly, then build.
