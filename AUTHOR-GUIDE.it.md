# Guida per gli autori: dall'idea a un plugin che si può installare

Per chiunque voglia scrivere un plugin per Cuelith: programmatori esperti e persone che costruiscono con un assistente AI sapendo poco di codice. Le due strade arrivano allo stesso punto: un file `.cpkg` che supera gli stessi controlli automatici. _English version: [AUTHOR-GUIDE.md](AUTHOR-GUIDE.md)._

Superare i controlli significa che il plugin si installa, parte, resta acceso e rispetta le regole tecniche. **Non** è una garanzia di qualità né di sicurezza, e il progetto non la dà. Per il plugin risponde l'autore; per la gestione del catalogo risponde il progetto.

Le regole in breve stanno in [DEVELOPERS.md](DEVELOPERS.md). Questa guida è la versione lunga: ogni passo e ogni modo noto in cui un plugin si rompe.

---

## 1. Cosa serve

| Serve                          | Perché                                                                                  |
| ------------------------------ | --------------------------------------------------------------------------------------- |
| Node.js 24 e pnpm              | Per costruire il plugin.                                                                |
| Git                            | Per prendere il modello e l'SDK.                                                        |
| Un account GitHub              | Il codice e la release del plugin stanno nel tuo repository.                            |
| Cuelith installato             | Per provare il plugin davvero.                                                          |
| `cuelith-conformance.mjs`      | Il programma di verifica. Si scarica dall'ultima release di `cuelith-core`.             |

L'SDK arriva da npm (`@cuelith/sdk`, `@cuelith/panel`, `@cuelith/ui`, `@cuelith/protocol`): il modello li elenca già, quindi basta `pnpm install`. Prendi il modello con `git clone https://github.com/Cuelith/plugin-template mio-plugin` (o con il pulsante **Use this template** su GitHub), poi `cd mio-plugin && pnpm install`.

`pnpm conformance` scarica da solo il programma di verifica; puoi anche scaricare `cuelith-conformance.mjs` a mano dall'ultima release di `cuelith-core`.

---

## 2. La strada con un assistente AI

Non serve capire il codice. Serve dare all'assistente le regole giuste e controllare il suo lavoro con lo strumento, non a occhio.

1. Prendi il modello come sopra. Apri `mio-plugin` nel tuo assistente (Claude Code, Cursor o simili).
2. Incolla il **testo guida** (sezione 3) come primo messaggio, poi descrivi a parole semplici cosa deve fare il plugin: cosa mostra, cosa clicca l'operatore, cosa gli serve dall'esterno (internet? file? un altro programma?).
3. Chiedi all'assistente di eseguire `pnpm build` e correggere ogni errore.
4. Esegui tu i controlli: `node cuelith-conformance.mjs dist/<id>-<versione>.cpkg`. Se qualcosa è segnato FAIL, incolla all'assistente tutto il rapporto e chiedi di correggere le cause. Ripeti finché scrive PASSED.
5. Installalo in Cuelith (**Plugin → Installati → Installa da cartella…**, scegli la cartella del plugin, quella che contiene `cuelith-plugin.json`, non la sottocartella `dist`; oppure **Installa da file…** con il `.cpkg`) e prova davvero: usalo, spegnilo e riaccendilo, chiudi e riapri Cuelith.
6. Solo quando il passo 4 è superato e il passo 5 convince, pubblica (sezione 7).

Due cose che un'AI sbaglia spesso e che i controlli trovano: scrive codice che usa internet senza dichiarare `network`, e mette nel pannello uno script o un font preso da un indirizzo web. Cuelith blocca entrambe le cose durante l'uso, quindi il plugin sembra «rotto» anche se il codice è giusto.

---

## 3. Il testo guida da incollare nell'assistente

Copia tutto il riquadro (è in inglese di proposito: gli assistenti lo seguono meglio, e puoi continuare a parlargli in italiano).

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

## 4. Come gira un plugin (il modello mentale)

Capirlo spiega quasi tutte le regole.

