# WPA Poster of the Day: TRMNL Frame recipe requirements

Nov 16, 2026 · @Niko

## Summary

WPA Poster of the Day shows one hand-picked 1930s WPA poster per day, full screen, with a small museum-style label. The posters are silkscreen prints made with a few flat inks, which makes them close to the ideal source image for a six-ink Spectra 6 panel.

The target is the mass market Frame buyer who bought Frame partly as art. The posters are bold, decorative and slightly unfamiliar, so they work as a conversation piece, and one poster a day matches Frame's slow refresh.

The recipe never calls the Library of Congress at render time. A curated, pre-processed set of posters is self-hosted on Cloudflare, and the recipe only picks today's entry.

Build on the Framework `color-6a` palette and native layout classes until Frame gets its own device profile.

## Source collection and rights

The source is the Library of Congress WPA Poster Collection: 907 posters made from 1936 to 1943, all digitized and viewable outside the Library. The Library states there are no known restrictions on these posters, but every poster's own record is still checked during curation.

- **Rights.** Item records carry the Rights Advisory "No known restrictions on publication." Curation keeps only posters whose record says exactly that, and logs the record ID.
- **Credit line.** The Library's requested form is "Library of Congress, Prints & Photographs Division, WPA Poster Collection" plus the reproduction number. It goes on screen in the label.
- **API.** The loc.gov JSON API needs no key: add `fo=json` to any loc.gov URL. A request to the collection's JSON endpoint returned an error on 2026-11-16, so the fields and image sizes are not verified yet.
- **Image files.** Item pages offer JPEG and TIFF downloads. The largest size available per item still has to be confirmed.

## Curation process

Every poster is picked by hand. The target is 365 for a full year without repeats, with at least 200 at launch. Random picks from all 907 would surface too many that don't belong on a living room wall.

**Keep:** travel and national parks, theater, music and dance events, art exhibitions, nature and seasons, city scenes, bold graphic designs.

**Drop:** public health warnings, text-heavy class and program announcements, posters whose scan is damaged or badly faded, and anything with a demeaning depiction of people (some 1930s material has this, so it needs a deliberate check).

**Steps**

1. A harvest script pulls the whole collection's metadata through the loc.gov JSON API into a sheet.
2. Each poster gets a keep or drop decision with a one-word reason.
3. Kept posters go through image preparation and a test render.
4. A poster that renders badly is dropped, not forced through.
5. The final list is committed to the GitHub repo as `posters.json`.

**Fields per kept poster**

| Field | Example | Source |
| --- | --- | --- |
| LoC record ID | 98516606 | Harvest |
| Title | As written on the record | Harvest |
| Artist | Name, or "Unknown artist" | Harvest |
| Date | "between 1936 and 1939" kept as text | Harvest |
| Place | State or city, from the record | Harvest |
| Reproduction number | For the credit line | Harvest |
| Theme | travel, arts, nature, city | Curator |
| Season | winter, spring, summer, autumn, any | Curator |
| Crop box and white point | Pixel values | Image prep |

## Image preparation for Spectra 6

The biggest risk is aged paper: a cream background turns into a dithered yellow haze on six inks. Every poster is cleaned once, offline, before it is ever served.

1. Download the largest file the Library serves (TIFF where available).
2. Crop the scan border, color target and mount board.
3. Set the white point from the paper, so the paper becomes pure white.
4. Raise contrast slightly to counter fading.
5. Optionally reduce to the poster's own ink count, so flat ink areas stay flat instead of speckled.
6. Resize to Frame's native resolution (unknown until TRMNL publishes it) and save as PNG.

**Conversion choice, to test.** Either serve the cleaned full-color PNG and let TRMNL's Lazarus engine convert it to the six inks, or pre-dither to the six inks ourselves. Compare both on the first 20 posters and keep one method for all.

Every kept poster is rendered and looked at once after prep. The whole list is checked, not a sample.

## Layouts: portrait, landscape with mat and label

Portrait is the native layout, because the posters are portrait. Landscape turns the spare width into a gallery-style mat and label.

**Portrait.** The poster fills the screen height, centered, on a white mat. A slim label strip at the bottom: title, artist and date on one line, the credit line below in smaller text.

**Landscape.** The poster sits on the left at full height. The label panel on the right reads like a museum wall card:

- Title, large
- Artist, date, place
- "WPA Poster Collection, Library of Congress" with the reproduction number

**Rules for both**

