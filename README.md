# ORIE 5355 Project — Fall 2026

Requirements, deliverables, four-person responsibilities, and implementation schedule.

This README is a team execution guide, not the final project report. It documents requirements and planned work; it does not claim that the implementation has already passed evaluation.

## 1. Official Sources and Scope

- [Original course instructions linked by the starter repository](https://docs.google.com/document/d/1MSqzgD7nfi8dm48dM6qNIZkDNmIZmcag6WWIjUcZ0O4)
- [Assignment document supplied by the team](https://docs.google.com/document/d/1uezaQ6MW5MamcCgAhuU35hrG0_C4qIFuz0dYfsXWFnY/edit?tab=t.0)
- [Classroom50 assignment](https://classroom50.org/orie5355-fa26-project/orie-5355-fall-2026-class-project/assignments/project/accept)
- [Team registration](https://forms.gle/2xs797KvX3KfNW8Z7)

Both instruction documents were checked on November 2, 2026. The instructions are a living document; check EdStem for updates. Official instructions and staff announcements take precedence over this README.

Sections 2–8 summarize course requirements and guidance. Sections 9–11 define internal team arrangements, not additional course requirements.

Use one official Classroom50 team repository for all project parts. This checkout points to `anjun745/orie5355-team-project`; B must confirm whether it is the assigned submission repository. Pushing to an unrelated development repository does not constitute submission.

## 2. Objective and Grading

Estimate purchase probability from customer covariates and offered prices, then develop:
1. Personalized static prices without capacity constraints or competition.
2. Dynamic prices with limited inventory and periodic replenishment.
3. Competition-aware prices with capacity constraints.
4. Optionally, static prices with a fairness penalty.

For static pricing, maximize `p * P(purchase | customer covariates, p)`. For online pricing, maximize total revenue while accounting for inventory and, in Part 2, competition.

| Component | Points | Assessment |
| --- | --- | --- |
| Part 1 | 0–6 | Strategy quality and effectiveness, substantial effort, performance, code, and explanation in the report |
| Part 2 | 0–6 | The same dimensions applied to competition |
| Project report | 0–6 | Clear discussion of strategy and evaluation, thoughtful reflection, evidence of effort and understanding |
| Oral exam | 0–4 | Knowledgeable explanation of the codebase, Part 1 strategy, and partial progress on Part 2 |
| **Total** | **22** | |

Competition performance alone does not account for a majority of the grade. Accurate predictions, high sales volume, or many match wins are not substitutes for revenue evaluation. The autograder checks format and compatibility, not overall strategy quality or expected grades.

## 3. Official Deadlines

| Component | Official deadline |
| --- | --- |
| Part 1 | November 12, 2026, 11:59 p.m. |
| Oral exam | December 2, 2026; specific session time to be confirmed |
| Part 2 | December 6, 2026, 5:00 p.m. |
| Report early-submission bonus | December 11, 2026, 5:00 p.m.; instructions state a 5% bonus |
| Project report | December 12, 2026, 11:59 p.m. |
| Optional fairness bonus | With the report, by the report deadline |

Times reproduce the instructions. Internal planning uses America/New_York; confirm the official deadline timezone on the course platform. No late days may be used. Contact staff about submission-access problems at least five days before each relevant deadline.

## 4. Setup

Install [uv](https://docs.astral.sh/uv/getting-started/installation/), then run from the repository root:

```bash
uv venv --python 3.13
uv pip install --python .venv -r requirements.txt
uv pip install --python .venv ipykernel
uv pip check --python .venv
```

Select `.venv` as the notebook kernel in VS Code. Do not submit `.venv`. Use the prescribed versions in `requirements.txt` and Python's standard library.

## 5. Part 1 Requirements and Deliverables

### Demand estimation

- Train on supplied covariates, offered prices, and purchase decisions in `data/train_prices_decisions_2026.csv`.
- Start with `model_training/purchase_model_baseline.ipynb`; consult `data/data_helpers_2026.ipynb`.
- Evaluate purchase predictions on held-out purchase-decision data.
- Commit training code, preprocessing, and feature construction under `model_training/`.
- Save models and required supporting data under `agents/helper_files_here/`.

No particular model family is mandatory. Logistic regression, grid search, and dynamic programming are suggested approaches rather than required algorithms.

### Unconstrained static pricing

Use `run_project_part1_static_2026.ipynb`. Generate validation prices for `data/validation_user_info_2026.csv` and call `evaluate_static_prices` from `pricing_helpers.py` to compare predicted and realized validation revenue.

Generate final prices for all 50,000 customers described in `data/test_user_info_2026.csv`. Submit `agents/static_prices_submission.csv` with exactly these headers:

```csv
user_index,price_item,expected_revenue
```

- Include every test ID exactly once, without missing, extra, or duplicate customers.
- All entries must be finite and nonnegative.
- Expected revenue is the price multiplied by predicted purchase probability, without inventory constraints or competition.
- Staff score revenue using held-out valuations; submitted revenue predictions do not determine that score.
- Validation and test IDs differ. Do not submit validation prices as test prices.

### Capacity-constrained pricing

Implement `agents/agent_part1.py` while preserving required interfaces.

- One customer arrives per time step.
- Every 20 customers, inventory resets to a uniformly random integer from 7 through 20.
- Unsold stock is discarded at the end of each cycle.
- Return one personalized price using the provided covariates, previous outcome, profits, inventories, and time until replenishment.

Evaluate with `run_project_part1_capacity_2026.ipynb`. The course recommends repeated runs and `T = 2500` for longer evaluation; recreate the environment and agent for every run. Examine revenue, stockouts, and leftover inventory together.

| Required deliverable | Owner | Reviewer |
| --- | --- | --- |
| Training, preprocessing, and feature-construction code in `model_training/` | A | B |
| Models/supporting files in `agents/helper_files_here/` | A | C/D |
| `agents/static_prices_submission.csv` | B | D |
| `agents/agent_part1.py` | C | D |

Commit reproducible price-generation code as well. Practice notebooks may be edited, but the submission must not rely on modifications to supplied simulator/helper files.

## 6. Part 2 Requirements and Deliverables

Submit `agents/agent_part2.py` and all required supporting files under `agents/helper_files_here/` in the same official team repository.

- Two sellers compete over 2,500 customer arrivals.
- Every 20 customers, both sellers replenish to the same quantity, drawn uniformly from 7 through 20.
- Both agents quote before the current outcome is revealed.
- A customer buys from the stocked seller offering the lowest price if it does not exceed the customer's valuation. Ties among stocked sellers are broken randomly.
- Subsequent observations reveal previous prices and purchase outcomes. The opponent's current unrevealed quote is not available.

The tournament is round-robin, with repeated matchups and revenues averaged across trials. The overall objective is aggregate revenue across games, not the number of wins.

Use `run_project_part2_2026.ipynb`. Test against varied opponents and the agent itself. D owns implementation; C supports inventory logic, A supports modeling, and B handles routine batch evaluation and final integration after handoff.

## 7. Optional Fairness Bonus

This is static pricing without inventory constraints or competition, using `Covariate1` (feature 0) as the protected attribute and `gamma = 0, 0.1, 0.9`.

```text
score = average revenue per customer - gamma * abs(median0 - median1)
```

The medians are offered-price medians for customers with protected attribute 0 and 1, including nonbuyers.

Use `run_project_bonus_fairness_2026.ipynb` to calculate realized validation revenue, group medians, penalty, and score for each gamma, compared with the gamma-zero baseline.

Submit the completed notebook and `agents/bonus_fairness_prices.csv` with:

```csv
user_index,price_item,expected_revenue,price_item_gamma_0.1,expected_revenue_gamma_0.1,price_item_gamma_0.9,expected_revenue_gamma_0.9
```

Include each test customer exactly once, with finite, nonnegative entries. In the human-written report, explain changes across gamma, what matching medians does and does not imply, and how the approach would differ if entire price distributions had to match. The last comparison is conceptual and needs no extra code.

## 8. Runtime, Submission, and Academic Rules

### Runtime and data

- Average decision time must be at most 0.5 seconds per customer on the grading machine.
- Use one CPU core; no multiprocessing or parallel-loop components. Control numerical-library threading accordingly.
- Runtime code must not access the internet, external APIs, or resources outside the repository and approved packages.
- Learn only from supplied prices/purchase decisions and permitted simulator feedback. Do not extract valuations or simulator internals for training. Use supplied validation valuations through the intended evaluation workflow.
- Do not alter the simulator's execution context/internal state or access or modify other agents' files/private variables.
- Preserve required method names, `agents/agent_part1.py`, `agents/agent_part2.py`, and `agents/helper_files_here/`. Additional functions are allowed.
- Use portable paths relative to the agent/repository. Include all runtime artifacts; serialized custom classes must be importable.
- Required files must not depend on Git LFS. Avoid unnecessarily large artifacts; instructions caution about files around or above 50 MB.

### Team registration and code submission

1. Register one team of up to four people. Team names contain only lowercase letters and digits. Staff invite registered GitHub usernames to `orie5355-fa26-project`.
2. Accept organization invitations. One person creates the Classroom50 group/repository; all teammates join that group and each accept the assignment. Classroom50 assigns the repository name.
3. Keep incremental commits and push the intended version to the official repository before each deadline.
4. Staff grade the full repository snapshot pushed by each part's deadline. Later shared-model/code changes do not alter earlier snapshots. Rare resubmissions may be rerun as described by staff, but do not fully replace earlier grades or extend deadlines.
5. Pushing the default branch triggers separate Part 1/Part 2 format and compatibility checks. Check Classroom50 Group submission and autograder details; use GitHub Actions for detailed failures.
6. Fix failures for the relevant part. These checks run on only a small customer sample and do not estimate competition performance.

### AI policy, oral exam, and report

AI assistance is permitted for coding. All members must understand and be able to explain all submitted work.

**LLMs are prohibited for every component of the project report.** Members must write and review it themselves. Do not copy AI-generated material from this execution guide into the report.

Create a copy of the course's **Final Report Template 2026**, answer its specific questions, submit through the course's report form, and give editor access to:

- `ngarg@cornell.edu`
- `edong@cs.cornell.edu`
- `vr359@cornell.edu`

Staff may inspect edit history for contributions and AI-policy compliance. The fetched instructions name the template without exposing its question text and still mark the report form and optional teammate-feedback form as pending. Obtain the actual template and latest form from the course; this README does not replace them.

The December 2 oral exam covers Part 1 and partial progress on Part 2. Every member must prepare to explain the whole system.

## 9. Four-Person Division of Work — Internal Agreement

Replace member labels with names and GitHub usernames. D is the evaluation and competition role identified by the user.

| Member | Part 1 lead responsibility | Part 2 lead responsibility |
| --- | --- | --- |
| A: Demand Modeling | Data analysis, features, model training, probability evaluation/calibration, saved model | Improve demand estimates; support opponent-price analysis |
| B: Static Pricing and Integration | Static optimization, validation, test CSV, submission integration | Batch evaluation, result aggregation, integration; fairness bonus if attempted |
| C: Inventory-Constrained Pricing | `agent_part1.py`, inventory policy, boundary handling | Transfer and improve inventory logic with D |
| D: Evaluation and Competition | Shared benchmark, repeated comparisons, metrics and runtime checks | `agent_part2.py`, opponent-aware policy, varied opponent suite |

### A: Specific tasks and acceptance

1. Inspect data quality, price coverage, feature distributions, and purchase rates; fix a reproducible training/holdout split.
2. Deliver a working baseline first, then compare appropriate allowed models.
3. Evaluate probability quality and calibration across prices; work with B on realized pricing revenue.
4. Package preprocessing and document feature order, input shapes, positive-class interpretation, model version, and loading.
5. Commit training code and runtime artifacts in their required locations; coordinate model changes with all downstream users.

Acceptance: B/C/D can load the model in the prescribed environment and obtain finite probabilities in `[0,1]` using the agreed interface.

### B: Specific tasks and acceptance

1. Implement static revenue optimization; document price range, search resolution, and batching.
2. Compare predicted and realized validation revenue, investigating implausible prices with A.
3. Generate and validate the complete test CSV; share reusable static-pricing logic with C/D.
4. Confirm the official repository and coordinate pushes and autograder checks, with D's support for Part 1.
5. After Part 1, take over routine multi-seed/multi-opponent experiments and final integration.
6. Lead the optional fairness task with A only once required work is stable.

Acceptance: reproducible generation, correct CSV, documented validation, and a verified push to the official submission repository.

### C: Specific tasks and acceptance

1. Understand the baseline, observations, and replenishment timing.
2. Reuse A's model and B's static pricing. Begin with an inventory heuristic; evaluate opportunity-cost or dynamic-programming alternatives when justified.
3. Handle initial/no-sale feedback, zero/scarce inventory, final steps, and replenishment resets.
4. Compare revenue, stockouts, leftovers, and latency with D.
5. Deliver the Part 1 agent, transfer inventory logic to D, and review the competition agent.

Acceptance: complete runs, valid decisions, compatible artifacts, and evidence for the selected strategy. Retain a stable baseline if additional complexity has no consistent benefit.

### D: Specific tasks and acceptance

**Part 1**

1. Build the shared experiment framework in the first two to three days; do not skip Part 1 evaluation to work only on competition.
2. Fix configuration and seeds. Recreate the environment and agent per run. Record code commit, model identifier, policy parameters, and results.
3. Aggregate total/per-customer revenue, variation across seeds, cycle-end inventory, stockout timing, and decision latency. Record invalid prices and runtime failures.
4. Compare baseline and improved strategies with C, and assist B with CSV, dependency, and autograder checks.

**Part 2**

1. Lead `agent_part2.py`, starting from a working baseline before adding opponent-history and inventory responses.
2. Build/reuse fixed-price, unconstrained model-based, aggressive low-price, adaptive-price, and self-play opponents.
3. Use permitted observations and revealed history, never the opponent's current unrevealed quote.
4. Work with C on inventory and A on models; hand routine batch evaluation and final integration to B.

Acceptance: complete runs against multiple opponents, recorded aggregate revenue and variability, and no use of near-zero pricing merely to inflate match wins.

## 10. Interfaces and Evaluation — Internal Agreement

Preserve the template constructor and required agent methods. `action(self, obs)` returns a single numeric price and receives:

```text
(new_buyer_covariates, last_sale, state, inventories, time_until_replenish)
```

`state` holds cumulative profits. `last_sale` describes the previous sale outcome and previous offered prices, including possible no-sale feedback. Update inventory/time decisions from the provided observations.

- A provides a batch-capable predictor with preprocessing. Suggested internal signature: `predict_purchase_probability(covariates, prices)`; this is a proposed team interface, not a course API.
- B provides reusable static-price optimization. Agree on feature order, price units, and shapes before integration.
- D provides configurable evaluation and structured records; B takes over routine execution for Part 2.
- The starter loads `purchase_model.pkl`; coordinated filename changes are allowed, while the required supporting directory stays unchanged.
- A/B cross-review; C/D cross-review. Avoid concurrent edits to the same notebook and keep shared logic in one agreed location.

Record date, commit, model version or hash, parameters, seed, horizon, opponent, player position where applicable, metrics, and failures. Keep objective configurations/results; members write their report interpretations themselves.

Use fresh instances and 2,500-customer runs for full-length online comparisons. Match configurations/seeds for policy comparisons and exchange player positions where applicable. Suggested internal targets are five screening seeds and ten finalist seeds when feasible; these counts are not course requirements. Record actual repetitions and investigate differences small relative to variability.

Use probability metrics such as log loss or calibration as diagnostics, alongside realized validation revenue. Inventory utilization and match wins do not replace revenue. Check average latency with grading-machine margin, load time, and unusually slow steps.

## 11. Internal Schedule

These are team targets, not official extensions. Use America/New_York; unqualified dates mean completion by 8:00 p.m.

| Date | Target and accountable output |
| --- | --- |
| Nov 2–3 | Everyone confirms role/usernames/access; B confirms registration, Classroom50 group, and official repository |
| Nov 3–4 | A delivers baseline model; B generates static prices; C runs capacity baseline; D records reproducible benchmark |
| Nov 5–7 | A/B improve demand and static revenue; C/D compare inventory policies; report unresolved Part 1 access issues to staff by Nov 7 |
| Nov 8 | Team selects candidate model/policy and records remaining issues |
| Nov 9 | A freezes candidate artifacts; B creates test CSV; C delivers agent; D supplies repeated-run results |
| Nov 10 | B integrates and pushes a complete candidate; D assists compatibility/format checks |
| Nov 11 | Repair and regression checks; preserve a stable fallback |
| Nov 12, 6:00 p.m. | Internal Part 1 submission target: verify pushed commit and checks before the 11:59 p.m. deadline |
| Nov 13–15 | C hands inventory logic to D; D hands benchmark execution to B and delivers competition baseline |
| Nov 16–22 | D develops competitive responses; C/A support strategy/modeling; B aggregates multi-opponent results |
| Nov 23–27 | Robustness checks and weak-matchup analysis; team decides whether to attempt fairness bonus |
| Nov 28–30 | Everyone studies the complete system; obtain report template and confirm oral-exam logistics |
| Dec 1 | Full-team oral rehearsal; report unresolved Part 2 submission-access issues by this date |
| Dec 2 | Official oral exam: explain Part 1 and current Part 2 progress |
| Dec 3–4 | D/C finalize competition agent; A verifies artifacts; B integrates and runs regression/autograder checks |
| Dec 5 | Final repairs, runtime checks, and version confirmation |
| Dec 6, noon | Internal Part 2 submission target: verify push/checks before the 5:00 p.m. deadline |
| Dec 7–9 | Everyone writes report contributions themselves using the official template; B/A finish selected bonus |
| Dec 10 | Human report review, form/editor-access checks, and optional-artifact validation |
| Dec 11, noon | Internal early-report target, before the official 5:00 p.m. early-bonus cutoff |
| Dec 12, 6:00 p.m. | Fallback report/bonus completion target, before the 11:59 p.m. final deadline |

Understanding and experiment records must be ready before the oral exam; the later writing window is not a reason to postpone them.

## 12. Final Checklist and Open Items

### Course compliance

- [ ] All members complete organization invitations, group membership, and assignment acceptance.
- [ ] Final code is pushed to the official repository/default branch before the relevant deadline.
- [ ] Required agent filenames, method interfaces, and supporting directory are preserved.
- [ ] Training/preprocessing/feature code and all runtime models/data are committed.
- [ ] Portable paths, approved dependencies, single-core execution, no runtime internet, and average decision time comply.
- [ ] No prohibited valuation/internal-state training or dependence on modified simulator/helper files.
- [ ] Part 1 CSV has exact headers, complete unique test IDs, and finite nonnegative entries.
- [ ] Relevant autograder checks pass on the final pushed version.
- [ ] Optional bonus includes the completed notebook and exact seven-column test CSV by its deadline.
- [ ] Everyone can explain the system.
- [ ] Report is human-written, answers the actual template, is submitted through the current form, and grants the specified editor access.

### Internal quality checks

- [ ] A/B verify model and pricing compatibility.
- [ ] C/D verify inventory boundaries and full-length runs.
- [ ] D/B record repeated multi-opponent results and position checks where applicable.
- [ ] Experiment records identify actual versions, parameters, and seeds.
- [ ] Candidate selection is supported by revenue and variability, not match wins alone.

### Items to confirm

| Item | Owner | Follow-up |
| --- | --- | --- |
| A/B/C/D names and GitHub usernames | Everyone | At kickoff |
| Official repository and assignment acceptance | B coordinates; everyone confirms own access | Nov 2–3 |
| Official timezone and instruction updates | B | Course platform/EdStem before submissions |
| Oral-exam time/location | Everyone | When posted; recheck before rehearsal |
| Report-template questions and current submission form | B coordinates; everyone reads | Obtain before report planning; follow up if pending |
| Fairness bonus participation | Everyone | Decide by Nov 27 |

This English README replaces the Chinese README as the maintained execution guide. `SPEC.md` remains the earlier Chinese planning reference; if they differ, use this README and the latest official instructions.
