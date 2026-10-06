# Author guide: from idea to a plugin people can install

For anyone who wants to write a plugin for Cuelith: experienced programmers, and people who build with an AI assistant and know little about code. Both paths end in the same place: a `.cpkg` file that passes the same automatic checks. _Versione italiana: [AUTHOR-GUIDE.it.md](AUTHOR-GUIDE.it.md)._

Passing the checks means the plugin installs, starts, stays up, and follows the technical rules. It is **not** a guarantee of quality or safety, and the project does not give one. The author answers for the plugin; the project answers for running the catalogue.

Short reference rules live in [DEVELOPERS.md](DEVELOPERS.md). This guide is the long version: every step, and every way a plugin is known to break.

---

## 1. What you need

| You need                         | Why                                                                                       |
| -------------------------------- | ----------------------------------------------------------------------------------------- |
| Node.js 24 and pnpm              | To build the plugin.                                                                      |
| Git                              | To get the template and the SDK.                                                          |
| A GitHub account                 | Your plugin's code and its release live in your own repository.                           |
| Cuelith installed                | To try the plugin for real.                                                               |
| `cuelith-conformance.mjs`        | The checking program. Download it from the latest release of `cuelith-core`.              |

The SDK comes from npm (`@cuelith/sdk`, `@cuelith/panel`, `@cuelith/ui`, `@cuelith/protocol`): the template already lists it, so `pnpm install` is enough. Get the template with `git clone https://github.com/Cuelith/plugin-template my-plugin` (or the **Use this template** button on GitHub), then `cd my-plugin && pnpm install`.

`pnpm conformance` downloads the checking program for you; you can also download `cuelith-conformance.mjs` by hand from the latest release of `cuelith-core`.

---

## 2. The path with an AI assistant

You do not need to understand the code. You need to give the assistant the right rules, and to check its work with the tool, not with your eyes.

1. Get the template as above. Open `my-plugin` in your assistant (Claude Code, Cursor, or similar).
2. Paste the **brief** (section 3) as the first message, then describe in plain words what the plugin should do. Say what it shows, what the operator clicks, what it needs from the outside world (internet? files? another program?).
3. Ask the assistant to run `pnpm build` and fix every error.
4. Run the checks yourself: `node cuelith-conformance.mjs dist/<id>-<version>.cpkg`. If anything is marked FAIL, paste the whole report back to the assistant and ask it to fix the causes. Repeat until it says PASSED.
5. Install it in Cuelith (**Plugins → Installed → Install from folder…**, choose the `dist` folder that `pnpm build` made, or the folder with the manifest) and try it for real: use it, turn it off and on, close and reopen Cuelith.
6. Only when step 4 passes and step 5 feels right, publish (section 7).

Two things an AI often gets wrong, and that the checks catch: it writes code that calls the internet without declaring `network`, and it puts a script or a font from a web address into the panel. Both are blocked at run time in Cuelith, so the plugin looks "broken" even if the code is fine.

---

## 3. The brief to paste into your assistant

Copy everything in the box.

