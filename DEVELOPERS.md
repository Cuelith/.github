# Developer guide: building for Cuelith

How to write a plugin that keeps working as Cuelith grows, and how to publish it. _Versione italiana: [DEVELOPERS.it.md](DEVELOPERS.it.md)._

New here, or building with an AI assistant? Read the [Author guide](AUTHOR-GUIDE.md) first: every step, a ready-made brief for your assistant, and every known way to break.

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

### 10. Settings (protocol 1.18)

Do not build a settings panel: declare the settings in `contributes.settings` and Cuelith draws one window for all plugins, in the user's language, saves every change on its own and remembers the values. The user finds it with the gear on your plugin's tab, in the **Plugins** window and by searching (Ctrl+K).

```json
"settings": [
  { "key": "size", "title": "yourname.s.size", "description": "yourname.s.size.d", "type": "number", "default": 40, "min": 10, "max": 100 },
  { "key": "mode", "title": "yourname.s.mode", "type": "string", "default": "a",
    "choices": [ { "value": "a", "title": "yourname.s.mode.a" }, { "value": "b", "title": "yourname.s.mode.b" } ] }
]
```

- Types are `string`, `number` and `boolean`. `min`/`max` apply to numbers, `choices` to strings and numbers (not booleans). The manifest is rejected if a default is outside the limits or the choices.
- `title`, `description` and the choice titles are translation keys of your plugin, as everywhere else.
- You receive the values in `ctx.settings` when your plugin starts. When the user changes one, Cuelith emits `core.plugin.settingsChanged` with `{ pluginId, settings }` (all current values): subscribe with `ctx.events.on` and ignore events for other plugins. Never assume a value changes only at start-up.
- Cuelith checks every value against your manifest before saving it: you never receive a value of the wrong type or outside the limits. A saved value that stops being valid after an update is ignored and the default returns.

### 11. Cover image and how-to guide (protocol 1.19)

The marketplace is a shop window: before installing, people see your plugin's picture and how to use it. Both come from the package itself, so there is nothing extra to fill in when you submit it.

- **Cover image**: put a PNG, JPEG or WebP in the package (at most **150 KB**, 16:9 looks best) and point to it with `"image": "media/cover.png"` in the manifest. Cuelith checks that it really is that kind of image. It shows on the marketplace card, in the welcome window and on your plugin's page on the website.
- **How-to guide**: it is your first-use guide, the `onboarding` steps (up to 8). Cuelith reads them together with your language files and shows them as **How to use it** before installation. A language is shown only if every step has its title and text in that language (title up to 80 characters, text up to 600); in the others people see the Italian or English one.
- Keep the image in later versions too: the registry compares it with the latest package.

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

Cuelith's marketplace lists paid plugins but sells nothing: the sale goes through your own store at a registered reseller (today Lemon Squeezy, which collects the payment, pays the VAT and handles refunds). The project never touches your money and there are no accounts.

1. Create the product in your store with **licence keys**, **3 devices per key**, and a key that **never expires** if you sell once (a permit in Cuelith never outlasts its key).
2. Give the address of the product in your store: it becomes the “Buy” button in the catalogue and in the program. **The project charges no commission** and publishing is free.
3. Create your author key and sign each package: in the template, `pnpm keys` and `pnpm sign`.
4. Propose the plugin from the form at <https://cuelith.lzrhive.it/en/marketplace/submit/>. After a check, it is published automatically.

Refunds are yours to handle in your store: a refund switches the licence off. If a buyer's computer breaks, free the seat from your store (Lemon Squeezy: License keys → the key → activations). Nothing is ever stopped during a live show: a licence that is lost takes effect when the show is over.