- Cuelith è fatto di un **motore** (unica fonte di verità dello show), di **postazioni** (gli schermi che le persone usano) e di **plugin**. Il tuo plugin non gira mai dentro il motore né dentro una finestra di uscita: non può intralciare ciò che è in onda.
- Un plugin **con processo** (`runtime: node`) è avviato dal motore in un processo suo, con i soli permessi approvati dall'utente. Parla col motore con un messaggio JSON per riga su ingresso e uscita standard. L'SDK nasconde tutto questo.
- Il motore manda `plugin.activate` quando avvia il plugin e `plugin.ping` ogni 10 secondi. Ogni richiesta va risposta entro **5 secondi**, altrimenti il plugin è considerato bloccato e viene riavviato.
- Se il processo cade viene riavviato, fino a **3 volte in 60 secondi**; poi resta spento finché l'utente non lo riaccende.
- Quando l'utente spegne il plugin (o chiude Cuelith) il motore manda `plugin.deactivate` e poi ferma il processo se è ancora lì.
- Un **pannello** è una pagina web mostrata in un riquadro isolato. Può parlare solo con i comandi del proprio plugin, tramite la postazione.
- Un plugin **senza processo** (`runtime: none`) è solo dati e pannelli: lingue, impaginazioni, pannelli che usano i comandi del motore.

---

## 5. Modi noti per rompersi, e rimedio

Ogni controllo dello strumento ha un nome breve. Se fallisce, cercalo qui.

| Nome del controllo    | Cosa significa                                                          | Rimedio                                                                                      |
| --------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `package-readable`    | Il `.cpkg` non si apre o ha un percorso non sicuro.                     | Un `.cpkg` è uno zip con `cuelith-plugin.json` nella **radice**, non in una sottocartella. Usa `pnpm build`; non comprimere a mano la cartella. |
| `manifest`            | Il manifest non rispetta lo schema (o un file di testi è sbagliato).    | Leggi il messaggio: nomina il campo. Comuni: `version` errata, `id` con maiuscole, chiave di testo che non inizia con l'id del plugin, trattino in una chiave. |
| `compat-range`        | Gli intervalli di `engines` escludono il Cuelith provato.               | Allarga `engines.cuelith` (es. `>=0.3.0 <1.0.0`) in modo che includa la versione attuale.    |
| `declared-files`      | Il manifest indica un file che non è nel pacchetto, o TypeScript.       | Esegui `pnpm build` prima di impacchettare. `runtime.entry` deve essere `.js`/`.mjs`, mai `.ts`. |
| `runtime-permissions` | Runtime e permessi non coincidono.                                      | Il runtime nativo richiede `native`; gli addon nativi (file `.node`) richiedono `addons`; un plugin di soli dati non deve dichiarare permessi. |
| `package-limits`      | Il pacchetto è troppo grande o ha troppi file.                          | Impacchetta il codice in un file; lascia fuori sorgenti, test, documenti, `node_modules`. I file multimediali grandi stanno nell'archivio dell'utente. |
| `junk-files`          | Dentro ci sono `node_modules`, `.git`, `.env`, chiavi o eseguibili.     | Toglili. `scripts/package.mjs` elenca cosa entra: non aggiungere cartelle alla cieca.        |
| `secrets`             | Nel pacchetto c'è una chiave privata, un token o una chiave API.        | Toglila **e cambia il segreto**: chi ha scaricato il pacchetto ce l'ha. Mai `author.key` nel repository. |
| `panel-assets`        | Un pannello carica qualcosa da internet, ha uno script in linea o `onclick=`. | Metti script, font, immagini e stili dentro il pacchetto; usa `<script src>` e `addEventListener`. |
| `code-permissions`    | Il codice sembra usare un permesso che il manifest non ha (avviso).     | Dichiaralo se serve davvero; altrimenti togli quel codice. Il motore blocca l'azione durante l'uso. |
| `package-name`        | Il file non si chiama `<id>-<versione>.cpkg` (avviso).                  | Rinominalo.                                                                                  |
| `install`             | Il motore ha rifiutato l'installazione.                                 | Valgono le stesse regole; leggi il messaggio.                                                |
| `served-files`        | Il pannello o l'icona non vengono serviti.                              | Confronta i percorsi del manifest con i file del pacchetto.                                  |
| `activation`          | Il processo non si è attivato in tempo.                                 | Usa l'SDK. Non fare nulla di lento in `activate`. Il file deve essere un solo `.mjs` impacchettato. |
| `stability`           | Il processo si è fermato da solo nel giro di pochi secondi.             | Eseguilo e leggi l'errore. Gestisci gli errori; mai `process.exit()`.                        |
| `recovery`            | Dopo un arresto brusco non è tornato su.                                | Deve partire pulito ogni volta: niente file di blocco o stati a metà che impediscano la ripartenza. |
| `clean-stop`          | Non si è fermato quando spento (o ci ha messo molto).                   | Chiudi timer, connessioni e processi figli alla disattivazione; non ignorare `plugin.deactivate`. |

