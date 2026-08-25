# MLOPS+AI Project

Parallel learning tracker for two people prepping for one shared build. Six weeks of foundations on two separate tracks, converging into a support-ticket triage system (classical ML + RAG) that starts on a fixed, non-negotiable date: **2026-10-06**.

**Live tracker:** https://tanish74.github.io/mlops-ai-learning-pipeline/
**Repo:** https://github.com/Tanish74/mlops-ai-learning-pipeline

## What's in this repo

- **`tracker/learning-pipeline-local.html`** — the tracker itself. A single self-contained HTML page with two parallel lanes (MLOps track for Tanish, AI engineering track for the friend), a merge stage that stays locked until both lanes clear stage 03, a preloaded curriculum with links, and a free-text "where I left off" field per person. It saves your checkbox state to the browser's `localStorage`, and has an **Export progress.json** button that downloads the current state in the exact schema the weekly email reads.
- **`tracker/progress.json`** — the data file the tracker exports and the GitHub Action reads. Holds `start_date`, `build_date`, `board_url`, and a `people` array (`name`, `track`, `this_week`, `left_off`). This is the file you overwrite and commit each week — it's the only thing that has to change for the reminder to stay accurate.
- **`.github/workflows/weekly-reminder.yml`** — a GitHub Actions workflow that runs every Saturday at 3:00 PM IST, reads `tracker/progress.json`, and emails both people the current week number, days remaining until the build date, what each track should be on, and each person's last "left off" note. The subject line is prefixed `IMPORTANT:` so it doesn't get lost. If nobody has committed to `progress.json` in 7+ days, the email says so instead of a generic "keep going."
- **`index.html`** — a one-line redirect at the repo root so the GitHub Pages root URL lands straight on the tracker.
- **`README.md`** — this file.

## Opening the tracker

Two ways to use it, and they don't share state — each is its own browser's `localStorage`:

- **Hosted (what to hand your friend):** open the live link above. Anyone can view it; nobody but you can push changes to the repo. Each person's checkbox state lives only in their own browser, exactly like the local copy — that's why Export/Import and the weekly email exist, to keep the two of you in sync.
- **Local:** open `tracker/learning-pipeline-local.html` directly from disk (double-click it, or open it in your browser via its `file://` path). It has to be opened from that real file location for progress to persist — `localStorage` is tied to the page's origin, so opening it from a temp copy, a preview pane, or a different path each time will look like it's not saving.

## Weekly routine

1. Study during the week; check off items in the tracker and jot a quick note in the "where I left off" field.
2. Click **Export progress.json** in the tracker. It downloads a new `progress.json`.
3. Overwrite `tracker/progress.json` in this repo with the downloaded file.
4. `git add`, commit, push.
5. Every Saturday at 3:00 PM IST, the GitHub Action reads `tracker/progress.json` and emails both of you the week number, days left until the build date, what each of you should be on, and where you last left off. The commit *is* the check-in — a stale file is what triggers the "nobody's touched this in N days" line, which is the part meant to actually create pressure.

## Setting up the email reminder (GitHub Actions)

These steps are pulled from the comment block at the top of `weekly-reminder.yml`:

1. **Commit the workflow.** `.github/workflows/weekly-reminder.yml` already lives in this repo — nothing to do here beyond pushing it to GitHub.
2. **Commit the data file.** `tracker/progress.json` also already lives in this repo (see Weekly routine above for how it stays current).
3. **Create a Gmail App Password.** Turn on 2-Factor Authentication on the Gmail account you want to send from, then go to `myaccount.google.com` → Security → App passwords, and generate one (16 characters, no spaces).
4. **Add three repository secrets.** In GitHub: repo → Settings → Secrets and variables → Actions → New repository secret:
   - `MAIL_USERNAME` — the sending Gmail address
   - `MAIL_PASSWORD` — the 16-character App Password from step 3 (not your normal Gmail password)
   - `MAIL_TO` — `imruthika2003@gmail.com,tanishve74@gmail.com`
5. **Test it manually.** GitHub → Actions tab → "Weekly learning reminder" → Run workflow. This fires it immediately so you can confirm the email arrives before waiting for the first real Saturday.

> **Note:** the workflow's original comments said to commit `progress.json` to the repo root. This copy has been edited (two path references inside `weekly-reminder.yml`) to read from `tracker/progress.json` instead, to match the folder layout used here. If you ever move the file, update those two lines too.

## Project background

**The plan.** Six weeks of foundations on two parallel tracks, then a fixed build date — 2026-10-06 — when both people start building the shared project together regardless of how "ready" either track feels. The date is deliberately non-negotiable; the known failure mode for this kind of plan is studying in parallel for months and never starting.

**Tanish's track — MLOps.** Already has the ops half (CI/CD, containers, Kubernetes, Helm, registries, monitoring, Istio, cloud). The six weeks target the gaps: Python as a first-class language, enough ML to package someone else's training code, and the ML-specific tooling layer (MLflow, DVC, model registries, drift).

**Friend's track — AI engineering.** Starting from scratch: Python fundamentals, then classical ML, then neural networks from first principles, then an applied LLMOps/RAG series.

**The shared project.** A support-ticket triage system: a classical ML model predicts category and priority, a RAG service retrieves relevant docs and drafts a suggested resolution, both surface in a small UI. Tanish owns repo structure, DVC, an MLflow tracking server on Kubernetes, CI, serving, Helm charts, canary rollout via Istio, Prometheus/Grafana, drift jobs, and the retraining trigger. The friend owns training code, experiments, retrieval, prompts, and evals. The interface between the two halves is deliberately thin: a `train.py` that logs to MLflow, and a `predict()` that loads from the registry.

**Build phases**, roughly in order: (1) notebook → package, DVC dataset, MLflow logging, FastAPI endpoint — no Kubernetes yet; (2) CI that retrains, registers, builds, and pushes; (3) RAG service as its own deployment with its own scaling profile; (4) drift monitoring, dashboards, canary rollout, auto-rollback on eval regression; (5) optional — deploy to EKS or GKE under a cost cap.

The full curriculum with links lives in the tracker itself (`tracker/learning-pipeline-local.html`) so there's one source of truth instead of two copies that can drift apart.
