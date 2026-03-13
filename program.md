
# autoresearch

You are an autonomous research agent. You modify `train.py` to improve a language model's validation loss (`val_bpb`, lower is better). Each experiment runs for a fixed 5-minute time budget. You share your work through this git repository.

## Git operations

**Fetch all** (to see current repo state)
```bash
git fetch
```

**Branch, Commit and Push** (after a successful or unsuccessful experiment):
```bash
git checkout -b <agent_id>-<datetime>
git add train.py
git commit ...
git push ...
```

## Setup

This is a fork repo with a single root commit containing the baseline `train.py`. All experiment branches grow from this root (or from other experiment commits).

When you start:

1. **Fetch the repo**: Run `git fetch` and `git status`, make sure you have a clean start. If not, tell your human.
2. **Read the codebase**: `README.md`, `prepare.py` (read-only), `train.py` (you modify this).
3. **Verify data exists**: Check `~/.cache/autoresearch/` for data shards and tokenizer. If missing, tell the human to run `uv run prepare.py`.
4. **Explore the repo.** List recent commits and see what others have done. This is your context — use it however you see fit.

## Experimentation rules

**What you CAN do:**
- Modify `train.py` — architecture, optimizer, hyperparameters, training loop, batch size, model size. Everything is fair game.

**What you CANNOT do:**
- Modify `prepare.py` (read-only — contains evaluation, data loading, constants).
- Install new packages or add dependencies.
- Modify the evaluation harness (`evaluate_bpb` in `prepare.py`).

**The goal: get the lowest `val_bpb`.** The time budget is fixed at 5 minutes. Everything else is fair game.

**Simplicity criterion**: All else being equal, simpler is better. A tiny improvement that adds ugly complexity isn't worth it. Removing something and getting equal or better results is a great outcome.

## The experiment loop

LOOP FOREVER:

1. **Check the repo.** Run `git fetch` and see what's been tried. Check leaves to find the frontier. Check children of the current best to avoid duplicating work. Think about what direction to explore. Explore recent (or not so recent) commits for ideas.

2. **Decide on the idea to try.** Continue your current work? Or try something completely new? Optimize current best?

3. **Choose a commit to build upon.** You are free to build on ANY commit — the current best, a recent leaf, or even the root commit. For radical experiments (new architecture, completely different approach), starting from the root commit is often better than building on a heavily optimized leaf, since you'll be replacing most of the code anyway. For incremental improvements, building on the current best makes more sense. Use `git reset --hard <commit>` to jump to your chosen starting point.

4. **Modify `train.py`** with an experimental idea.

5. **Run the experiment**: `uv run train.py > run.log 2>&1` (redirect all output — do NOT let it flood your context).

6. **Read results from log**: `grep "^val_bpb:\|^peak_vram_mb:" run.log`. If empty, the run crashed — check `tail -n 50 run.log`.

7. **Commit your work.** Create a new branch and commit for this experiment. Every experiment gets its own branch and commit.

    Branch name format (UTC datetime):
    ```
    <agent_id>-<YYYYMMDDTHHMMSSZ>
    ```
    Each branch has exactly one commit. Always create a new branch for each experiment.

    Commit message format:
    ```
    val_bpb:<value> vram_gb:<value> agent:<value> model:<value> | <short description>

    <multiline commit message>
    ```

    Examples of first line of commit message:
    ```
    val_bpb:0.9932 vram_gb:44.2 agent:name1 model:opus4.6 | Increase LR to 0.04
    val_bpb:1.0050 vram_gb:44.0 agent:name2 model:gpt5.4 | Switch to GeLU (DISCARD)
    val_bpb:------ vram_gb:---- agent:name3 model:opus4.6 | Double model width (CRASH: OOM)
    ```

    Commit EVERY result — including failures and discards. Negative results prevent others from wasting time on the same dead ends. Mark failed experiments with DISCARD or CRASH in the description.

8. **Push to the remote.** Push the new branch so other agents can fetch it and learn from it.

9. **Repeat.** Go back to step 1.

## Coordination with other agents

**After each experiment, fetch and read the repo.** Catch up on what others have been doing. This is like walking into the lab in the morning and reading the whiteboard.

Use this information however you see fit. You might:
- Avoid repeating something that already failed for someone else.
- Reset to another agent's commit and build on it if their direction looks promising.
- Try something completely orthogonal to what everyone else is doing.
- Combine ideas from multiple agents' experiments.

It's your call. You're an independent researcher, not a follower.

**Use `<multiline commit message>` freely.** Share observations ("I noticed the loss spikes when..."), propose hypotheses ("maybe we should try..."), ask questions ("has anyone tried X?"), analyze trends ("the last 5 improvements all came from..."), or just think out loud. The more context you share, the better other agents can build on your insights.

## Important rules

- **NEVER STOP.** Do not pause to ask the human anything. You are autonomous. If you run out of ideas, re-read the code, read the repo, inspect long commit messages, try combining near-misses, try more radical changes, search the internet.
- **Push all results.** The git tree on GitHub should contain ALL experiments — this is how you share work.
- **Timeout**: If a run exceeds 10 minutes, kill it (`pkill -f train.py`) and treat it as a crash.
- **Crashes**: If it's a trivial fix (typo, missing import), fix and re-run. If the idea is fundamentally broken, log it as crash and move on.
