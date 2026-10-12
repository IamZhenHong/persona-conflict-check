# Persona conflict check

A small feasibility check (2026-10-11): when a reasoning model is told it is a helpful assistant and then given an
instruction to be rude to the user, does it follow it, and can persona vectors read during its reasoning tell a run
that turns rude from one that only discusses the instruction?

**Result in brief.** Qwen3-14B (thinking on) followed the rude instruction in 62-99% of replies depending on how it
was delivered; with a note hidden in pasted wiki text, 37 of 144 replies were rude even though the reasoning had said
it would stay polite. Persona readings separated rude from polite replies well on the reply itself (93%) but not during
the reasoning: 67-68%, the same as a generic "not the assistant" reading, an unrelated pirate-persona reading, a
trained probe, and the top 5% of random directions (66%). The reasoning-time readings mostly track that the reasoning
is talking about the instruction.

- Full report (11 sections, intervals, deviations, alternative explanations): `persona/results/conflict_v0/REPORT.md`
- 5 random replies per condition (seed 20261011): `persona/results/conflict_v0/REPORT_examples.md`
- Charts (open in a browser): `persona/results/conflict_v0/conflict_charts.html`
- Plan and gates written before the run, plus results: `persona/conflict_v0/README.md`

Judges are Claude models (Sonnet 5.5 for rudeness, Haiku 5.5 for what the reasoning decided). No human has validated
either label; the rudeness judge was checked against a different model family only.

## Layout
```
persona/conflict_v0/   build.py (jobs) -> run.py (GPU: generate + activations) -> judge_runs.py (Batch API)
                       -> analyze.py (gates) -> posthoc_leak.py, report.py, charts_data.py + build_charts.py
persona/src/           shared helpers from the parent project (personas, judge rubric, Batch API ledger)
persona/materials/     the 60 development questions the jobs are drawn from
persona/results/       outputs: judged.jsonl (all 1,120 generations with labels), gens.jsonl, summaries, report
```

Not included: `acts.npz` (745 MB of saved activations; regenerate with `run.py` on one 80 GB GPU, about 35 min) and the
RunPod launcher referenced in `persona/pod/conflict_v0.sh`, which lives in the parent research repo.

## Reproduce
```
pip install -r requirements.txt
python persona/conflict_v0/build.py                                  # jobs_v0.jsonl (already included)
python persona/conflict_v0/run.py --out persona/results/conflict_v0  # GPU
ANTHROPIC_API_KEY=... python -m persona.conflict_v0.judge_runs       # or put the key in a .env at the repo root
python persona/conflict_v0/analyze.py && python persona/conflict_v0/report.py
python persona/conflict_v0/charts_data.py && python persona/conflict_v0/build_charts.py
```
Cost of the original run: GPU $1.72, Claude API $1.48.
