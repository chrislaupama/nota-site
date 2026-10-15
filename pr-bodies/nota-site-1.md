Before: the site wore the old Ember look (warm browns, brass accent, uppercase tracked kickers, rounded 6–12 px corners) and always opened in dark mode, with a Dark/Light word switch. The hero main window, the Nota Remote phone and the Settings window were hand-built Ember-era mockups, and the device gallery was made of Ember screenshots, all dark. The release version was hard-coded to 0.43.1 until the browser's GitHub call returned, the copy said 15 instruments and 46 devices, and the gallery had no Keys or Mosaic.

After: the site matches the app's Studio redesign. It opens in the visitor's system theme (Studio Dark or Studio Light), and a sun/moon icon switch overrides it. Every picture of the app is rendered from the latest app code in both themes, and the site shows the one matching its theme: the hero main window, the six Nota Remote screens, Settings → Downloads and all 149 device gallery views (now including Keys and Mosaic). The screenshots are AVIF, 6.7 MB for all 314 of them. A new Contributors section follows Open source, and the release version, its downloads and the contributor list are snapshotted from GitHub on every deploy and once a day. The instrument, effect and device counts in the copy (16 instruments, 25 audio effects, 6 MIDI effects, 48 devices) come from the gallery, so they stay correct as devices are added.

Colour values come from the app's `Theme/Themes/studio-dark.json` and `studio-light.json`. Page layout and copy are unchanged, apart from one sentence about the phone and the counts (see below).

How:
- `src/styles.css` maps each page token to a Studio role (ground, panel, well, raised, seam, rule, ink steps, accent), and the `--brass*` variables are renamed to `--accent*`. In light mode, link text uses Studio Light's darkest accent ink (`AccentPale`), because `AccentBright` is below 4.5:1 on the light ground.
- Theme: `index.html` applies the saved choice or else `prefers-color-scheme` before first paint, and sets the browser's `theme-color` to match; `useTheme` keeps following system changes until the visitor picks one (remembered in storage, and in memory where storage is blocked).
- Images: `<name>.avif` (Studio Dark) and `<name>-light.avif` (Studio Light), encoded at quality 60 from the lossless 2x renders; `useShot()` picks the one for the current theme. The hero is fetched with high priority; the Remote and Settings images load lazily.
- The hero and Settings → Downloads were rendered headlessly at 2x with the nota-docs `shots/app` harness from the newest nota branch (feat/remote-studio plus the review fixes). The Downloads page lists a local build of nota-plugins-registry's `index.json`.
- The phone shows screenshots of the real Remote web app (`src/web/remote`, `?theme=studio-dark` / `studio-light`) against its mock server at 390 × 844, 2x. The screen chips beside it switch between them. The phone used to be an interactive mock, so "The phone here is live: tap the pads…" became "Pick a screen below to see it."
- The interactive mocks are removed: `MainWindow`'s mock layout, `hero/Transport.tsx`, `remote/Phone.tsx`, `plugins/SettingsWindow.tsx` and the old hero Devices panel image.
- Device gallery: one view per real tab in the app, or a factory preset for devices without tabs (labelled with the preset name). Each render was checked to show the tab or preset its label names; the light renders use the same scenes and select the same tabs.
- GitHub figures: `npm run snapshot` (`scripts/github-snapshot.mjs`) writes `src/data/github.json` with nota-daw/nota's latest release (version, date, assets) and the contributors summed across nota, nota-docs, nota-site and the registries (bots skipped). `deploy.yml` runs it before `npm run build`, with the workflow's token, on every push to main and daily at 06:00 UTC. If GitHub can't be reached it keeps the committed file, so a build never fails on it. `release.ts` starts from the snapshot and still refreshes from the live API in the browser.
- The committed snapshot is a seed (v0.53.1, and contributors from the repos' git authors) because the GitHub API wasn't reachable from where this was built; the first deploy replaces it with GitHub's own figures and avatars.
- Counts: `src/lib/counts.ts` derives them from `devices.json` (racks are not counted as instruments); the meta description gets the device count through a small Vite `transformIndexHtml` plugin.

## Screenshots

**Hero in both themes.** The page opens in the visitor's system theme; the sun/moon switch overrides it.

| Studio Dark | Studio Light |
|---|---|
| <img src="https://raw.githubusercontent.com/chrislaupama/nota-site/pr-assets-qsdspf/site/hero-dark.jpg" width="440"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota-site/pr-assets-qsdspf/site/hero-light.jpg" width="440"> |

<details><summary>The whole page in both themes</summary>

| Studio Dark | Studio Light |
|---|---|
| <img src="https://raw.githubusercontent.com/chrislaupama/nota-site/pr-assets-qsdspf/site/full-page-dark.jpg" width="360"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota-site/pr-assets-qsdspf/site/full-page-light.jpg" width="360"> |

</details>