### Rotture che i controlli non vedono

Sono reali e le può prevenire solo chi scrive.

- **Un controllo rigido sui dati del motore.** Il prossimo Cuelith aggiunge un campo, il plugin lo rifiuta e diventa bianco. Leggi solo ciò che serve e ignora il resto.
- **Dipendere dall'interno dell'app.** Solo il protocollo, l'SDK e le tre librerie sono un contratto. Tutto il resto cambia senza preavviso.
- **`activate` lento.** Caricare un file grande o chiamare un servizio web prima di rispondere fa sembrare il plugin bloccato. Rispondi prima, carica dopo.
- **Memoria senza limite.** Dichiara numeri veri in `resources` e rispettali. Cuelith confronta la tua dichiarazione con ciò che misura.
- **Tenere lo stato solo in memoria.** Il processo può essere riavviato in qualsiasi momento (caduta, utente, aggiornamento). Salva ciò che conta.
- **Un plugin che richiede internet, senza comportamento offline.** Le sale spesso hanno una rete scarsa. Il plugin deve fallire con garbo e non bloccare mai lo show.
- **Cambiare il significato di un comando tra due versioni.** Altri plugin e show salvati usano i nomi dei tuoi comandi ed eventi. Aggiungi; non rinominare né riutilizzare.
- **Un permesso nuovo in un aggiornamento.** Cuelith chiede di nuovo l'approvazione all'utente. È giusto, ma alcuni utenti esiteranno: spiegalo nelle note di versione.
- **Un `id` diverso.** L'`id` è l'identità: licenze, show salvati e copie installate vi fanno riferimento. Non cambiarlo mai.

---

## 6. Lo strumento di verifica

```bash
node cuelith-conformance.mjs dist/mio.plugin-1.0.0.cpkg            # tutto
node cuelith-conformance.mjs dist/mio.plugin-1.0.0.cpkg --static   # solo lettura dei file, non parte nulla
node cuelith-conformance.mjs percorso/della/cartella               # una cartella invece di un pacchetto
node cuelith-conformance.mjs mio.cpkg --core 0.4.0                 # prova contro un'altra versione di Cuelith
node cuelith-conformance.mjs mio.cpkg --json                       # per gli script
```

Finisce con `PASSED` o `FAILED`. Il codice di uscita è 0 se non ci sono errori; gli avvisi non lo fanno fallire. Con la prova completa avvia un vero motore di Cuelith in una cartella temporanea, installa il plugin, lo accende, aspetta qualche secondo, lo arresta di colpo per vedere se risale, poi lo spegne e lo riaccende. Non tocca mai il tuo Cuelith né i tuoi dati.

Lo stesso programma gira a ogni proposta al registro, e di nuovo per ogni plugin del catalogo a ogni nuova versione di Cuelith.

---

## 7. Impacchettare e pubblicare

