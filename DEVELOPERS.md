# Developer guide: building for Cuelith

How to write a plugin that keeps working as Cuelith grows, and how to publish it. _Versione italiana: [DEVELOPERS.it.md](DEVELOPERS.it.md)._

Start from [`plugin-template`](https://github.com/Cuelith/plugin-template): a complete example with a panel, a command and its own process. The full specification is in [`cuelith-docs`](https://github.com/Cuelith/cuelith-docs).

## What a plugin is

A folder with a manifest, `cuelith-plugin.json`, and whatever the manifest points to. Packaged as a `.cpkg` file (a zip with the manifest at the root).

| Part              | What it is                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| `id`              | Reverse-domain name, unique: `yourname.something`. The `cuelith.*` names are reserved for the project.  |
| `family`          | `function`, `mode`, `video`, `audio`, `control`, `integration` or `locale`.                             |
| `runtime`         | `none` (data and panels only), `node` (your code in its own process) or `native` (a native program).    |
| `permissions`     | What the plugin needs. The user sees the list before installing.                                        |
| `contributes`     | Panels, commands, item types, events, layouts, languages.                                               |
| `engines`         | Which versions of Cuelith and of the protocol the plugin works with.                                    |
| `resources`       | How much memory and processor the plugin uses, at rest and at most.                                     |
| `icon`            | One simple SVG, different from every other plugin's.                                                    |

Libraries: `@cuelith/sdk` for the plugin's process, `@cuelith/panel` for panels, `@cuelith/ui` for colours and fonts, `@cuelith/protocol` for types.

## Compatibility rules

These are the rules that keep a plugin working when Cuelith is updated. The marketplace review checks the ones that can be checked automatically.

### 1. Declare versions correctly

```json
"engines": { "cuelith": ">=0.1.0 <1.0.0", "protocol": "^1.9.0" }
```

- **Protocol** follows SemVer. Within major version 1 it only adds: new methods, new optional fields. Write `^1.x.0` with the **lowest** version that has everything you use.
- **Cuelith** is still at version 0. For 0.x versions the caret is narrow (`^0.1.0` means "0.1 only" and excludes 0.2), so write an explicit range as above.
- A plugin whose range does not match is not loaded, and the user is told why.

### 2. Ignore what you do not know

A newer Cuelith sends fields your plugin has never seen.

- **Never validate the state you receive with a strict schema.** Read the fields you need; ignore the rest.
- Treat a missing optional field as its default, and an unknown value in a list of options as "something else", not as an error.
- `@cuelith/panel` and `@cuelith/sdk` already behave this way. If you parse messages yourself, do the same.

A plugin that rejects unknown fields goes blank at the next update. This is the most common way a plugin breaks.

### 3. Use only the public contract

- Talk to Cuelith only through the protocol methods and the two libraries.
- Do not depend on the structure of the interface, on file locations inside the app, or on anything not declared in `@cuelith/protocol`. It will change without notice.
- Commands are named `area.verb`. A panel can call only its own plugin's commands.

### 4. Never get in the way of what is on air

- Your code never runs in the engine or in the output windows. You cannot draw on an output directly.
- Answer every request within **5 seconds**. Cuelith checks that your process is alive every 10 seconds.
- If your process crashes it is restarted, up to 3 times in 60 seconds; then it stays off until the user turns it on again. Keep your data safe across a restart.
- Do long work in the background and report progress; do not hold a command open.

### 5. Ask for the least

- Declare only the permissions you use. `network:<host>` instead of `network` whenever you know the host.
- Without a permission Cuelith refuses: no files outside your own folder, no other programs, no network.
- `native` means full access to the computer and the user is told so. Use it only when there is no other way (for example a hardware SDK).
- The data space of a plugin is limited to 10 MB. Large files belong in the user's media archive.

### 6. Panels

- A panel runs in an isolated frame. It has no network access unless the plugin has the permission, and it cannot load anything from the internet: bundle your fonts, scripts and images.
- `<form>` submission does not work there: use plain buttons.
- Take colours and fonts from `@cuelith/ui`, so the panel matches the app and follows its updates.
- `side` panels are tabs in the left column. `center` panels are editors and open in their own window.
- Pass the control-desk keys to the host (`host.key`) so the operator's shortcuts keep working while your panel has focus.

### 7. Texts and languages

- No text in code: only translation keys, and every key starts with your plugin id (`yourname.something.title`). Keys cannot contain hyphens.
- Ship at least one language. If the app is in a language your plugin does not have, your texts are shown in the language you do have.
- The names of song sections (Verse, Chorus, Bridge…) are fixed and never translated.

### 8. Be honest about resources

Declare in `resources` what the plugin uses at rest and at most. Cuelith shows the user whether the computer can cope and compares your declaration with what it measures.

### 9. Names

- Call a plugin "Something for Cuelith", not "Cuelith Something". See [TRADEMARK.md](TRADEMARK.md).
- Do not name other products in texts and descriptions. Open formats and protocols are fine.

## Trying it

1. `pnpm install && pnpm build` in your plugin.
2. In Cuelith: **Plugins → Installed → Install from folder…** and choose the plugin's folder.
3. Test with a show on air: turn the plugin off and on, kill its process, change language, restart the app.

## Publishing in the marketplace

1. In your repository, create a release with the package `<id>-<version>.cpkg`.
2. Open a pull request to [`cuelith-registry`](https://github.com/Cuelith/cuelith-registry) adding `plugins/<id>.json` and `plugins/<id>.svg`.
3. The checks verify that the package downloads, that its fingerprint matches, and that id, version, compatibility and **permissions** are the same in the package and in the registry.
4. Once merged, the plugin appears in every Cuelith marketplace.

Plugins from outside the Cuelith organisation are shown as "not verified", with a notice before installing. Your plugin's code is yours and under the licence you choose.

## The licence of your plugin

Cuelith itself is under the GNU GPL version 3 or later, so you might wonder whether your plugin must be too. **It does not**, as long as it works with Cuelith only through the public interface: the plugin protocol, the SDK, its panels and its data files. The [Plugin Exception](https://github.com/Cuelith/cuelith-core/blob/main/PLUGIN-EXCEPTION.md) says so explicitly. You can publish a plugin as open source under any licence, or sell it as closed source under your own terms (EULA).

- The SDK (`cuelith-sdk`) and the template are under Apache 2.0: you can include them in a plugin with any licence, keeping the Apache notices.
- **Do not copy code from `cuelith-core`** into your plugin, and do not load it into your process: that code is GPL, and the exception would no longer cover you. If you need something the protocol does not offer, ask for it (see below).
- Put your licence in the manifest (`license`) and in a `LICENSE` file in the package; for a proprietary plugin, give the name of your EULA and where to read it.
- Do not use the name "Cuelith" in the name of your plugin: say "for Cuelith" (see [TRADEMARK.md](TRADEMARK.md)).

## Selling a plugin

- **Today** the marketplace lists free plugins only. You are free to sell a plugin yourself, outside the marketplace, under your own licence: users install it from a file, and Cuelith treats it like any other plugin installed from a file.
- **Planned**: paid plugins listed in the marketplace, with your own licence terms and licences tied to the buyer's computer. How it will work is still being designed (the project does not handle payments: sales go through an external seller of your choice); dates and rules are not decided yet. Nothing in the app pretends this exists before it does.

## Changing Cuelith itself

See [CONTRIBUTING.md](CONTRIBUTING.md). If your plugin needs something the protocol does not offer, open an issue describing what you are trying to build: adding a method to the protocol is better than working around it.
