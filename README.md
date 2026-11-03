# PL / COP Meta ads automation

Workflow that takes pictures and videos from Google Drive and launches them on Meta
as **1 campaign → 1 ad set "Broad" → Pic 1-10 / Video 1-10**, set up the way Meta's Andromeda delivery system works best
(broad audience, Advantage+, CBO, many different creatives).

Claude runs it with the Google Drive and Meta Ads connectors. Nothing goes live without your "GO".

## How to use

In Claude Code, say for example:

> Launch the videos in Drive folder "October - COP / Nov". Budget RM50/day, Malaysia, Malay.

> New PL campaign from folder "Dec PL pics", cities KL + Shah Alam, Malay + English.

Claude will:

1. List the files in the folder and map them to `Video 1…10` / `Pic 1…10`.
2. Show you the plan (page, budget, location, languages, copy) and **wait for you to confirm**.
3. Upload the media to the ad account, then create the campaign, the "Broad" ad set and the ads as a **draft**.
4. Show you the final summary, and publish only when you reply **GO**.
5. Save every ID in `campaigns/<date>-<product>.yaml`.

The step-by-step instructions Claude follows are in `.claude/skills/launch-meta-ads/SKILL.md`.

## Files

| Path | What it is |
|---|---|
| `config/defaults.yaml` | Ad account, pages, ad set template, city coordinates, language IDs |
| `campaigns/*.yaml` | One file per launch: folder, creatives, budget, location, copy, resulting IDs |
| `.claude/skills/launch-meta-ads/SKILL.md` | The runbook Claude follows |

## Drive rules

- Name files with a number: `COP V1.mp4 … COP V10.mp4`, `COP P1.png … COP P10.png`.
  Files with random names get numbered by upload time, and Claude shows you that order to check.
- Share the folder as **Anyone with the link: Viewer**, because Meta downloads the files straight from Drive.

## Things to know

- **Cities** are targeted as a radius (default 25 km) around the city centre. Claude always shows the list before creating anything.
- **Languages:** Malay uses the same Meta language ID as your current COP ad set. The English and Chinese IDs get checked in Ads Manager on the first launch that uses them.
- **Special ad category:** property and loan ads may count as Meta's HOUSING or FINANCIAL PRODUCTS categories.
  Your current campaigns run without one. If Meta asks for one, it limits targeting: no age limits, 15 mile minimum city radius.
- **Instagram:** the connector finds no Instagram account linked to this ad account. To show ads under your own Instagram profile, link the account in Business Settings first.
