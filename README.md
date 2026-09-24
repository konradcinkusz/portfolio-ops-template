# Portfolio

This repository holds one person's portfolio of side projects as data: what is in
progress, what waits and until when, what was decided and what was verified.
[portfolio-ops](https://github.com/konradcinkusz/portfolio-ops) checks it on every push,
keeps one weekly-review issue current, and keeps an overview of the whole portfolio in
`overview/`: pages GitHub renders here, with charts. With a read-only token it also scans
your GitHub account, and names the repositories you worked on outside the plan.

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
   At most `wip_limit` products may be active. `repos` names a product's GitHub
   repositories, for the account scan.
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
- **After every push to `main`, every Monday, and on demand:** `overview` renders the
  whole portfolio as Markdown pages, and a second job commits them to `overview/` when
  they changed (see [The overview](#the-overview)).

The workflow pins portfolio-ops to a release by its full commit SHA. Dependabot opens a
pull request when a new release comes out: read its
[changelog](https://github.com/konradcinkusz/portfolio-ops/blob/main/CHANGELOG.md), and
merge once `validate` passes.

## The overview

The weekly issue lists only what needs you. For the whole picture, open
`overview/README.md` — on the web or in GitHub's mobile app. The workflow writes it after
its first run on `main`, and the weekly issue links it:

| Page | Holds |
|---|---|
| `README.md` | the numbers at a glance, what needs attention, this week's focus and next actions, where the week's work went, the latest decisions |
| `products.md` | every product by status and every kernel, with the review dates on a timeline |
| `repositories.md` | your account's repositories, those you worked on this week first |
| `risks.md` | the risks, the findings and the copy-paste debt |
| `decisions.md` | every decision, newest first, with its text |

The pages are generated: edit the data files, never `overview/`, and the workflow
rewrites them. Only people who can read this repository can see them; keep it private.
The job that commits them is the only one that can write here, runs none of
portfolio-ops' code and touches nothing but `overview/`. If you protect `main` against
direct pushes, that job fails, and the pages stay an artifact of the run.

## Scanning your account

The workflow's own token sees only this repository. With a second, read-only token, the
report, dashboard and overview jobs also scan your GitHub account. They find every
repository you own and which of them **you** pushed to since the same weekday last week;
pushes by Dependabot or other bots do not count. The weekly issue then names:

- repositories you worked on that no product or kernel lists;
- products that are not active, but were worked on;
- repositories your `repos` lists that the account does not have.

The dashboard and the overview list all of them, with the product or kernel each one
belongs to. To set it up:

1. On GitHub, open **Settings → Developer settings → Personal access tokens → Fine-grained
   tokens → Generate new token**.
   - **Resource owner:** your account.
   - **Expiration:** a date you will remember, for example a year ahead.
   - **Repository access:** All repositories. GitHub preselects "Public repositories",
     which leaves every private repository out; the weekly issue says so when that
     happens.
   - **Permissions:** nothing to add; the read-only `Metadata` permission every token has
     is enough.
2. In this repository, open **Settings → Secrets and variables → Actions → New repository
   secret**, name it `PORTFOLIO_ACCOUNT_TOKEN`, and paste the token. Paste it nowhere
   else.
3. Add `repos: [OWNER/NAME, …]` to your products and kernels, and list the repositories
   that are not projects under `account.ignore` in `config.yaml`.
4. Open **Actions → portfolio → Run workflow** to see the first scan now instead of on
   Monday.

A classic token (`ghp_…`) is refused: it cannot be read-only. When the token expires, the
weekly issue says so, and everything else keeps working. Without the secret, the account
is simply not scanned.

## On your machine

The gates and the lookup are for the moment you are about to act. Run them in a clone of
this repository:

```bash
pipx install git+https://github.com/konradcinkusz/portfolio-ops@v0.5.0
portfolio-ops validate
portfolio-ops gate <product> --context <context>    # may this external move go ahead?
portfolio-ops idea-gate <idea>                      # does an existing product do this already?
portfolio-ops lookup <subject> <type>               # was this verified already?
portfolio-ops dashboard --output dashboard.html     # the whole portfolio, in a browser
portfolio-ops export                                # context to paste into an LLM session
```
