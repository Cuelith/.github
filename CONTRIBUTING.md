# Contributing to Cuelith

Thank you for wanting to help. This page explains how to propose a change to any Cuelith repository. _Versione italiana: [CONTRIBUTING.it.md](CONTRIBUTING.it.md)._

## First, choose the right road

| You want to…                                       | Do this                                                                                                  |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Add a feature for your own use case                | Write a **plugin** (plugin). You do not need our permission and you keep your code. See [DEVELOPERS.md](DEVELOPERS.md). |
| Fix a bug, improve the core, the SDK or the docs   | Propose a change to the repository, as described below.                                                  |
| Report a problem or suggest an idea                | Open an issue in the repository it concerns.                                                             |
| Report a security problem                          | Do **not** open a public issue. See [SECURITY.md](SECURITY.md).                                          |

Cuelith has a small core on purpose. A feature that only some users need belongs in a plugin, and a proposal to add it to the core will usually be declined for that reason, not because it is a bad idea.

## How a change gets in

1. **Talk first for anything big.** Open an issue before working on a new feature, a change to the protocol, or a change to how something looks or behaves. A short discussion saves you from writing code that cannot be accepted.
2. **Fork** the repository and create a branch from `dev`.
3. Make the change. Keep it small and about one thing.
4. Run the checks (below). They must pass.
5. Open a **pull request to `dev`**. Say what changes for the person using Cuelith and how you tested it.
6. The first time, a bot asks you to accept the [Contributor License Agreement](CLA.md) with one comment.
7. The automatic checks run. A maintainer reviews, asks for changes if needed, and merges.

`main` only receives tagged releases. Pull requests to `main` are closed.

## Checks

The repositories sit side by side in one folder, because they use each other during development:

```
Cuelith/
  cuelith-sdk/        protocol, SDK, panel library, UI tokens
  cuelith-core/       the app: engine, desktop, client, renderer
  plugin-locale-it/   Italian
  plugin-locale-en/   English
  plugin-songs/       Songs plugin
  plugin-template/    example plugin
```

In `cuelith-sdk` first, then in the repository you changed:

```
pnpm install
pnpm build
pnpm check          # typecheck, lint, tests
```

In `cuelith-core`, changes to the interface or the engine also need the tests that drive the real app: `pnpm e2e`.

## Rules every change follows

- **The outputs never go down.** No plugin code runs in the engine or in the output windows. Nothing may block or delay what is on air.
- **One contract.** Every method, type and event is declared once in `@cuelith/protocol`. A protocol change updates, together: the types, the engine and client, the documentation, and the version number. Protocol changes are additive within a major version.
- **Forward compatibility.** Whoever receives data from the engine checks only the shape it needs and ignores fields it does not know. Never validate received state with strict schemas.
- **No text in code.** The interface shows translation keys only. A new key is added to **both** language plugins, Italian and English, in the same pull request.
- **No fake features.** Do not add buttons, settings or fields for something that does not work yet.
- **Works offline.** No fonts, scripts or data loaded from the internet at run time.
- **Do not name other products** in code, comments, tests, texts or documentation. Open formats (OpenLyrics, ChordPro) and protocols (NDI, ASIO, Dante, MIDI, OSC, DMX…) are fine.
- **Tests come with the change.** A bug fix includes a test that fails without the fix.
- Match the style around you. `pnpm format` formats the code.

## Licence of your contribution

Cuelith is licensed under Apache 2.0. By contributing you agree to the [Contributor License Agreement](CLA.md): you keep the copyright on your work and give the project the rights it needs to distribute it. Only contribute work you wrote or have the right to submit.

## Names and logo

The licence covers the code, not the name. See [TRADEMARK.md](TRADEMARK.md) before publishing a modified version or naming a plugin.

## Conduct

Be respectful and assume good faith. See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
