# Portfolio

This repository holds one person's portfolio of side projects as data: what is in
progress, what waits and until when, what was decided and what was verified.
[portfolio-ops](https://github.com/konradcinkusz/portfolio-ops) checks it on every push,
keeps one weekly-review issue current, and renders a dashboard you download from the
workflow run.

## Creating your repository from this template

This template is public and holds only fictional examples. Your copy must not be public.

1. Choose **Use this template → Create a new repository**, and make it **Private**.
   portfolio-ops refuses to run on a public repository: these files list your risks,
   rejections and stalled projects.
2. In your new repository, open **Actions** and enable workflows if GitHub asks.
3. Replace the fictional examples with your own, and delete this section.

## Replacing the examples

`validate` runs on every push to `main` and every pull request, and names anything
missing, with the line to fix. A good order:

1. **`config.yaml`** — your contexts (where external moves happen: a store, a grant, a
   talk) and your capabilities (what products do, in words you will reuse). Keep the
   thresholds until you have a reason to change them.
2. **`products.yaml`** — each product with its status. An active product needs a
   `next_action`; a paused or dormant one needs a `status_reason` and a `review_by` date.
   At most `wip_limit` products may be active.
3. **`decisions.md`** — start the log. Every status change gets a decision dated the day
   it happens, and each week gets a `focus`.
4. **`kernels.yaml`, `risks.yaml`, `findings.yaml`** — empty them, or delete them: they
   are optional. Fill them in as shared code, risks and verified facts come up.

| File | Holds |
|---|---|
| `config.yaml` | settings, thresholds and vocabularies |
| `products.yaml` | your products: idea, active, paused, dormant or archived |
| `decisions.md` | the decision log: `## <YYYY-MM-DD> · <id>[, <id>…] · <type>` per decision |
| `kernels.yaml` | shared code products reuse, and how |
| `risks.yaml` | what could go wrong, and where it matters |
| `findings.yaml` | what was verified, and until when it holds |

The format, and what every rule checks, is in the
[business rules](https://github.com/konradcinkusz/portfolio-ops/blob/main/docs/business-rules.md).

## What runs, and when

`.github/workflows/portfolio.yml` runs portfolio-ops as a GitHub Action:

- **On every push to `main` and every pull request:** `validate`. A failed check names the
  file, the line and the fix.
- **Every Monday at 07:00 UTC, and on demand from the Actions tab:**
  - `report` keeps one issue labelled `weekly-review` current. It lists what is stale,
    overdue or unrecorded, and closes the issue when nothing is.
  - `dashboard` renders the whole portfolio as one page, kept as the workflow artifact
    `portfolio-dashboard` for seven days. **Never publish it with GitHub Pages.**

The workflow pins portfolio-ops to a release by its full commit SHA. Dependabot opens a
pull request when a new release comes out: read its
[changelog](https://github.com/konradcinkusz/portfolio-ops/blob/main/CHANGELOG.md), and
merge once `validate` passes.

## On your machine

The gates and the lookup are for the moment you are about to act. Run them in a clone of
this repository:

```bash
pipx install git+https://github.com/konradcinkusz/portfolio-ops@v0.3.0
portfolio-ops validate
portfolio-ops gate <product> --context <context>    # may this external move go ahead?
portfolio-ops idea-gate <idea>                      # does an existing product do this already?
portfolio-ops lookup <subject> <type>               # was this verified already?
portfolio-ops dashboard --output dashboard.html     # the whole portfolio, in a browser
portfolio-ops export                                # context to paste into an LLM session
```
