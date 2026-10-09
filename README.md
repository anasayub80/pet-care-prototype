# Pet Care — Interactive Mobile Design Prototype

Complete 66-screen pet-care mobile UI/UX prototype and review website. **The app name is undecided.** This repository contains the current violet/lime design, all website code, original generated artwork, optimized runtime assets, animations, and design handoff documentation.

[View the private live preview](https://pet-care-design.m-anasayub80.chatgpt.site)

## Product requirements

[Complete product and engineering requirements](docs/project-requirements.md): feature specifications, acceptance criteria, all 66 screens, permissions, data model, backend integrations, offline behavior, design/motion, release phases, and open decisions. Production requirements are distinguished from the current simulated prototype.

## Run locally

Requires Node.js 18 or newer. There are no external npm dependencies and no build step.

```sh
npm start
```

Open `http://localhost:8080`. Alternatively:

```sh
python3 -m http.server 8080 --directory dist
```

## Verify the prototype

```sh
npm run check
```

The check renders every screen in a simulated document context, validates routes and local assets, and exercises care logging, reminders, currency handling, sitter dates, weight conversion, and an empty routine. It does not replace browser layout or native-app testing.

## Explore the design

- **Prototype:** click through mobile screens, submit sample forms, log care, and explore shared-care flows.
- **All screens:** inspect the full 66-screen gallery.
- **Design system:** palette, typography, UI conventions, illustration downloads, UX flows, and native implementation guidance.
- **Motion on/off:** toggle illustration motion, transitions, and completion feedback. Reduced-motion preferences are respected.

## Repository structure

```text
dist/                     Complete deployable website and frontend source
  index.html              Review studio and device frame
  app.js                  Screen inventory, base layouts, state, and interactions
  design.js               Redesigned screen compositions and motion behaviors
  style.css               Base layout and components
  design.css              Current visual direction, illustration layouts, animations
  assets/                 Six optimized WebP assets used by the website
assets/originals/         Six full-resolution original PNG assets
docs/                    Screen inventory, asset manifest, UX and implementation notes
scripts/serve.mjs         Dependency-free local static server
tests/prototype-smoke.cjs Portable rendering and interaction checks
.openai/hosting.json      Existing Sites project reference and static output configuration
```

## Prototype boundaries

Care state is in memory and resets when the page reloads. Authentication, subscriptions, emails, invitations, uploads, phone calls, offline syncing, push notifications, and live ads are simulated. Selected local files are not uploaded. Exports contain sample information.

No real veterinary records, live AdMob SDK, backend credentials, production user accounts, or payment services are included. The app should be implemented and tested natively before store release. The privacy and terms dialogs are draft outlines, not final policies.

## Deployment

Serve `dist/` as a static website on any compatible host. All runtime images, CSS, and JavaScript are local. The `.openai/hosting.json` file references the existing private preview; it does not contain credentials or provision a new GitHub deployment. This repository is a complete independent export of the current preview source.

## Assets

All six images were generated specifically for this project. Original PNGs retain their supplied resolution and alpha where applicable. Optimized WebPs are the exact versions used in the current website. See `docs/assets.md` and `docs/asset-manifest.json`.
