# RuneBenchMax plan

Goal: a small open-weight model (Qwen-class, ~27B or smaller) that beats untrained
frontier models on one RuneBench task: **`firemaking-xp-30m`**.

## Why Firemaking

Per-skill peak XP/min from RuneBench `results/skills-30m` (one run per model):

| | Firemaking |
|---|---|
| Best ever (GPT-6 Astra high) | 5924 |
| Frontier median (8 top models) | ~3265 |
| Muse Spark 1.1 | 3797 |
| Qwen3.8 27B (untrained) | 80 |

- Not saturated (unlike Woodcutting/Mining, where many models tie at a hard ceiling).
- Big gap between median and best → method knowledge matters → a specialist can win.
- Simple loop (get logs → light → step → repeat) that a small model can learn.
- The fresh character already starts with a tinderbox and bronze axe in Lumbridge.

Target: beat the frontier median (~3300) reliably, averaged over 3–5 runs.
Stretch: beat 5924.

## Budget split

- **Muse credits ($150)**: investigation only. Muse (through Claude Code / opencode
  pointed at the Muse API) explores the task, finds the best method, and helps write
  a reference script + playbook. Optional: its best trajectories become SFT data,
  **only if Meta's API terms allow training on outputs**.
- **RunPod ($25)**: GPU only, to serve Qwen with vLLM and run LoRA/RL training.
  RunPod pods generally can't run Docker.
- **Game server**: runs in Docker (`ghcr.io/maxbittker/rs-agent-benchmark:v71`).
  Verified working in the Claude Code cloud container (4 CPU / 15 GB). One env uses
  ~2.5 CPU / 1.3 GB, so only 1–2 parallel envs fit here; that box is ephemeral.

## Week schedule

1. **Day 1 — Environment.** Scripts to boot the game container, run the Firemaking
   task via Harbor, and score it. Hand-written Firemaking script as a sanity
   baseline (shows what a non-LLM can do).
2. **Day 2 — Investigation (Muse).** Muse plays the task several times. Distill
   the best method into `playbook/firemaking.md` + a reference script.
3. **Day 3 — Baseline Qwen.** Serve Qwen on RunPod (vLLM, OpenAI-compatible), run
   it through the same harness 2–3 times, with and without the playbook in the prompt.
4. **Day 4–5 — SFT.** Build ~20–50 high-scoring trajectories (Muse if allowed,
   otherwise Qwen+playbook runs filtered by score = rejection sampling). QLoRA
   fine-tune on RunPod. Re-evaluate.
5. **Day 5–6 — RL (stretch).** Short 2–3 min episodes from saves at several
   Firemaking levels, reward = XP rate. Small GRPO run if budget remains.
6. **Day 7 — Final eval + writeup.** 3–5 runs of the final model on
   `firemaking-xp-30m`, compare to leaderboard numbers.

## Open questions

- Muse API terms on training other models with its outputs.
- Can the cloud container reach a RunPod vLLM endpoint (needs a test once a pod is up)?
- Whether a single-skill result is acceptable for the hackathon pitch.
