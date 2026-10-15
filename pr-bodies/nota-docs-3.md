Before: the docs site wore Ember Graphite / Ember Paper (warm browns, brass accent), had Starlight's Dark / Light / Auto dropdown, its app screenshots showed the Ember app, and its text named Ember-era labels and colours.

After: the site wears Studio Dark / Studio Light (grey grounds, panels on dark seams, one orange accent for the current page and primary button), with the same sun/moon toggle as the Nota site, following the system theme until a reader picks one. Every app and Nota Remote screenshot a page shows is a fresh Studio render, served as AVIF, and the text matches the Studio app.

How: `src/styles/ember.css` becomes `studio.css`, with values taken by token name from the app's `Theme/Themes/studio-*.json`; sidebar and table labels are sentence case without tracking, and corners use Studio's 2/3/4 px radii. `ThemeToggle.astro` replaces Starlight's `ThemeSelect` and keeps Starlight's storage key, so its pre-paint script still applies the theme; a pick holds for the visit when storage is blocked, and the browser's theme colour follows the site. `Shot.astro` emits AVIF.

All 149 app shots were re-rendered by `shots/app` against nota `feat/remote-studio`, and the Nota Remote screens from the Studio mock. New harness scenes (`connect`, browser `expand`/`pick`, device-window `window`) replace the last three Ember mockups pages used: Connect phone, the preview player and the Bass device window. `shoot.mjs` now counts a shot the harness never rendered as a failure (a long run had stopped silently part-way), and the manifest follows Studio's sentence-case labels.

Page text, EN and RU: UI labels are sentence case as the app shows them, accent marks are no longer called brass (Prism's bands are olive / neutral ink / teal), the view switcher is described as icons, and the shortcuts page is regenerated from the app (`scripts/shortcuts.mjs` now keeps the Russian group headings). The logo/icon and unused design mockups are unchanged.

Stacked on `docs/appearance-studio` (upstream nota-daw/nota-docs#1). `npm run check`: 0 errors; `astro build`: 207 pages.

## Screenshots

Rendered from this branch. The logo still shows the old brass dial here, because it is recoloured in a separate PR.

| Home, Studio Dark | The main window page, Studio Light |
|---|---|
| <img src="https://raw.githubusercontent.com/chrislaupama/nota-docs/pr-assets-qsdspf/docs/home-dark.png" width="440"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota-docs/pr-assets-qsdspf/docs/main-light.png" width="440"> |

**A device page in Studio Dark**, with its fresh Studio app render:

<img src="https://raw.githubusercontent.com/chrislaupama/nota-docs/pr-assets-qsdspf/docs/volt-dark.png" width="640">
