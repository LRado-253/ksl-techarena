# TechArena 2026: Career Survival Agent (team working repo)

Our working repository for the Huawei TechArena UK 2026 **Career Survival Agent** track. We write Skills that guide a single JiuwenSwarm agent through a 48-month simulated career.

This repo is built on the official participation repository, [RAI-Internal/techarena-career-sim](https://github.com/RAI-Internal/techarena-career-sim) (MIT licence, see `LICENSE`). The official documentation is the authoritative reference: https://techarena-career-sim.readthedocs.io/en/latest/ (source copy in `docs/`).

> **Windows:** run every command in **PowerShell**.
> **macOS:** run every command in **Terminal**.

---

## What's in here

```
preliminary/          # package uploaded to request the API key
  manifest.json
  design/skills_design.md
  skills/
    career-state-assessment/SKILL.md
    event-decision/SKILL.md
    quarterly-planning/SKILL.md
solution/             # final submission folder (used by `make play`); not tracked if .gitignore excludes it
docs/                 # official docs (rules.rst, handbook.rst, api-key.rst, submission.rst, ...)
career_sim_runner/    # official runner - do not modify
example_*_structure/  # official templates
```

---

## Setup on a new laptop

### 1. Install the tools (one time)

**Windows**
```powershell
winget install --id Git.Git -e
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
winget install ezwinports.make
```

**macOS** (needs [Homebrew](https://brew.sh))
```bash
xcode-select --install          # gives you git and make; skip if already installed
brew install uv
```

Then **close and reopen the terminal**, and check that all three print a version:

**Windows and macOS**
```bash
git --version
uv --version
make --version
```

> If you use **Anaconda**, run `conda deactivate` in every new terminal before working here. Conda's Python can clash with uv.

### 2. Clone the repo

**Windows**
```powershell
cd "$HOME\Desktop"
git clone https://github.com/krsnkkv/techarena-career-simu.git
cd techarena-career-simu
```

**macOS**
```bash
cd ~/Desktop
git clone https://github.com/krsnkkv/techarena-career-simu.git
cd techarena-career-simu
```

### 3. Install dependencies

**Windows and macOS**
```bash
make sync
```
This installs everything into the repo's own `.venv`, including JiuwenSwarm `0.2.4b3`, the required version. Always run tools with `uv run ...` or via `make`. Do not install JiuwenSwarm globally or upgrade it.

### 4. Add the API key

The key comes **by email from the organisers** after the preliminary submission is approved. Ask a teammate for it, and never paste it into chat or commit it.

**Windows**
```powershell
Copy-Item .env.example .env
notepad .env
```

**macOS**
```bash
cp .env.example .env
open -e .env
```

Fill in exactly these three lines, using the values from the organiser email, then save:
```
API_BASE="..."
API_KEY="..."
MODEL_NAME="..."
```
Do not add any other routing or provider settings.

Check that `.env` will never be committed. This should print a line mentioning `.gitignore`:

**Windows and macOS**
```bash
git check-ignore -v .env
```

### 5. Initialise JiuwenSwarm

**Windows and macOS**
```bash
make setup
```
When it asks for a language, **type `2` (English)**. Everything in this competition must be in English.

If you accidentally chose Chinese, open the config and change `preferred_language: zh` to `preferred_language: en`:

**Windows**
```powershell
notepad "$HOME\.jiuwenswarm-instances\career_emu\config\config.yaml"
```

**macOS**
```bash
open -e ~/.jiuwenswarm-instances/career_emu/config/config.yaml
```

### 6. Put our Skills into `solution/`

**Windows**
```powershell
if (-not (Test-Path solution)) { Copy-Item -Recurse example_submission_structure solution }
Copy-Item -Recurse -Force preliminary\skills solution\
Copy-Item -Force preliminary\manifest.json solution\
```

**macOS**
```bash
[ -d solution ] || cp -R example_submission_structure solution
cp -R preliminary/skills solution/
cp preliminary/manifest.json solution/
```

---

## Running a game

Open **two terminals**, both in the repo folder.

**Terminal 1** starts JiuwenSwarm. Leave it running.

**Windows and macOS**
```bash
make start-jiuwen
```
The web UI is at `http://localhost:6173`. Check the port with `uv run jiuwenswarm-start --list`.

**Terminal 2** plays and inspects.

**Windows and macOS**
```bash
make play      # agent plays one full game using solution/skills
make score     # ending score of the last run
make replay    # readable Markdown recap of the last run
```

Run output goes to `.career_sim_runner/career_emu/outputs/<submission>/<timestamp>/`, which contains the transcript, `events-*.jsonl` and `score_report.json`.

To open the latest outputs folder:

**Windows**
```powershell
explorer .career_sim_runner\career_emu\outputs
```

**macOS**
```bash
open .career_sim_runner/career_emu/outputs
```

> **Each `make play` spends shared credit.** The key has USD 50 in total for the whole team and only works during competition week (26 Oct to 1 Nov 2026, UK time). Agree with the team before running, and note down what each run is testing.

---

## Validating and packaging

**Final submission check**

**Windows and macOS**
```bash
uv run python -m career_sim_runner validate --submission solution
```
This fails until `solution/presentation.mp4` is a real video. That's expected.

**Zip the preliminary package**

**Windows**
```powershell
Compress-Archive -Path preliminary -DestinationPath preliminary.zip -Force
```

**macOS**
```bash
rm -f preliminary.zip
zip -r preliminary.zip preliminary -x '*.DS_Store'
```
Upload it at https://techarena-uk.hydro.ac/.

**Zip the final submission**

**Windows**
```powershell
Compress-Archive -Path solution -DestinationPath solution.zip -Force
```

**macOS**
```bash
rm -f solution.zip
zip -r solution.zip solution -x '*.DS_Store'
```
Before uploading, extract the zip into an empty folder and check it contains a single top-level `solution/` folder.

---

## Saving your work to GitHub

**Windows and macOS**
```bash
git pull
git add -A
git status          # check that .env and .venv are NOT listed
git commit -m "Describe what you changed"
git push
```

---

## Rules we must follow

- **English only** in every deliverable: manifest instruction, README, design docs, every `SKILL.md`, and the video.
- `manifest.json` must use `"mode": "agent"`. Team mode is banned.
- **No hard-coded answers, event-specific scripts or unpublished thresholds** in Skills.
- **Never unpack or read the `career-emulator-techarena` data package.** It counts as cheating. Play only through the official MCP tools: `new_game`, `observe`, `take_action`, `check_latest_logs`, `show_employee_handbook`.
- **Never commit the API key.** If it might have leaked, tell the organisers on Discord immediately.

## Key dates

Scores are collected at 24:00 (UK time) on **27 Oct**, **30 Oct** and **1 Nov 2026**. The team's best score counts.

## Help

- Docs: https://techarena-career-sim.readthedocs.io/en/latest/
- Organiser Discord: https://discord.gg/Ds4JC3gJjj

## Getting official updates

**Windows and macOS**
```bash
git remote add upstream https://github.com/RAI-Internal/techarena-career-sim.git   # once only
git fetch upstream
git merge upstream/main
```
