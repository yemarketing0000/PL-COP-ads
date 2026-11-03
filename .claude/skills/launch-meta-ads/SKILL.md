---
name: launch-meta-ads
description: Take pictures/videos from a Google Drive folder and launch them as a Meta ads campaign (1 campaign, 1 "Broad" ad set, Pic 1-10 or Video 1-10 ads) for the PL / COP business. Use when the owner says things like "launch the new COP videos", "upload ads from Drive folder X", or "new campaign with pic 1-10".
---

# Launch Meta ads from Google Drive

Settings live in `config/defaults.yaml`. Each launch gets its own file in `campaigns/` (copy the latest one).
Tools: Google Drive MCP (`mcp__Google_Drive__*`) and Meta Ads MCP (`mcp__META__*`, load with ToolSearch).

## Hard rules
- Never publish or activate anything without the owner typing an explicit go-ahead (e.g. "GO") for this launch.
- Always show location (country or cities + radius), languages, budget, page and copy, and wait for confirmation
  BEFORE uploading media or creating anything. City coordinates come from `config/defaults.yaml`; ask if a city is missing.
- Never guess budget, page or ad copy. Ask.
- Ask the owner whether the creatives were made with AI tools (`self_ai_disclosure`); never decide it for them.

## Steps

1. **Find the files.** `search_files` with `parentId = '<folder id>'` (paginate). Keep image/* and video/* only.
   Map names: `... V<n>` -> `Video n`, `... P<n>` -> `Pic n`. Files without a number: sort by createdTime and
   number them, then show the mapping to the owner. Max 10 per launch.
2. **Check sharing.** `get_file_permissions` on each file. Needs `{"type":"anyone","role":"reader"}`.
   If missing, ask the owner to share the folder as "Anyone with the link" (or ask permission to change it).
3. **Write the plan** into `campaigns/<date>-<product>.yaml` and show the owner a short summary table.
   Wait for confirmation (this is the location/language verification step).
4. **Upload media.** `ads_creative_upload_media` with `upload_source: URL`, `media_type` IMAGE/VIDEO,
   `media_url` = `drive.direct_url_template` with the file id, `name` = ad name.
   Videos: poll `ads_get_ad_videos` (with `video_ids`) until `status` is ready; take `picture` as thumbnail.
   If Meta rejects the Drive URL (HTML interstitial), stop and tell the owner; fallback is uploading
   from Ads Manager media library, then continue from step 5 using the existing video IDs.
5. **Campaign.** `ads_create_campaign`: objective, CBO `campaign_daily_budget` = MYR x 100 (sen),
   `campaign_bid_strategy` LOWEST_COST_WITHOUT_CAP, `special_ad_categories` from config.
6. **Ad set "Broad".** `ads_create_ad_set` with `optimization_goal` CONVERSATIONS, `destination_type` WHATSAPP,
   `billing_event` IMPRESSIONS, `promoted_object` `{"page_id": "<page>"}`, targeting:
   - country: `{"geo_locations":{"countries":["MY"]},"age_min":25,"age_range":[25,50],"locales":[41],"targeting_automation":{"advantage_audience":1}}`
   - cities: replace geo_locations with
     `{"custom_locations":[{"latitude":..,"longitude":..,"radius":25,"distance_unit":"kilometer","country":"MY"}, ...]}`
   - languages: `locales` = ids of the chosen languages from config.
   No interests, no manual placements (Andromeda wants broad + Advantage+).
7. **Creatives + ads.** For each file: `ads_create_creative` (page_id, video_id + image_url thumbnail, or
   image_hash; `message` = primary text; `headline`; `call_to_action_type` WHATSAPP_MESSAGE;
   `advantage_plus_creative` true), then `ads_create_ad` named `Video n` / `Pic n` with `{"creative_id": ...}`.
   Use `hide_ui: true` when creating many objects.
8. **Check.** `ads_get_ad_entities` `object_state=draft` on the campaign. Fix every `active_errors` item.
9. **Confirm + publish.** Show the final summary (IDs, budget, location, languages, ad list). Only after "GO":
   `ads_activate_entity` on the campaign (publishes campaign + ad set + ads). Read back live status and report it.
10. **Record.** Save all IDs in the campaign yaml `result:` block, commit and push.

## Andromeda checklist (why the structure is like this)
- 1 campaign, 1 ad set, CBO: Meta's retrieval engine picks the audience; splitting into many ad sets starves learning.
- Broad targeting + Advantage+ audience: creatives do the targeting.
- 10 creatives that look different (different hook, person, format, language) beat 10 near-copies.
- Don't edit the ad set during the first 3-5 days; add new creatives as new ads instead.
