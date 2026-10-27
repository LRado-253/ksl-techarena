# Skills Design (Preliminary)

## 1. Our understanding of the task

The agent plays a new hire on a product R&D team for up to 48 months. Each month brings up to three story events, each resolved by choosing one option. At the end of each quarter the agent spends up to 3 Energy points on Energy actions and may choose one main action. Every six months there is a performance review (grades S to D, with a bonus tied to monthly base salary) and a promotion window across ten levels (L1 to L10).

The visible statistics are Health and Dignity (0 to 10), Skill, Network and Output (0 and above), Wealth (unbounded, can become negative), Level, Energy and CurrentLevelDuration. Exact formulas, failure thresholds, hidden risks and event probabilities are not published, and many risks accumulate and settle later rather than in the month they are created.

A good run therefore has two goals that pull against each other:

1. **Survive all 48 months** by keeping Health, Dignity and Wealth away from dangerous lows and by limiting hidden risk.
2. **Progress** by building Skill, Network and Output steadily enough to earn good review grades and promotions, which raise salary (from 0.3 per month at L1 to 9.1 at L10).

Our view is that this is a sequential decision problem under incomplete information. The agent should judge trends and warnings rather than chase single-month gains, and it should treat recovery as an investment rather than a waste.

## 2. Skills and responsibilities

We use a single agent (JiuwenSwarm 0.2.4b3, `"mode": "agent"`) with three Skills. Each Skill owns one kind of judgement so that the instructions stay short and token use stays low.

| Skill | Responsibility | When it is used |
|---|---|---|
| `career-state-assessment` | Reads the current state and recent logs, then produces a short status: which statistics are healthy, which are at risk, the direction of each trend, the current career phase, and the single most important priority right now. | Before every decision. |
| `event-decision` | Chooses an option for a monthly story event by weighing the immediate effect against later risk and against the priority from the assessment. | When the current step is a story event. |
| `quarterly-planning` | Allocates the 3 Energy points and selects the quarterly main action, taking into account the timing of the next six-month review and promotion window. | When the current step is the quarterly action phase. |

### How the outputs combine into a decision

1. `observe` returns the state, the current event or phase, and the available choices.
2. `career-state-assessment` turns that into a compact status with one top priority (for example "recover Health", "protect Dignity", "build Output before the review", or "balanced growth").
3. Depending on the step type, `event-decision` or `quarterly-planning` uses that status to pick a choice.
4. The agent calls `take_action` with the choice and a one-line reason in `notes`.
5. After the action, the agent reads the returned feedback and warnings. These feed into the next assessment, so the agent learns from the run as it goes.

## 3. Information the agent uses

- **Public rules and the handbook**, from `show_employee_handbook`: the level structure, salary schedule, review grade meanings, and the published direction of every Energy and main action (for example, high-intensity work gives `Output +`, `Health -` and `Dignity -`).
- **Current visible statistics and the time position**, from `observe`: the month, the quarter, how far away the next review is, and CurrentLevelDuration.
- **In-run feedback**: warnings, review grades received, and promotion outcomes, from `observe` and `check_latest_logs`.
- **The agent's own notes** in the log, to keep track of trends over several months.

The agent does **not** use hard-coded answers for specific events, unpublished thresholds, or any knowledge of the engine or dataset internals. Its judgements are relative (for example, "Health is falling for three quarters in a row" or "Health is low compared with its 0 to 10 range") and come from the published effect directions and the feedback received during the run.

## 4. Planned decision principles

- **Survival first.** When Health or Dignity is low or falling, or a warning appears, recovery takes priority over growth.
- **Delayed costs.** Options that give large immediate gains at a cost to Health, Dignity or hidden risk are treated with suspicion, because consequences may settle later.
- **Balanced growth for promotion.** Promotion depends on Skill, Network and Output together and on sustained results, so the agent avoids pushing one statistic while neglecting the others.
- **Review timing.** In the quarter before a review, weight should shift toward Output and visible results. Just after a review, it can shift toward Skill and Network.
- **Risk housekeeping.** "Defuse legacy risk" and "Lay low and reduce risk" are used when warnings or patterns suggest hidden risk is building up, even though they have no visible statistical gain.
- **Wealth floor.** Early salary is small, so the agent avoids choices that push Wealth negative unless the gain is clearly worth it.

## 5. Main risks in this approach

- **Unknown thresholds.** We cannot see how close a statistic is to a failure point, so we may be too cautious (slow promotion) or too aggressive (early failure). We will tune the balance using local runs on the dev dataset, without memorising specific events.
- **Delayed hidden risk.** Some choices may look safe but create risk that only appears much later. We rely on warnings and on the risk-defusing actions to manage this.
- **Overfitting to the dev dataset.** The official evaluation uses unseen data, so our Skills describe general principles rather than responses to specific events.
- **Token cost.** A long reasoning chain on every one of up to 144 events would be expensive. The Skills are kept short and the assessment is compact so that each decision uses few tokens.
- **Model variance.** Each version is scored over three runs. We aim for consistent, steady play rather than high-variance strategies.

## 6. Next steps after receiving the API key

1. Run a baseline game and read the replay to see where the run fails or stalls.
2. Adjust the balance between recovery and growth based on the warnings and review grades observed.
3. Add a reference file to a Skill only if it clearly improves decisions without adding much token cost.
4. Write `decision_report.md`, the README and the presentation for the final submission.
