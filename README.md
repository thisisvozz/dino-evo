# Dino·Evo — *This Game Is Too Dumb For AI*

A neuroevolution sandbox where 200 dinosaurs learn to jump a cactus by dying repeatedly — and then get beaten by an obstacle so stupid that a five-year-old solves it in one second.

**[▶ Watch the video](https://youtu.be/dc-oYbgWEKY)** · **[🎮 Try the simulation](https://dino-evo.vercel.app)**

> One HTML file. No dependencies, no build step, no framework. Open it and it runs.

---

## The premise

The game is the dumbest thing you can build: you run, a cactus comes, you jump. A genetic algorithm solves it in **15 generations**.

So the game gets *dumber*, not harder. A new obstacle appears — the **fake cactus**. It's cactus-shaped, cactus-sized, and **not solid**. Run straight through it and nothing happens. Jump, and you die.

The correct answer is to do nothing. Humans get this instantly.

A veteran dinosaur with 60 generations of "cactus-shaped thing ahead → jump" burned into its weights **cannot** get this. It took **159 generations** to unlearn a rule it learned in 15. A brand-new random population — which never learned the rule in the first place — solved it in **12**.

That's the whole video. The simulation is the evidence.

---

## Run it

**Locally** — it's a single static file, so just open it:

```bash
open dino-evo.html          # macOS
xdg-open dino-evo.html      # Linux
start dino-evo.html         # Windows
```

Or serve it if you prefer a real origin:

```bash
python3 -m http.server 8000
# → http://localhost:8000/dino-evo.html
```

Everything runs client-side on `<canvas>`. The only network request is Google Fonts; without it the page falls back to system fonts and works fine.

---

## Reproducing the numbers from the video

The RNG is seeded, so the same seed reproduces the same obstacle course *and* the same evolution, every run, regardless of what ran before it.

| Setting | Value |
|---|---|
| Seed | `dumb-game-3` |
| Mutation rate | **3%** |
| Population | 200 |
| Fake cactus | on · How often **35%** · Looks real **35%** |
| Goal | score 100 |

Measured results on that configuration:

| Milestone | Generation |
|---|---|
| Plain game solved | **15** |
| Veteran trained (before the fake cactus) | 60 |
| Veteran's best over the next 100 generations | 15 |
| Rookie solves the fake cactus | **12** |
| Veteran finally solves it | **159** |

Raw per-generation records, if you need them for editing:

```
PLAIN GAME  gen 1-15:  7,5,9,30,4,10,7,9,1,4,6,4,6,21,100
VETERAN     gen 1-20:  2,0,8,5,3,1,1,0,1,5,0,1,1,5,1,2,2,0,1,3   -> 100 at gen 159
ROOKIE      gen 1-20:  2,1,4,5,3,1,1,18,3,0,2,100,100,...        -> 100 at gen 12
```

### Two settings that will ruin the result

- **Mutation rate matters more than anything else.** At 8% the veteran solves the fake cactus *faster* than the rookie and the whole point collapses. At 3% the steps are small enough to trap the veteran in its own optimum. Across 6 seeds the rookie wins 4 at 3%, but only 2 at 8%.
- **Don't push "Looks real" to 100.** Past roughly 60% the fake cactus sits exactly on the line between the two real sizes, there's no distinguishing feature left, and *nobody* solves it — rookie included. That isn't a twist, it's a shared failure.

---

## How the AI works

No training data, no backpropagation, no reward function. Just copy-the-best-and-jiggle-it:

1. Run all 200 dinosaurs until every one is dead (or one hits the goal).
2. Find the single dinosaur that got furthest.
3. Make 200 copies of it, mutating each weight with probability = mutation rate (Gaussian nudge, σ 0.5).
4. Repeat.

That's top-1 elitism with mutation and no crossover. Nobody teaches them anything; they die in slightly different ways until one dies less.

**The brain** is a tiny feed-forward net — 4 inputs → 6 hidden (`tanh`) → 2 outputs (`softmax`), **44 weights total**:

| Layer | Neurons |
|---|---|
| Input | distance to obstacle · its height · its width · current game speed |
| Hidden | 6 — nobody, including the author, knows what these are doing |
| Output | **Jump** or **Nothing** — the entire decision it gets to make |

Note the input layer: the dinosaur sees **four numbers**. It never sees colour. The fake cactus is drawn with a magenta dashed outline and a `FAKE` label so *you* can tell them apart — that's a courtesy to the viewer, not information the network receives. Its size is deliberately placed between the two real cacti (`20×34` and `30×52`), so any rule of the form "cactus-sized thing ahead → jump" fires on it too. Escaping requires a rule with a hole in the middle: jump at 34, jump at 52, do nothing at 44.

A generation also ends after 8000 frames, so a population that has genuinely solved the game doesn't run forever.

---

## Controls

**Simulation** — speed (1× / 3× / 8× / turbo), mutation rate, game speed, seed, pause, restart, and **Play it yourself** (space or ↑ to jump, same physics, same fake cactus).

**The dumb obstacle** — fake cactus on/off, how often it spawns, how closely it resembles a real one.

**Lab · veteran vs rookie**
- `Save champion` — stores the current best brain plus its full history *in memory* (see the caveat below)
- `Download .json` / `Load .json` — persist a champion to disk
- `Run champion` — watch the saved brain alone
- `Continue from champion` — resume evolving from it
- `Veteran vs rookie` — split screen: saved champion's lineage on the left, a fresh random population on the right, identical course

**Replay & brain evolution**
- `Replay` — replay the champion of any generation, frame-accurate
- `Brain history` slider + `Play brain evolution` — scrub the weights across every generation and watch connections form and get torn down
- Click any node in the brain overlay (top-right of the game canvas) to thicken its connections and print their weight values — this is how you show the `cactus → jump` edge getting fat

> ⚠️ **`Save champion` is memory-only** — it does not survive a page reload. If you're recording a session and want the veteran to survive a refresh or a crash, use `Download .json`.

---

## Recording setup

The page has a built-in capture mode so the footage doesn't need cropping.

| Key | Action |
|---|---|
| `R` | Record mode — hides chrome, fills the frame, fades the cursor when idle |
| `G` | Show/hide the record-per-generation graph |
| `H` | Hide the HUD chips |
| `P` | Pause |
| `S` | Cycle simulation speed |
| `V` | Toggle veteran-vs-rookie split |
| `1`–`4` | Reveal stages (below) |
| `?` | Pin the shortcut list |

### Reveal stages

So the first frame doesn't spoil the twist, panels stay hidden until you call them in. This is keyboard-driven and invisible to the viewer.

| Key | Reveals | Use from |
|---|---|---|
| `1` | just the dumb game | sections 1–3 |
| `2` | + fake cactus panel | section 4 |
| `3` | + veteran vs rookie | section 6 |
| `4` | + replay & brain evolution | section 7 |

### URL flags

Handy for an OBS browser source, so the page opens already framed:

```
dino-evo.html#rec&stage=2&speed=3&graph&seed=dumb-game-3
```

`rec` · `graph` · `stage=1-4` · `speed=1|3|8|40` · `seed=NAME`

---

## Champion file format

`Download .json` produces a plain object — 44 weights, plus enough history for replay and the brain-evolution slider to work after loading:

```json
{
  "gen": 60,
  "score": 100,
  "seed": "dumb-game-3",
  "w": [0.41, -1.13, "… 44 floats total"],
  "hist": [7, 5, 9, 30, "…"],
  "log": [{ "gen": 1, "run": 0, "score": 7, "w": ["…"] }]
}
```

A champion is only itself on its own seed — the course is generated from the seed, so loading a file also restores the seed it was trained on.

---

## Deploying to Vercel

It's a static site, so there's nothing to configure. Rename the entry file so it lands on the root URL:

```bash
mv dino-evo.html index.html
```

Then either push the repo and import it at [vercel.com/new](https://vercel.com/new) (Framework Preset: **Other**, no build command, no output directory), or:

```bash
npm i -g vercel
vercel --prod
```

If you'd rather keep the filename, add a `vercel.json` instead:

```json
{ "rewrites": [{ "source": "/", "destination": "/dino-evo.html" }] }
```

---

## Repo contents

```
.
├── dino-evo.html            # the entire simulation — rename to index.html to deploy
├── dumb-game-ai-script-v2.md  # video script + production notes (Hungarian)
└── README.md
```

---

## Notes

The dinosaur, the cactus and the jump are borrowed from Chrome's offline game, but none of the code is: this is a reimplementation, built specifically so the game could be made *worse* on purpose.

## License

MIT — do whatever you like with it. If you make something interesting, I'd like to see it.