You can sell elsewhere too: there is no exclusivity. Read the full [conditions for publishing](https://cuelith.lzrhive.it/en/marketplace/terms/). In short, a plugin in the marketplace:

- respects rights (copyright, trademarks) and does not copy code from the Cuelith core, which is GPL;
- contains no malware, hidden code or undeclared data collection, and declares only the permissions it uses;
- follows the technical rules in this guide and passes the compatibility checks, without getting in the way of what is on air;
- shows the real price and where to buy, and says where users get support (support belongs to the author);
- fixes vulnerabilities as soon as it knows of them.

The marketplace may check now and then that the package is still reachable and that you are still around, by email with a one-click confirmation. If you do not reply after three reminders, the plugin may become “dormant”: it can no longer be installed or bought from the catalogue, people who already have it keep using it, and you get a final message with how to fix it. If a plugin leaves the catalogue, people who already bought it keep their licence.

### Checking the licence inside your plugin (protocol 1.15)

Cuelith is GPL: whoever modifies it can remove its own checks. So a paid plugin checks its licence **by itself**, at start-up (never in the middle of a show):

```ts
import { definePlugin } from "@cuelith/sdk";

export default definePlugin({
  async activate(ctx) {
    const licence = await ctx.license.verify();
    if (!licence.valid) {
      ctx.log.warn(`No valid licence (${licence.reason}): the plugin stays idle`);
      return;
    }
    // licence.expires, licence.renewing (the app is renewing it in the background)
  },
});
```

`verify()` asks Cuelith for a signed permit and for the computer's signature on a random challenge, and checks both with the project's public keys. A permit copied from another computer, a recorded answer or a modified Cuelith that "says yes" do not pass. Reasons for `valid: false`: `none` (no valid licence on this computer: never activated, expired or revoked), `invalid` (the proof does not hold), `unavailable` (Cuelith cannot answer). What to do in each case is your decision; the app already refuses to install or start a marketplace plugin without a valid licence.

## Formatted words in slide text (protocol 1.21)

A slide's `text` field can carry **formatted words** next to the plain text: the text stays an ordinary string, and an optional list of ranges says how each piece looks.

```ts
{
  kind: "text",
  value: "The Lord is my shepherd",
  spans: [{ start: 4, end: 8, size: 1.5, bold: true, color: "#FFD166" }],
}
```

- `start` / `end` are positions in `value` (UTF-16 units, like `String.length`), `end` excluded.
- `size` is a multiple of the style's size (0.5 to 3); `bold`, `italic` and `color` (`#RRGGBB`) are optional. From protocol 1.24 a range can also have an `outline` (`{ width, color }`) and a `shadow` (`{ offset, blur, color }`), in pixels on an output 1080 high like the text styles. At most 300 ranges, all inside the text.
- Anything that does not know about `spans` reads `value` and nothing breaks: that is the **plain text** export. The **customised text** export is `value` plus `spans` as they are.

`@cuelith/protocol` has the helpers: `segmentsOf(value, spans)` splits the text into pieces with their style (the starting point for writing HTML, ChordPro or any other format), `plainText`, `styleRange` (apply a style to a selection), `shiftSpans` (keep the formatting in place after the text is edited), `sliceRich` and `joinRich` (cut and join texts). The outputs and the preview draw it the same way, never break a line, and the "fit if it does not fit" option shrinks everything together. The stage monitor shows the text without formatting.

### Formatting in plugin editors (protocol 1.22)

If your plugin has a text editor you do not need to build a bar: formatting is a **companion plugin** (Formatting), and anyone who does not install it sees nothing. You connect your text with `@cuelith/panel`:

```ts
import { bindRichText } from "@cuelith/panel";

const binding = bindRichText(panel, "slide-12", (spans) => {
  // The companion plugin asked for a change: adopt the new ranges.
  model.spans = spans;
});
// On every change of the text, the selection or the formatted words:
binding.update({ text, spans: model.spans, selection: { start, end }, font: "lora" });
// When the editor closes:
binding.end();
```

For a plain text box there is `bindTextarea(panel, area, field, { getSpans, setSpans })`, which follows text and selection by itself and moves the formatted words while you type. Only you write the text; the engine keeps the conversation in `live.richText` (it is not part of the show) with `richtext.session`, `richtext.end` and `richtext.apply`.

A companion plugin (such as Formatting) declares a panel with `placement: "editor"` (protocol 1.23): the station shows it as a toolbar **on top of the editor windows**, only while the editor has a text that uses it. It has no icon and is not a tab.

## Changing Cuelith itself

See [CONTRIBUTING.md](CONTRIBUTING.md). If your plugin needs something the protocol does not offer, open an issue describing what you are trying to build: adding a method to the protocol is better than working around it.