```text
You are writing a plugin for Cuelith (live projection software). Follow these rules exactly.
If a rule conflicts with what the user asks, tell the user instead of breaking the rule.

STRUCTURE
- Start from the existing template in this folder. Keep its layout: cuelith-plugin.json, src/main.ts
  (the plugin's process), src/ui/ (panels), locales/, scripts/package.mjs.
- cuelith-plugin.json is the manifest. "id" is a reverse-domain name that is lowercase letters and
  digits, e.g. "yourname.something". Never use ids starting with "cuelith.".
- "version" is SemVer (1.2.3). Raise it on every release. The git tag, package.json version and
  manifest version must be the same.
- "engines": {"cuelith": ">=0.3.0 <1.0.0", "protocol": "^1.9.0"}. Never write ^0.x for cuelith.
  Never use "*" or an open-ended ">=" range.

PROCESS (src/main.ts)
- Use only @cuelith/sdk: export default definePlugin({ activate(ctx) { ... } }).
- The process is bundled by vite into ONE file, dist/main.mjs, with the SDK inside. It can read only
  its own folder, so it cannot load node_modules at run time. Every dependency must be bundled.
- Answer every command in under 5 seconds. Do slow work in the background and report progress.
- Do not call process.exit(), do not start servers, do not leave timers running after deactivate.
- Never throw an uncaught error. Wrap handlers; throw PluginError with a translation key for
  problems the user should see.
- Store data only with ctx.storage (max 10 MB) or in ctx.dataDir. Never write anywhere else.

PERMISSIONS (manifest "permissions")
- Declare only what the code really uses. Without a permission the engine blocks the action.
  storage | network | network:<host> | fs:read | fs:write | devices:video | devices:audio |
  devices:midi | serial | process | addons | native
- Prefer "network:api.example.com" over "network". Never ask for fs:*, process, addons or native
  unless there is no other way, and say why in the README.
- A plugin with no process (only data or panels) uses "runtime": {"type": "none"} and no permissions.

PANELS (src/ui/)
- Panels run in an isolated frame with a strict policy. Everything must come from files inside the
  package: no CDN, no Google Fonts, no external images, no inline <script>, no onclick= attributes,
  no <form> submission. Use <script src="..."> and addEventListener.
- Use @cuelith/panel to call the plugin's commands and @cuelith/ui for colours and fonts.

TEXTS
- No visible text in code. Use translation keys that start with the plugin id and contain no
  hyphens, defined in locales/<lang>.json. Ship at least one language. Every key used anywhere in
  the manifest (titles) must exist.

DATA FROM THE ENGINE
- Never validate the data you receive with a strict schema. Read the fields you need, ignore all
  others, treat missing optional fields as defaults, treat unknown option values as "other".

DO NOT
- Do not copy code from cuelith-core. Do not import anything that is not in @cuelith/sdk,
  @cuelith/panel, @cuelith/ui, @cuelith/protocol.
- Do not put secrets (API keys, tokens, private keys, .env files, author.key) in the package.
- Do not name other products in descriptions. Name the plugin "Something for Cuelith", not
  "Cuelith Something".
- Do not invent protocol methods or manifest fields. If unsure, read
  node_modules/@cuelith/protocol (the types are the truth) or ask the user.

WORKFLOW
- After every change run: pnpm check && pnpm build
- Then run: node cuelith-conformance.mjs dist/<id>-<version>.cpkg
- The work is finished only when it prints PASSED. Never say a plugin is "safe" or "certified".
```

---

## 4. How a plugin runs (the mental model)

Knowing this explains almost every rule.

- Cuelith is made of an **engine** (the one source of truth for the show), **stations** (the screens people use) and **plugins**. Your plugin never runs inside the engine or inside an output window. It cannot get in the way of what is on air.
- A plugin **with a process** (`runtime: node`) is started by the engine in its own process, with only the permissions the user approved. It talks to the engine with one JSON message per line on standard input/output. The SDK hides this.
- The engine asks `plugin.activate` when it starts your plugin and `plugin.ping` every 10 seconds. Any request must be answered within **5 seconds**, or the plugin is considered stuck and is restarted.
- If the process crashes, it is restarted, up to **3 times in 60 seconds**. After that it stays off until the user turns it on again.
- When the user turns the plugin off (or closes Cuelith) the engine sends `plugin.deactivate`, and then stops the process if it is still there.
- A **panel** is a web page shown in an isolated frame. It can only talk to its own plugin's commands, through the station.
- A plugin **without a process** (`runtime: none`) is only data and panels: languages, layouts, panels that use the engine's own commands.

---

## 5. Known ways to break, and the fix

Each check in the tool has a short name. If it fails, find it here.

