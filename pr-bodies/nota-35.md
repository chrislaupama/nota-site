## Summary

Nota Remote, the phone controller Nota serves on the local network, never got the Studio look. It still wore the old Ember palette, and the two Ember palettes were hard-coded in the page, so a phone next to a Studio-themed Nota looked like another app. This PR makes the phone match the desktop:

- **It wears Nota's theme, live.** Nota now sends the phone the colours of the theme it's wearing (every token from `Theme/Themes/*.json`), plus your *In dark mode* / *In light mode* picks. The phone puts them on the page as CSS custom properties. Pick another theme in Preferences → Appearance, or let Follow system flip it, and the phone follows within milliseconds. No reload, and held notes and faders aren't touched.
- **Follow phone uses your picks.** The phone's own ⋯ → Theme → **Follow phone** now wears your dark-mode pick when the phone is dark and your light-mode pick when it's light. It switches the moment the phone's scheme flips, without waiting for Nota. The menu shows which theme it's wearing and why.
- **Every screen reads as Studio.** Pads, Keys, XY, Mixer, Macros, Scenes, the big transport, the header, every sheet, the pairing and connection covers, the count-in and the toast all follow the desktop's rules:
  - neutral greys, with the orange accent only for what is live or chosen;
  - knob arcs, faders, the XY point and the pan dot neutral at rest, and orange only while a finger moves them;
  - mute amber, solo blue, arm and record red, solid, with ink derived from the fill;
  - panels on 1 px seams, radii 2 / 3, sentence-case labels with no tracking;
  - Geist for words and Geist Mono for numbers, with U+2212 minus and thin spaces before units.
- **No colour lives in the phone any more.** The page holds no colour value at all. Its first-paint fallback (Studio Dark, before Nota has said anything) is generated at build time from the theme files by a small Vite plugin. A phone that has connected before paints its last theme on the very first frame, from a stored copy applied by a head script before the app loads.

Stacked on `feat/studio-ui`, which is open upstream as nota-daw/nota#34, and up to date with it (develop 0.53.1 included). Once #34 merges, this branch moves onto `develop` and opens on nota-daw. Spec: [`specs/014-remote-studio/`](https://github.com/chrislaupama/nota/blob/feat/remote-studio/specs/014-remote-studio/spec.md) (spec · plan · research · data model · contracts · tasks, converged).

## How it works