1. **Costruisci**: `pnpm build` crea `dist/<id>-<versione>.cpkg`. Il `.cpkg` è deterministico: lo stesso sorgente dà la stessa impronta.
2. **Controlla**: sezione 6.
3. **Release**: su GitHub crea una release con tag `v<versione>` e allega il `.cpkg`. **Non sostituire il file dopo**: il registro conserva la sua impronta e un file cambiato non supera il controllo. Per una correzione pubblica una nuova versione.
4. **Proponilo**: esegui `pnpm registry`: scrive `dist/registry/<id>.json` e `<id>.svg` con impronta e dimensione già calcolate (con `pnpm registry --add-to plugins/<id>.json` aggiungi una versione nuova a una voce esistente). Apri una pull request a [`cuelith-registry`](https://github.com/Cuelith/cuelith-registry) con quei due file in `plugins/`. Fallo solo dopo che la release su GitHub esiste, perché la voce punta al file lì dentro. I controlli automatici verificano scaricamento, impronta, che id, versione, compatibilità e permessi siano identici nel pacchetto e nella voce, e lanciano lo strumento di verifica. Se richiesto, accetta con un commento l'[accordo di contribuzione](CLA.md).
5. **Plugin a pagamento**: segui anche «Vendere un plugin» in [DEVELOPERS.it.md](DEVELOPERS.it.md): un prodotto nel tuo negozio con chiavi di licenza, la tua chiave d'autore (`pnpm keys`), pacchetti firmati (`pnpm sign`) e il modulo di proposta sul sito. Il progetto non prende commissioni.
6. **Aggiornare**: `pnpm registry --add-to plugins/<id>.json` mette la nuova versione **in cima** a `versions`. Le versioni vecchie restano elencate.

### Scegliere una licenza

Il tuo plugin può avere qualsiasi licenza, aperta o chiusa, purché usi solo l'interfaccia pubblica e non abbia copiato codice da `cuelith-core`. Se non sai quale scegliere: MIT o Apache-2.0 se vuoi che chiunque possa riusarlo; GPL-3.0 se vuoi che i miglioramenti restino aperti; una licenza proprietaria (condizioni tue) se lo vendi. Scrivi il nome in `license` nel manifest e metti un file `LICENSE` nel pacchetto. È una nota tecnica, non una consulenza legale.

---

## 8. Dopo la pubblicazione

- **Resti il proprietario.** Il plugin è tuo, il supporto è tuo e sei tu a rispondere ai tuoi utenti. Scrivi nel README dove raggiungerti.
- **Stati nel marketplace.** Attivo: normale. **Obsoleto**: il suo intervallo di versioni non include più l'ultimo Cuelith; continua a funzionare su quelli precedenti. **Non conforme**: non supera i controlli; non viene proposto finché non è sistemato. **Dormiente**: non hai risposto alle email di verifica (sotto). **Ritirato**: tolto su richiesta o per un motivo grave. In ogni caso le copie già installate e le licenze già vendute continuano a funzionare.
- **Nuove versioni di Cuelith.** A ogni rilascio di Cuelith si riprovano tutti i plugin del catalogo e i risultati sono pubblici. Se il tuo fallisce, il progetto ti scrive e, se resta rotto, può segnarlo come non conforme. Tieni `engines` onesto: allargalo solo dopo aver provato la nuova versione.
- **Email di verifica.** Circa ogni sei mesi ricevi un'email con un solo pulsante: «sì, è ancora curato». Se non rispondi ricevi altri due promemoria a due settimane l'uno dall'altro; dopo il terzo senza risposta il plugin diventa dormiente (niente nuove installazioni) e ricevi un ultimo messaggio che spiega cos'è successo. Un clic lo riporta attivo in qualsiasi momento. Tieni aggiornato l'indirizzo di contatto che hai dato: risposte e conferme vanno lì.
- **Problemi di sicurezza.** Se qualcuno segnala una vulnerabilità, correggila in fretta e pubblica una nuova versione. Vedi [SECURITY.md](SECURITY.md).
- **Andarsene.** Per ritirare un plugin apri una pull request che sposta la tua voce in `withdrawn/`, oppure scrivi al progetto. Nessuna copia già installata viene rimossa.

---

## 9. Lista di controllo prima di proporre

- [ ] `pnpm check` e `pnpm build` passano.
- [ ] `node cuelith-conformance.mjs dist/<file>.cpkg` dice PASSED.
- [ ] L'ho installato in Cuelith, spento e riacceso, e ho riavviato Cuelith.
- [ ] `version` è uguale in `cuelith-plugin.json`, in `package.json` e nel tag.
- [ ] I `permissions` sono il minimo, e ognuno è spiegato nel README.
- [ ] Nessun segreto nel pacchetto; `author.key` non è nel repository.
- [ ] Il README dice cosa fa, come avere supporto e cosa richiede.
- [ ] Il nome è «Qualcosa for Cuelith» e non nomina altri prodotti.
- [ ] Non ho copiato codice da `cuelith-core`.