| Check name            | What it means                                                         | Fix                                                                                          |
| --------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `package-readable`    | The `.cpkg` cannot be opened, or has an unsafe path.                   | A `.cpkg` is a zip with `cuelith-plugin.json` at its **root**, not inside a subfolder. Use `pnpm build`; do not zip the folder by hand. |
| `manifest`            | The manifest does not match the schema (or a translation file is wrong). | Read the message: it names the field. Common: bad `version`, `id` with capitals, a translation key that does not start with the plugin id, a hyphen in a key. |
| `compat-range`        | The `engines` ranges exclude the Cuelith being tested.                | Widen `engines.cuelith` (e.g. `>=0.3.0 <1.0.0`). Check that it includes the current release.   |
| `declared-files`      | The manifest points to a file that is not in the package, or to TypeScript. | Run `pnpm build` before packaging. `runtime.entry` must be `.js`/`.mjs`, never `.ts`.       |
| `runtime-permissions` | Runtime and permissions disagree.                                      | Native runtime needs `native`; native addons (`.node` files) need `addons`; a data-only plugin should declare no permissions. |
| `package-limits`      | The package is too big or has too many files.                          | Bundle the code; leave out source, tests, docs, `node_modules`. Large media belongs in the user's media archive. |
| `junk-files`          | `node_modules`, `.git`, `.env`, key files, or executables are inside.   | Leave them out. `scripts/package.mjs` lists exactly what goes in; do not add folders to it blindly. |
| `secrets`             | A private key, token or API key is in the package.                     | Remove it, **and change the secret**: anyone who downloaded the package has it. Never put `author.key` in the repo. |
| `panel-assets`        | A panel loads something from the internet, has an inline script, or `onclick=`. | Put scripts, fonts, images and styles inside the package; use `<script src>` and `addEventListener`. |
| `code-permissions`    | The code seems to need a permission the manifest lacks (warning).      | Declare it if it is truly needed; otherwise remove that code. The engine blocks the action at run time. |
| `package-name`        | The file is not called `<id>-<version>.cpkg` (warning).                | Rename it.                                                                                   |
| `install`             | The engine refused to install.                                         | The same rules as above; read the message.                                                   |
| `served-files`        | The panel or icon is not served.                                       | Check the paths in the manifest against the files in the package.                           |
| `activation`          | The process did not become active in time.                             | Use the SDK. Do nothing slow in `activate`. Make sure the file is a single bundled `.mjs`.     |
| `stability`           | The process stopped on its own within seconds.                         | Run it and read the error. Handle errors; never call `process.exit()`.                       |
| `recovery`            | After a sudden kill, it did not come back.                             | It must start cleanly every time: no lock files or half-written state that block a restart.   |
| `clean-stop`          | It did not stop when turned off (or took long).                        | Close timers, sockets and child processes on deactivate; do not ignore `plugin.deactivate`.    |

### Breaks the checks cannot see

These are real, and only you can prevent them.

- **A strict parser on engine data.** The next Cuelith adds a field, your plugin rejects it, and goes blank. Read only what you need and ignore the rest.
- **Depending on the app's internals.** Only the protocol, the SDK and the three libraries are a contract. Anything else changes without notice.
- **Slow `activate`.** Loading a big file or calling a web service before answering makes the plugin look stuck. Answer first; load afterwards.
- **Unbounded memory.** Declare real numbers in `resources`, and keep to them. Cuelith compares your declaration with what it measures.
- **Keeping state only in memory.** The process can be restarted at any time (crash, user, update). Save what matters.
- **A plugin that needs the internet, with no offline behaviour.** Venues often have poor connectivity. The plugin must fail softly and never block the show.
- **Changing the meaning of a command between versions.** Other plugins and saved shows use your command names and event names. Add; do not rename or repurpose.
- **A new permission in an update.** Cuelith asks the user to approve it again. That is correct, but expect some users to hesitate; explain it in the release notes.
- **A different `id`.** The `id` is the identity: licences, saved shows and installed copies refer to it. Never change it.

---

## 6. The checking tool

```bash
node cuelith-conformance.mjs dist/my.plugin-1.0.0.cpkg            # everything
node cuelith-conformance.mjs dist/my.plugin-1.0.0.cpkg --static   # only reading the files, nothing is started
node cuelith-conformance.mjs path/to/folder                       # a plugin folder instead of a package
node cuelith-conformance.mjs my.cpkg --core 0.4.0                 # test against another Cuelith version
node cuelith-conformance.mjs my.cpkg --json                       # for scripts
```