- The poster is never cropped or stretched to fit. A wide poster just gets a deeper mat.
- Label text is black on white only. Colored text on Spectra 6 is reserved for the art.
- Long titles are clamped to two lines with Framework text classes.
- No clock, weather or other widgets. The poster is the whole point.
- With Show label off, the poster sits alone on the mat.

## Selection logic and user settings

Everyone sees the same poster on the same day, so it works as a shared "poster of the day" people can talk about. No state is stored anywhere.

**Selection.** The curated list is shuffled once with a fixed seed. Today's poster = days since a fixed start date, modulo the list length. With Season matching on, the list is ordered so winter posters land in winter and summer posters in summer.

| Setting | Options | Default |
| --- | --- | --- |
| Show label | On, Off | On |
| Themes | Any mix of travel, arts, nature, city | All |
| Season matching | On, Off | On |

A theme filter selects from the same daily order, skipping posters outside the chosen themes, so two users with the same filter still match. Orientation follows the device; there is no setting for it. The recipe refreshes once a day, early morning local time.

## Architecture and hosting

All heavy work happens once, offline. At render time the recipe makes a single small request to a Cloudflare Worker.

1. **Harvest** (GitHub Actions, manual run): pull collection metadata from loc.gov into a CSV.
2. **Curate** (by hand): keep or drop each poster in a sheet.
3. **Prepare** (GitHub Actions): download, crop, white balance and resize kept posters; write `posters.json` and the PNGs.
4. **Host**: images and `posters.json` on Cloudflare (R2 or Pages).
5. **Serve**: a Worker endpoint returns today's poster for the given settings, with image URL, label text and credit line.
6. **Render**: the recipe polls the Worker; Liquid lays it out with Framework classes.

Adding posters later means re-running steps 2 to 4. Adding posters changes the list length, so the daily order changes too. To prevent that, new posters are appended after the current cycle ends rather than shuffled in.

## TRMNL recipe outlines

### Shared recipe setup

