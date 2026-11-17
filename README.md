# Dynamic Color Changes in Minimalistic Robot Swarm Segregation

A swarm-robotics study that extends the minimalistic segregation model of Mitrano et al., *"A Minimalistic Approach to Segregation in Robot Swarms"* (MRS 2019), by making the attribute that drives segregation — each robot's **color** — **change over time**. We ask: when the very property robots segregate on is dynamic, does segregation still emerge, how fast does it recover, and can structured patterns appear?

Course project for **RBE 511: Swarm Intelligence, WPI**. Implemented in **Buzz** on the **ARGoS** simulator. Team project.

---

## Overview

In the baseline model, each robot carries only a **ternary sensor** that reports whether the closest robot in its line of sight is (i) absent, (ii) the same color, or (iii) a different color. A purely reactive rule maps that reading to wheel speeds, and segregation emerges with no global communication or computation.

Real agents aren't static, though — their properties change with time. We extend the model by **changing robot colors during the simulation** using four strategies, and measure how segregation holds up using the paper's cost function. A swarm of **24 robots** across **8 color classes** runs in a 5×5 arena for 2000 ticks; each color switch is expected to spike the cost function, and "success" is the segregation controller **recovering** quickly afterward.

## The segregation controller (baseline)

Each robot broadcasts its color, finds the closest neighbor within its field of view (azimuth gate), and sets a ternary state `S`:

- `S = 0` — no neighbor in line of sight → turn to search
- `S = 1` — closest neighbor is the **same** color → move to stay with kin
- `S = 2` — closest neighbor is a **different** color → turn away

`S` maps directly to left/right wheel speeds. Segregation is quantified with a **cost function** `γ(t)`, computed on-robot via **virtual stigmergy**: each robot shares its position and color, the swarm clusters agents of each color by proximity, and `γ` is the (negated) ratio of the largest same-color cluster to the total agents of that color, averaged over all 8 classes. `γ` ranges from **0 (no segregation)** to **−1 (fully segregated)**.

## Color-change strategies

| # | Strategy | File | Idea |
|---|----------|------|------|
| 1 | **Random** | `FP2_Random.bzz` | Every *N* timesteps, each robot picks a new color uniformly at random. |
| 2 | **Fixed cycles** | `FP2_Fixed_Cycles.bzz` | Each robot gets a randomly-ordered fixed color list and advances to the next color every *N* timesteps. |
| 3 | **Schelling (stochastic majority)** | `FP2_Schelling_Model.bzz` | Each robot checks the fraction of neighbors of each color; if any fraction exceeds a threshold, it adopts that color. |
| 4 | **Grayscale (continuous)** | `FP2_Grayscale.bzz` | Each robot walks through 8 shades of gray following an independent sine-based schedule (black → white → black). |

Common parameters: `switchFreq` (how often colors change) and `aggTime` (initial settling period before switching starts). Wheel speeds are held at the segregation-optimal values from the baseline paper so that **frequency** is the variable under study. The Schelling strategy adds a `sm_threshold`; grayscale adds a sine period.

## Key results

- **Random & fixed-cycle switching:** each color switch sharply raises the cost function, then the controller works it back down. At **low frequencies** (switch every 250 or 500 steps) the swarm re-segregates well; at **high frequency** (every 100 steps) it can't fully recover. Recovery also degrades later in a run — we attribute this to **occlusion** building up, which starves the line-of-sight sensor.
- **Schelling / majority:** robots converge to a single color for thresholds **up to 0.65**; above that (e.g. 0.7) the swarm can split into multiple color groups. Low thresholds converge fast, high thresholds slowly. The interesting case is **threshold = 0.5**, a critical point where the swarm still converges to one color but **not necessarily the initial majority** — the outcome depends on the spatial layout and visibility of the majority-colored robots. (See the `SchellingModelData1/` videos.)
- **Grayscale:** behavior tracked the first two strategies; no distinct pattern formation emerged.

## Repository structure

| Path | Description |
|------|-------------|
| `FP2.argos` | ARGoS experiment config — 24 foot-bots, 5×5 arena, 10 ticks/s, Buzz foot-bot controller. |
| `FP2.bzz` | Base / shared experiment script. |
| `FP2_Random.bzz` | Strategy 1 — random color switching. |
| `FP2_Fixed_Cycles.bzz` / `FP2_Fixed_Cycles_new.bzz` | Strategy 2 — fixed-cycle switching. |
| `FP2_Schelling_Model.bzz` | Strategy 3 — stochastic majority (Schelling-inspired). |
| `FP2_Grayscale.bzz` | Strategy 4 — continuous grayscale evolution. |
| `segregation.bzz` | Baseline segregation controller. |
| `SchellingModelData1/` | Logged `γ` data (`.txt`) and recorded runs (`.webm`) for thresholds 0.3 / 0.5 / 0.65 / 0.7. |

## Running it

Requires [ARGoS3](https://www.argos-sim.info/) with the [Buzz](https://github.com/buzz-lang/Buzz) extension installed.

```bash
# make sure the Buzz include path is set (for vec2.bzz etc.)
export BUZZ_INCLUDE_PATH=/usr/local/share/buzz/include

# launch a strategy by pointing the .argos config at the chosen .bzz script,
# then run:
argos3 -c FP2.argos
```

Open `FP2.argos` and set the controller's `bytecode_file` to the strategy you want (e.g. `FP2_Random.bzz`). Tune `switchFreq` / `aggTime` (and `sm_threshold` for Schelling) inside the script to reproduce the frequency/threshold sweeps.

## My contributions

This was a team project. I implemented the **random** and **fixed-cycle** color-switching strategies and worked on the **Schelling-model (stochastic majority)** strategy, and I contributed to the **analysis** — running the frequency/threshold sweeps and interpreting the cost-function behavior (recovery vs. switching frequency, and the convergence behavior around the 0.5 threshold).

## Acknowledgements

- Baseline model: M. Giancola, C. Pinciroli, P. Mitrano, J. Burklund, *"A Minimalistic Approach to Segregation in Robot Swarms,"* MRS 2019.
- Built with [ARGoS](https://www.argos-sim.info/) and [Buzz](https://github.com/buzz-lang/Buzz).

**Authors:** Adil Shiyas, Laasya Kodandaramu, Lorenzo Manfredi Segato