It ends with `PASSED` or `FAILED`. Exit code is 0 when there are no errors; warnings do not fail it. With the full test it starts a real Cuelith engine in a temporary folder, installs your plugin, turns it on, waits a few seconds, kills it to see that it comes back, then turns it off and on. It never touches your real Cuelith or your data.

The same program runs on every proposal to the registry, and again for every plugin in the catalogue each time a new Cuelith is released.

---

## 7. Packaging and publishing

1. **Build**: `pnpm build` makes `dist/<id>-<version>.cpkg`. The `.cpkg` is deterministic: the same source gives the same fingerprint.
2. **Check**: section 6.
3. **Release**: on GitHub, create a release tagged `v<version>` and upload the `.cpkg` as an asset. **Do not replace the file afterwards**: the registry stores its fingerprint, and a changed file fails the check. For a fix, publish a new version.
4. **Propose it**: open a pull request to [`cuelith-registry`](https://github.com/Cuelith/cuelith-registry) with `plugins/<id>.json` and `plugins/<id>.svg` (use the existing entries as a model). The automatic checks verify the download, the fingerprint, that id, version, compatibility and permissions are identical in package and entry, and run the checking tool. If you accept the [contributor agreement](CLA.md) comment when asked.
5. **Paid plugins**: also follow "Selling a plugin" in [DEVELOPERS.md](DEVELOPERS.md): a product in your own store with licence keys, your author key (`pnpm keys`), signed packages (`pnpm sign`), and the proposal form on the website. The project takes no commission.
6. **Updating**: add the new version at the **top** of `versions` in your entry, with its url, fingerprint, size and permissions. Old versions stay listed.

### Choosing a licence

Your plugin can have any licence, open or closed, as long as it only uses the public interface and you did not copy code from `cuelith-core`. If you do not know which one to pick: MIT or Apache-2.0 if you want anyone to reuse it; GPL-3.0 if you want improvements to stay open; a proprietary licence (your own terms) if you sell it. Put the name in `license` in the manifest and a `LICENSE` file in the package. This is a technical note, not legal advice.

---

## 8. After publishing

- **You stay the owner.** The plugin is yours, its support is yours, and it is you who answers to your users. Say where they can reach you in the repository's README.
- **Marketplace states.** Active: normal. **Outdated**: its version range no longer includes the newest Cuelith; it keeps working on older ones. **Non-compliant**: it fails the checks; it is not offered until fixed. **Dormant**: you did not answer the liveness emails (below). **Withdrawn**: removed on request or for a serious reason. In every case, copies already installed and licences already sold keep working.
- **New Cuelith versions.** Each release of Cuelith retests every plugin in the catalogue, and the results are public. If yours fails, the project writes to you and, if it stays broken, may mark it non-compliant. Keep `engines` honest: widen it only after you tested the new version.
- **Liveness emails.** About every six months you receive an email with one button: "yes, it is still maintained". If you do not answer, you get two more reminders two weeks apart; after the third without a reply, the plugin becomes dormant (no new installs) and you get a final message that explains what happened. One click brings it back at any time. Keep the contact address you gave current: replies and confirmations go there.
- **Security problems.** If someone reports a vulnerability, fix it quickly and publish a new version. See [SECURITY.md](SECURITY.md).
- **Leaving.** To withdraw a plugin, open a pull request that moves your entry to `withdrawn/`, or write to the project. Nobody's installed copy is removed.

---

## 9. Checklist before you propose

- [ ] `pnpm check` and `pnpm build` pass.
- [ ] `node cuelith-conformance.mjs dist/<file>.cpkg` says PASSED.
- [ ] I installed it in Cuelith, and turned it off and on, and restarted Cuelith.
- [ ] `version` is the same in `cuelith-plugin.json`, `package.json` and the tag.
- [ ] `permissions` are the minimum, and each one is explained in the README.
- [ ] No secrets in the package; `author.key` is not in the repository.
- [ ] The README says what it does, how to get support, and what it needs.
- [ ] The name is "Something for Cuelith", and no other product is named.
- [ ] I did not copy code from `cuelith-core`.