- **Recipe.** Create one full-screen Private Plugin named **WPA Poster of the Day**. In the Full markup, use the device’s orientation to switch between the two layouts below. Keep common Liquid markup in Shared. Use Framework 3.4 native classes and `color-6a` for Spectra 6 previews until the Frame device profile is available. [TRMNL Private Plugins](https://help.trmnl.com/en/articles/9510536-private-plugins) · [Framework responsive layouts](https://trmnl.com/framework/docs/3.4/responsive) · [Framework color palettes](https://trmnl.com/framework/docs/3.4/color_palettes)
- **Polling.** Set the strategy to Polling with one `GET` to the Cloudflare Worker and a 1,440-minute refresh interval. Confirm the first eligible Frame check-in shows the new poster in the early-morning local window. TRMNL refresh timing follows device check-ins and playlist schedules. [Polling strategy](https://help.trmnl.com/en/articles/17434705-plugin-strategies) · [Import settings](https://help.trmnl.com/en/articles/10542599-importing-and-exporting-private-plugins) · [Refresh timing](https://help.trmnl.com/en/articles/10113695-how-refresh-rates-work)

| TRMNL form field | Type and default | Used by |
| --- | --- | --- |
| `show_label` | Boolean, On | Liquid omits the label and gives its space back to the art when Off. |
| `themes` | Multi-select, all four themes | The Worker accepts travel, arts, nature and city. An empty selection means All. |
| `season_matching` | Boolean, On | The Worker chooses the seasonal or unseasonal schedule. |

### Portrait outline

1. Use a full-screen white mat with a vertical flow. Center the PNG in all the height available above the optional bottom label. Fit with `image--contain` so wide artwork stays whole and has more white mat around it.
2. Add a slim bottom label. Set title, artist and date on the first line when they fit. Let long titles use up to two lines with `data-clamp="2"` and move artist and date to the next line when needed. Place the full Library of Congress credit and reproduction number below in smaller text.
3. When Show label is Off, center the poster in the entire white mat.

### Landscape outline

1. Use a full-screen white mat with a horizontal flow. Place the PNG in a full-height area on the left, centered within that area and fitted with `image--contain`.
2. Put a white museum label panel to the right. Set the title in large black text with `data-clamp="2"`, then display artist, date and place when supplied. Put the credit and reproduction number below. Use the full Library of Congress credit in both layouts.
3. When Show label is Off, remove the panel and center the poster across the full white mat.

### Worker and Liquid handoff

- **Request.** Pass the theme subset, season matching value and the TRMNL account’s IANA time zone through the polling URL. Use the time zone to derive the local calendar date, including across daylight-saving changes. Normalize theme order and use All if a filter leaves fewer than 30 posters. Show label affects markup only. [Custom form fields](https://help.trmnl.com/en/articles/10513740-custom-plugin-form-builder) · [TRMNL time-zone variable](https://help.trmnl.com/en/articles/10693981-advanced-liquid?q=liquid)
- **Selection.** With season matching Off, filter the fixed seeded order and index it by elapsed calendar days from the fixed start date, modulo the eligible count. With season matching On, use the same seeded order to assign eligible posters to dates within their tagged season, with `any` posters filling gaps. Check coverage and repeat intervals for filtered schedules. Keep the active catalog fixed through the current rotation and append new entries for the next cycle. The same date and normalized settings select the same poster in both layouts.
- **Response.** Return one JSON object containing `record_id`, `image_url`, `title`, `artist`, `date`, `place` and `credit_line`. Use `Unknown artist` when needed and preserve the date text from the record. Render the complete `credit_line` without truncation. The image request goes to Cloudflare. The Library of Congress is used during curation and image preparation.
- **Fallback.** Keep a fixed, rights-checked fallback PNG and its label accessible to the Worker independently of the primary image origin. If the selected image or its origin is unavailable, return the fallback record so its art and credit stay paired. Test the outage in TRMNL’s hosted render as well as in the editor preview. [TRMNL render troubleshooting](https://help.trmnl.com/en/articles/12814634-plugins-base-process-standarderror-private_plugin)

### Preview and release checks

- Preview both orientations with Show label On and Off. Check a long title, unknown artist, missing place and a wide poster. Review each curated poster for whole-image fit, white paper and label legibility at 2 m. [Framework image fit](https://trmnl.com/framework/docs/3.4/image) · [Framework text clamp](https://trmnl.com/framework/docs/3.4/clamp)
- Confirm the same local date and settings select the same record across installs. Exercise a seasonal boundary, a filter below 30, a rotation wrap and a primary image-origin outage. Verify the Frame profile, hosted fallback render and early-morning refresh before submitting the Private Plugin for Recipe review. [TRMNL recipe publishing](https://help.trmnl.com/en/articles/10122094-plugin-recipes)

## Edge cases

| Case | Required behavior |
| --- | --- |
| End of the list | Loop to the start of the same shuffled order |
| Theme filter leaves fewer than 30 posters | Fall back to All, so nobody sees a repeat every few weeks |
| Landscape poster on a portrait device | Shown whole, centered, with a deeper mat |
| Image fails to load | The Worker serves a fixed fallback poster, never a blank screen |
| Unknown artist or date | "Unknown artist"; the date is shown as written on the record |

## Open questions and to do

- [ ] Verify the loc.gov collection JSON endpoint and its fields with a live response (failed on 2026-11-16)
- [ ] Confirm the largest image size the Library serves per poster
- [ ] Write the harvest script and run the first curation pass, target 200 kept
- [ ] Compare TRMNL's Lazarus conversion with our own pre-dither on 20 posters
- [ ] Get Frame's resolution and device keyname; re-check https://trmnl.com/framework/docs/3.4/devices.md (checked 2026-11-16: no Frame entry)
- [ ] Build the Worker and the Liquid templates (.txt, native Framework classes)

## Acceptance criteria

1. Every poster in `posters.json` has a logged record ID whose Rights Advisory reads "No known restrictions on publication."
2. Every kept poster, not a sample, is rendered in both orientations and passes review.
3. Paper areas render as white, with no yellow haze.
4. The label is legible at 2 m, and the credit line is on screen whenever the label is on.
5. Two users with the same settings see the same poster on the same date.
6. With the image host down, the fallback poster renders.

## Sources

- [WPA Poster Collection, Library of Congress](https://loc.gov/pictures/collection/wpapos), checked 2026-11-16
- [Example item record with Rights Advisory](https://www.loc.gov/item/98516606), checked 2026-11-16
- [WPA posters rights and credit line (archived LoC page)](https://webarchive.library.unt.edu/eot2008/20090114054823/http:/memory.loc.gov/ammem/wpaposters/res.html)
- [loc.gov JSON API guide (archived)](https://webarchive.library.unt.edu/web/20201218004151/https://libraryofcongress.github.io/data-exploration)
- [TRMNL Framework 3.4 color palettes](https://trmnl.com/framework/docs/3.4/color_palettes.md), checked 2026-11-16
