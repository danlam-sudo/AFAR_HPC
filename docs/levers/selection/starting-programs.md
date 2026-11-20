# Starting programs

| | |
|---|---|
| Stage | selection |
| Status | idea (2026-11-19). One seed per campaign today: LRU (`seeds/evolved_repl.h`) or, since 2026-11-19, Mockingjay (`seeds/mockingjay.h`) |
| Switch | `seed:` in the run spec, one file. Several seeds would need a list there |
| Class | changes results |

## Question

Does a campaign find better or more varied policies when it starts from several published policies at once (say Mockingjay, SHiP and Hawkeye) instead of one? Measured against the single-seed baseline, `crc2-evolve-mj1`, at equal generations and CPU-hours: the best so far against CPU-hours, validated speedup, and the behaviors found (`make campaign-report`, `make campaign-diversity`).

## How it works

**Today, one seed.**
- Generation 0 is the seed alone, evaluated like any candidate, and placed on island 0 (`controller/campaign.py`, `start`; `oe.add_program(..., target_island=0)`).
- OpenEvolve fills an empty island with a copy of the best program when it first samples it (`openevolve/database.py`, around line 1469).
- So every island starts from the seed.

**Several seeds, one per island,** would be a small controller change:
- `seed:` takes a list.
- Generation 0 evaluates every seed.
- Seed *i* goes to island *i* (`database.add(program, target_island=i)` already exists); the spec needs at least as many islands as seeds.

**What it needs:**
- Every seed must keep the seed frame. `evaluator/static_checks.py` compares the code outside the EVOLVE-BLOCK with `seeds/evolved_repl.h`'s; `seeds/mockingjay.h` already does.

**Separate campaigns, one per seed,** need no code: one spec per seed file.

## Evidence

- 2026-11-19, design only. The baseline uses one seed, Mockingjay, as ArchAgent v1 did (arXiv 2602.22425, §3.2: it tried LRU first and moved to Mockingjay for faster progress). ArchAgent reports no multi-seed runs.

## Costs and risks

- **Budget split.** With one seed per island, each lineage gets a third of a generation's candidates (14 of 42) instead of all of them.
- **Islands blend.** Migration copies each island's best to its neighbours every 50 new programs per island (about every 4 generations at 14 per island). A much stronger seed (Mockingjay scores 1.1174 against SHiP's 1.0523 on `crc2-evolve-v1`) soon fills the other islands, and the run drifts toward a single-seed campaign with less budget.
- **Harder to read.** Which seed a winner descends from is recorded (parent chains), but the effect of each seed is confounded with the others'.
- **Cost of seeds.** Each seed's evaluation in generation 0 costs about 5 CPU-hours.

## Decision and next step

- **2026-11-19:** the baseline campaign uses one seed. Several seeds per campaign is a later experiment against that baseline, beside separate per-seed campaigns.
- First, port and check the other seeds the way Mockingjay was checked (`ports/mockingjay/`: split hooks, bypass = `NUM_WAY`, geometry).

## Sources

- ArchAgent v1, arXiv 2602.22425, §3.2 and §8.1.1.
- `docs/campaigns.md` §9: islands and behaviors in the pilot.
