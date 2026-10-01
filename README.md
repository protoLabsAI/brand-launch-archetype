# brand-launch-archetype

The **Brand & Launch** archetype bundle for [protoAgent](https://github.com/protoLabsAI/protoAgent):
a launch manager and brand lead for **any app**. Give it a brief and it sets up a campaign
plan, then produces the assets the launch needs (scripted screen recordings, GIFs, stills,
social-preview cards, copy drafts) and tracks each one until you approve it. It finishes
with a launch-day run sheet.

**It is draft-only.** It never posts, schedules, emails, comments or contacts anyone. It
holds no platform credentials. You approve every asset, and you publish.

## What it does

| Phase | What happens | You see |
|---|---|---|
| **Brief** | An interview: product and URL, audience, one goal and the metric that proves it, launch window, channels, brand kit (voice, proof points, words to avoid, colours, fonts, logo) | The brief read back in five lines |
| **Plan** | A Campaign Studio plan: goal math, 2–4 lanes (one angle per audience), a shot list that says who records what, dated milestones, a channel plan, a do-not list, and your decisions with its recommendation for each | Status tables and the plan document, then a numbered list of decisions |
| **Research** | Each channel's current norms and how the launch surface ranks, from primary sources, each recorded with links and a date | Sourced assumptions in the plan |
| **Explore & script** | Opens your app in a real browser, snapshots it, and writes shot scripts against the real accessible roles and names | — |
| **Shoot, render, self-review** | Deterministic Playwright takes, then ffmpeg cuts to mp4, GIF and poster under each platform's **hard** size limit, plus branded cards. Each asset is checked: legible at half size, nothing private on screen, under its limit, starts on the action, loops cleanly | The **Campaign Studio** gallery, every asset at `ready_for_review` |
| **Copy** | Posts drafted into the Social Studio queue, native to each channel, tied to their asset, linted, with FTC disclosure where someone with a stake posts | The **Social Studio** board, ready to approve |
| **Run sheet** | A time-ordered launch-day sheet in your timezone: pre-flight checks, each post (channel, time, approved file, approved copy), who does what, what to watch, a day-after check. It uses approved assets and copy only | An artifact, plus the copy-ready export pack |

It stops **once per phase**, not once per step, with what it made, what it decided on its own
and why, and the decisions only you can make.

## The rules it keeps

These are written into the persona (protoAgent's `config/soul-presets/brand-launch.md`):

1. **Nothing leaves without a human.** No posting, scheduling, emailing, DMing, commenting or
   filing on your behalf. If you say "just send it", it hands you the exact text and where it goes.
2. **You are the only approver.** The agent takes an asset as far as `ready_for_review`.
   Approve and Reject are buttons in the gallery, and there is no agent tool for them. A
   rejection note becomes the spec for its next take.
3. **It never invents a number.** Stars, users, installs and benchmarks come from your brand
   kit's proof points or from you. Without one it still drafts, and names the gap under the
   draft. A missing baseline becomes a decision for you, not a guess.
4. **No competitor names and no exclusivity claims** ("the only", "the first", "#1") unless
   your brand kit allows them.
5. **Nothing private on screen.** Shot scripts mask secrets, tokens, home paths, emails and
   internal hostnames from the first frame. It checks every still afterwards and re-shoots if
   anything leaks.
6. **Norms are researched, sourced and dated, never remembered.** Clip length, posting times,
   hashtag counts and trending mechanics all come from sources it records with links and the
   date it read them. Anything with a single source is labelled a guess. Hard limits come
   from Campaign Studio's own sourced table.
7. **One checkpoint per phase.**

## What's inside

| plugin | source | role |
|---|---|---|
| `campaign` | [campaign-plugin](https://github.com/protoLabsAI/campaign-plugin) (Campaign Studio) | Plans and production: 17 `campaign_*` tools, the `campaign_producer` background subagent, the skills `campaign-planning`, `shot-scripting` and `asset-review`, and the **Campaign Studio** gallery view with the operator-only Approve and Reject buttons |
| `social` | [social-plugin](https://github.com/protoLabsAI/social-plugin) (Social Studio) | Copy: the brand kit, researched and dated platform norms, the content queue, the linter, FTC disclosure checks, the export pack, a writer, editor and researcher crew, and the **Social Studio** board view |
| `agent_browser` | builtin (protoAgent ≥ 0.165.0) | Explores the target app before scripting. Used to look only: it never signs in to third-party accounts or submits forms |
| `artifact` | builtin | Shows the plan, contact sheets and the run sheet |
| `notes` | builtin | Ideas and loose ends between sessions |

The persona is protoAgent's `brand-launch` soul preset. It ships with core, and this bundle
names it, so there is one copy to keep correct.

**Not included:** any publishing connector. Platform APIs are the expensive, brittle half of
social, and when an autonomous poster fails, the result is a permanent, screenshot-able wrong
post. Export the pack (or CSV) into whatever scheduler you already use.

## Create an agent

**Core floor: protoAgent ≥ 0.188.0** on the hub. Campaign Studio declares
`min_protoagent_version: 0.188.0`, and the loader refuses it on older cores. Bundles can't
enforce a floor themselves, so it's stated here.

### From the picker (once the archetype is listed)

The `brand-launch` row is in protoAgent's archetype catalog but **held**, so the picker
doesn't serve it yet. Once it's listed: **Fleet ▸ New agent ▸ Brand & Launch**, optionally
enter the app's URL, then **Create**.

Until then, there's a console route on any ≥ 0.188.0 hub: **Settings ▸ Plugins ▸ Install from URL**
→ `https://github.com/protoLabsAI/brand-launch-archetype`. An installed bundle with an
`archetype:` block registers itself as a picker card. This installs the member plugins
(disabled) on the hub too. If the hub's core predates the `brand-launch` preset, paste the
persona into the set-up step's **Advanced ▸ Persona** field. Without it the agent starts with
the base persona.

### From the API (works now, on any ≥ 0.188.0 hub)

This is the exact body the picker sends. The persona goes in inline, so it doesn't matter
whether the hub's core ships the preset yet:

```bash
# The persona: from a protoAgent checkout that has it, or straight from GitHub.
curl -fsSL https://raw.githubusercontent.com/protoLabsAI/protoAgent/main/config/soul-presets/brand-launch.md -o brand-launch.md

jq -n --rawfile soul brand-launch.md '{
  name: "brand-launch",
  bundle: "https://github.com/protoLabsAI/brand-launch-archetype",
  soul: $soul,
  requires_tools: ["campaign_create", "campaign_asset_update", "social_queue_add", "browser_open"],
  config_inputs: { "agent_browser.home_url": "https://your-app.example" }
}' | curl -s -X POST http://127.0.0.1:7870/api/fleet \
      -H 'content-type: application/json' \
      ${PROTOAGENT_TOKEN:+-H "authorization: Bearer $PROTOAGENT_TOKEN"} \
      -d @-
```

`7870` is the default instance; use your hub's port. Leave out `config_inputs` to give the
URL in the brief instead. The member inherits the hub's model connection
(`inherit_config: true` is the default). Add `"port": 7880` to pick its port.

Offline CLI equivalent, run on the hub's machine. It records no capability contract and
doesn't inherit the hub's model (set one in the member's Settings):

```
protoagent workspace new brand-launch --bundle https://github.com/protoLabsAI/brand-launch-archetype \
  --soul brand-launch.md --input agent_browser.home_url=https://your-app.example --port auto
protoagent fleet up
```

### Direct install onto an existing agent

```
python -m server plugin install https://github.com/protoLabsAI/brand-launch-archetype
```

Then enable the suggested list (`campaign, social, agent_browser, artifact, notes`).

## First run

1. **Media setup** (planning and copy work without it, and the media tools say exactly what's
   missing):
   - Campaign Studio's banner: **Install dependencies** (the `playwright` package), then
     **Install Chromium** (~150 MB, only from that button).
   - **ffmpeg** on PATH: `brew install ffmpeg`, `sudo apt install ffmpeg`, or
     `winget install Gyan.FFmpeg`. Or set Settings ▸ Plugins ▸ Campaign Studio ▸ *ffmpeg path*.
   - agent_browser's banner: **Download agent-browser**, then **Install Chrome**.
2. Say **"We're launching <product> at <url>. Plan the campaign."** It starts with the brief
   interview. If there's no brand kit, building it comes first (Social Studio's
   `brand-kit-setup`), because every draft and every card depends on it.

## Reuse it for any app

Nothing in the bundle or the persona is specific to one product. Everything app-specific
lives in data the agent builds with you:

- **The brand kit** (Social Studio's YAML): voice, proof points (the only numbers it may
  cite), banned words, CTAs, and the visual keys Campaign Studio reads for cards (colours,
  fonts, logo). One kit per brand. Point `social.data_dir` at a versioned folder to keep it in git.
- **The campaign plan**: one per launch. One agent can run several launches for the same
  brand: `campaign_list` shows them all.
- **Shot scripts**: stored per campaign. Re-shoot after a UI change with the same script and
  you get the same take.

For a second, unrelated brand, create a second member from the archetype so the kits and
queues stay separate.

### Worked example: protoAgent on GitHub trending

> We're launching protoAgent (https://github.com/protoLabsAI/protoAgent) for GitHub trending.
> Audience: developers building agents. Goal: stars in the first 72h. Channels: the README,
> X, Bluesky, LinkedIn, Hacker News, r/LocalLLaMA. The app runs at http://localhost:7871.

From that brief it will research how the trending list ranks and when it resets (sourced and
dated, never assumed), make the target's baseline a decision for you rather than a number,
plan lanes such as "install a plugin from a URL" and "a fleet of agents in one console",
record a hero clip and GIFs for each lane against the running console (with home paths and
tokens masked), render the 1280×640 social-preview card under GitHub's 1 MB limit, draft each
channel's post, and produce a run sheet for launch morning. You approve; you post.

## Pin lifecycle (ADR 0049)

External members are pinned to release tags, and each pin is a **floor**: installs take the
newest *compatible* release (caret semantics, so for 0.x the minor is the boundary).
`scripts/verify_bundle.py`, run by `.github/workflows/verify-bundle.yml` on every PR and
weekly, installs this manifest into a scratch agent on a fresh protoAgent checkout, loads
every member, probes each declared console view, and checks the capability contract
(`requires_tools`). `scripts/check_bundle_updates.py` opens a bump PR only for an
out-of-range release.

### Pin-bump PR lifecycle

The `bump` job (weekly, on dispatch, and on a member's `member-released` dispatch) reuses
**one** `bump-pins` branch and PR, rewritten wholesale each run, so don't hand-edit it. A PR
opened with the repository `GITHUB_TOKEN` never auto-starts its `pull_request` run: GitHub
holds it as `action_required` until a maintainer approves it. The job detects that, labels
and comments on the PR, and fails, so an unapproved candidate turns the schedule red instead
of going stale unnoticed.

While this bundle depends on core that hasn't merged, the repo variable `PROTOAGENT_REF`
(e.g. `refs/pull/<n>/head`) points `verify` at that ref. Delete it once the change lands.

Run the verify locally from a protoAgent checkout:

```
uv run --no-sync python /path/to/brand-launch-archetype/scripts/verify_bundle.py /path/to/brand-launch-archetype
```
