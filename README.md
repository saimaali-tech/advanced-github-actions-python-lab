# Advanced GitHub Actions — Python Lab

A small calculator project designed to practice reusable workflows, composite actions,
matrix builds, dependencies, conditions, caching, artifacts, environments, concurrency,
workflow outputs, secrets, and runner choices.

## 1. Run locally

```bash
python -m unittest discover -s tests -v
python -m pip install build
python -m build
```

Requires Python 3.10+.

## 2. Push to GitHub

Create an empty repository, then from this folder:

```bash
git init
git add .
git commit -m "Create advanced GitHub Actions lab"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/advanced-github-actions-python-lab.git
git push -u origin main
```

The `Python CI` workflow runs on pushes to `main`, pull requests targeting `main`,
version tags (`v*`), and manual dispatch.

## 3. Concepts implemented

| Concept | Where to inspect |
|---|---|
| Reusable workflow | `.github/workflows/reusable-python-ci.yml` called by `ci.yml` |
| Composite action | `.github/actions/setup-python/action.yml` |
| Matrix builds | Python 3.10, 3.11, 3.12 in reusable workflow |
| Job dependencies | `test-and-package` needs `prepare`; `stage` needs `ci` |
| Conditional execution | `stage` runs for manual runs or version tags |
| Dependency caching | `actions/setup-python` with `cache: pip` |
| Artifacts | `actions/upload-artifact@v4` |
| Environments | `staging` environment in `ci.yml` |
| Concurrency | workflow/ref group cancels superseded runs |
| Workflow outputs | reusable workflow exposes `build-version` |
| Secrets | optional `staging` environment secret `DEPLOY_TOKEN` |
| Runner choices | GitHub-hosted by default; optional self-hosted demo |

## 4. Configure environment protection and secret

In GitHub:
1. Open **Settings → Environments → New environment** and create `staging`.
2. Add required reviewers or deployment branch restrictions if you want to test approval gates.
3. Optionally add an environment secret named `DEPLOY_TOKEN`. Use a harmless test value only;
   this lab merely checks whether it is present and does not send it anywhere.
4. Run the workflow manually or push a version tag to activate the `stage` job.

## 5. Download the versioned artifacts

Open **Actions → Python CI → latest successful run → Artifacts**.
You should see one artifact for each Python version, named similar to:

`python-package-0.1.0-run-12-py3.12`

Each contains the built distribution and `BUILD_INFO.txt`.

To create a release-style version:

```bash
git tag v0.2.0
git push origin v0.2.0
```

The artifact names will use `v0.2.0`.

## 6. Practice tasks

1. Add Python 3.13 to the matrix and observe the matrix fan-out.
2. Add a `lint` job using Ruff; make packaging depend on lint and tests.
3. Change `fail-fast` to `true`, introduce a deliberate failing test, and compare behavior.
4. Add a `workflow_dispatch` input called `target` with `test` and `staging` choices.
5. Add a job output containing the package filename and consume it in `stage`.
6. Add a path filter so documentation-only changes do not run CI.
7. Compare `pull_request` and `push` event contexts.
8. Register a self-hosted Linux runner, then uncomment the `runs-on` line in
   `self-hosted-demo.yml`. Use a disposable machine and do not expose it to untrusted PRs.
9. Add a protected `production` environment and require reviewer approval before a release job.
10. Replace the demonstration stage step with a real deployment only after configuring
    a least-privilege credential and a safe target.

## 7. Suggested learning sequence

- First inspect `ci.yml` and trace how it calls the reusable workflow.
- Then inspect the composite action and matrix expansion.
- Trigger a push, pull request, manual run, and `v0.2.0` tag.
- Inspect each job's `needs`, conditions, outputs, logs, and uploaded artifacts.
- Finally, experiment with environment approvals and a self-hosted runner.

**Security note:** The workflows use read-only repository permissions. Do not expose secrets to
untrusted pull-request workflows, and do not use a self-hosted runner for untrusted code.