**Hub → phone: one new `theme` message** ([contract](https://github.com/chrislaupama/nota/blob/feat/remote-studio/specs/014-remote-studio/contracts/theme-message.md)):

```json
{ "t": "theme", "on": "studio-dark", "dark": "ember-graphite", "light": "studio-light",
  "ink": ["#17181A", "#FFFFFF"],
  "th": { "studio-dark": { "name": "Studio Dark", "dark": true, "c": { "BgApp": "#1E1F22", "…": "…" } }, "…": {} } }
```

- `IRemoteHost.DarkTheme` (a bool) becomes `IRemoteHost.Appearance`, a `RemoteAppearance` record (active theme, dark pick, light pick, ink pair). The app caches it on `(NotaPalette.Theme, NotaThemeService.Choice)` and hands back the same instance until one of them changes. The hub serialises only when that reference changes and sends a phone the string only when it differs from what that phone last got. That makes the ticks in between free, and a change costs one ~8 KB message per phone. It goes out before `proj`, so the first screen paints in the right colours.
- `proj.dark` stays, so a page cached from an older Nota still gets dark or light (the fallback's two sides).
- `NotaPalette.ReadableInkOn`'s two inks are named (`InkDark` / `InkLight`) and sent, so the phone derives text on a solid fill (accent, mute, solo, record, a track colour) exactly the way the desktop does.

**Phone:**
- `lib/palette.ts` turns tokens into custom properties (`BgApp` → `--bg-app`) and computes the WCAG ink choice. It's pure and shared with the build.
- `lib/theme.svelte.ts` holds the stored message and a live `prefers-color-scheme` listener, picks the worn theme for Follow Nota / Follow phone, applies it, tints `theme-color`, and stores the result for the head script.
- `theme-plugin.ts` turns `virtual:nota-theme.css` into the fallback, generated from `studio-dark.json` / `studio-light.json`, and fills the `theme-color` placeholder in `index.html`.
- `app.css` and every component were rewritten against the tokens. The old `.caps` style is gone.

**Dev tool: `npm run mock`** (`src/web/remote/mock/nota-mock.mjs`, no dependencies) serves the built page with a stand-in Nota that speaks the Remote protocol: a demo project, transport, mixer, XY, macros, Session and real theme messages. `?theme=` and `?pick=` set the theme, and `GET /mock?theme=…` switches it live. That's how the screenshots below were made, since the docs harness can't render the phone UI.

## Screens

Phone width (390 × 844), rendered headlessly in Chrome against the stand-in. *Before* is `feat/studio-ui` (the Ember look); *after* is this branch.

| Screen | Before · dark | After · Studio Dark | Before · light | After · Studio Light |
|---|---|---|---|---|
| Pads | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-pads.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-pads.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-pads.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-pads.png" width="200"> |
| Keys · keyboard | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-keys.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-keys.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-keys.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-keys.png" width="200"> |
| Keys · scale | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-scale.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-scale.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-scale.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-scale.png" width="200"> |
| Keys · chords | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-chords.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-chords.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-chords.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-chords.png" width="200"> |
| XY | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-xy.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-xy.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-xy.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-xy.png" width="200"> |
| Mixer | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-mixer.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-mixer.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-mixer.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-mixer.png" width="200"> |
| Macros | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-macros.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-macros.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-macros.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-macros.png" width="200"> |
| Scenes | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-scenes.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-scenes.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-scenes.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-scenes.png" width="200"> |
| Big transport | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-transport.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-transport.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-transport.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-transport.png" width="200"> |
| Track sheet | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-tracks.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-tracks.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-tracks.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-tracks.png" width="200"> |
| Menu sheet | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-menu.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-menu.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-menu.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-menu.png" width="200"> |
| Key sheet | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-key.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-key.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-key.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-key.png" width="200"> |
| Pairing | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-dark-pair.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-dark-pair.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/before/ember-light-pair.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/studio-light-pair.png" width="200"> |

**The Ember themes**, now that the phone follows them (track colours too):

| | Pads | Mixer | Macros | Big transport |
|---|---|---|---|---|
| Ember Graphite | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-graphite-pads.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-graphite-mixer.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-graphite-macros.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-graphite-transport.png" width="200"> |
| Ember Paper | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-paper-pads.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-paper-mixer.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-paper-macros.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/after/ember-paper-transport.png" width="200"> |

**Follow phone**, with Ember Graphite as the dark-mode pick and Studio Light as the light-mode pick (Nota itself in Studio Dark):

| Phone dark | Phone light |
|---|---|
| <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/follow-phone/dark-menu.png" width="200"> | <img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-remote-studio/follow-phone/light-menu.png" width="200"> |

## Testing

- `scripts/build.sh`: native + managed + smoke test. Passed in full (3,397 checks) on the final C#. After a later markup-only tweak to Scenes, a rerun failed one timing budget only: *Nota Consort stress* wants the average under half of 2.9 ms and measured 1.7 ms on a loaded Mac. That's engine DSP timing this PR doesn't touch, and it's a known flaky budget. Rerun on an idle machine to confirm.
- **Smoke test, Remote session** (a real WebSocket against `RemoteServer`): the phone gets a `theme` message with the worn theme and both picks, each theme once with all its tokens and the ink pair. An unchanged theme isn't sent again. A host theme change reaches the phone exactly once, and `proj.dark` follows it.
- **Smoke test, new `-- design: remote --` section** (`DesignTokenCheck.RunRemote`), which guards the phone sources:
  - no colour literal (hex, `rgb()`, `hsl()`), `index.html` included;
  - the fallback is built from the theme files;
  - no letter-spacing, `uppercase`, `.toUpperCase()` in a component, or `.caps`;
  - radii on 0 / 1 / 2 / 3 / 4 or 50 % for dots;
  - every custom property the page reads is a theme token or one it defines, so a renamed token fails the build instead of silently painting nothing.
- After merging `feat/studio-ui` (develop 0.53.1): `scripts/build.sh` passes in full, **3,397 checks**. CI on macOS, Windows and Linux: [run 38660948020](https://github.com/chrislaupama/nota/actions/runs/38660948020) (started by hand, since CI only runs on PRs into `develop`/`main`).
- `svelte-check`: 0 errors, 0 warnings. The bundle in `Nota.Remote/wwwroot` is rebuilt.
- **Live checks** (Playwright against the stand-in, 12/12 pass):
  - Follow Nota switches across all four themes, background and `theme-color` both, in under 50 ms after the message.
  - Follow phone wears the dark pick in dark and switches to the light pick in about 20 ms when the scheme flips; a new pick from Nota arrives live; switching back to Follow Nota applies at once.
  - With the app's JS blocked, a reload still paints the stored theme.
  - An older Nota that only says "light" gets the light fallback.
  - A brand-new phone starts in Studio Dark.

Not covered here: this hasn't been tried on a physical phone against a running Nota; the stand-in and the smoke test's real socket cover the protocol. The user manual's Remote page is updated in nota-daw/nota-docs#1, which waits for this PR to land upstream.

## App icon

The app icon, tray icons and Nota Remote home-screen icon keep the dial's shape and take on the Studio Dark colours: body #1E1F22, track #141517, arc #FF9F1C, needle #F4F5F6. The PNGs were recoloured through the same colour map so the hand-tuned small sizes keep their anti-aliasing, and `nota.ico` / `nota.icns` were rebuilt from them. The Remote manifest's background and theme colour move to #1E1F22. Top row before, bottom row after:

<img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-qsdspf/icons/before-after.png" width="560">

## README

Every README screenshot and animation is re-shot in Studio (the animations are now animated WebP, about 1.5 MB instead of 17 MB of GIFs), a new Settings ▸ Appearance shot shows the four colour themes, and the badges use the Studio orange. Before on the left, after on the right:

<img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-qsdspf/readme/before-after-hero.png" width="900">

<img src="https://raw.githubusercontent.com/chrislaupama/nota/pr-assets-qsdspf/readme/before-after-light.png" width="900">
