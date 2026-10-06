# Guida per chi sviluppa: costruire per Cuelith

Come scrivere un plugin che continua a funzionare mentre Cuelith cresce, e come pubblicarlo. _English version: [DEVELOPERS.md](DEVELOPERS.md)._

Sei alle prime armi o costruisci con un assistente AI? Leggi prima la [Guida per gli autori](AUTHOR-GUIDE.it.md): ogni passo, un testo pronto per il tuo assistente e ogni modo noto in cui un plugin si rompe.

Si parte da [`plugin-template`](https://github.com/Cuelith/plugin-template): un esempio completo con un pannello, un comando e il suo processo. La specifica completa è in [`cuelith-docs`](https://github.com/Cuelith/cuelith-docs).

## Cos'è un plugin

Una cartella con un manifest, `cuelith-plugin.json`, e ciò a cui il manifest rimanda. Si distribuisce come file `.cpkg` (uno zip con il manifest nella radice).

| Parte         | Cos'è                                                                                                        |
| ------------- | ------------------------------------------------------------------------------------------------------------ |
| `id`          | Nome a dominio inverso, unico: `tuonome.qualcosa`. I nomi `cuelith.*` sono riservati al progetto.            |
| `family`      | `function`, `mode`, `video`, `audio`, `control`, `integration` oppure `locale`.                              |
| `runtime`     | `none` (solo dati e pannelli), `node` (il tuo codice in un processo suo) oppure `native` (programma nativo). |
| `permissions` | Ciò che serve al plugin. L'utente vede l'elenco prima di installare.                                         |
| `contributes` | Pannelli, comandi, tipi di elemento, eventi, disposizioni, lingue.                                           |
| `engines`     | Con quali versioni di Cuelith e del protocollo funziona il plugin.                                           |
| `resources`   | Quanta memoria e quanto processore usa il plugin, a riposo e al massimo.                                     |
| `icon`        | Un SVG semplice, diverso da quello di ogni altro plugin.                                                     |

Librerie: `@cuelith/sdk` per il processo del plugin, `@cuelith/panel` per i pannelli, `@cuelith/ui` per colori e font, `@cuelith/protocol` per i tipi.

## Regole di compatibilità

Sono le regole che tengono in piedi un plugin quando Cuelith si aggiorna. La revisione del marketplace controlla quelle che si possono controllare in automatico.

### 1. Dichiara bene le versioni

```json
"engines": { "cuelith": ">=0.1.0 <1.0.0", "protocol": "^1.9.0" }
```

- Il **protocollo** segue SemVer. Dentro la versione maggiore 1 può solo aggiungere: metodi nuovi, campi facoltativi nuovi. Scrivi `^1.x.0` con la versione **più bassa** che ha tutto ciò che usi.
- **Cuelith** è ancora alla versione 0. Per le versioni 0.x l'accento circonflesso è stretto (`^0.1.0` vuol dire «solo la 0.1» ed esclude la 0.2): scrivi un intervallo esplicito come sopra.
- Un plugin con un intervallo che non corrisponde non viene caricato, e all'utente viene detto perché.

### 2. Ignora ciò che non conosci

Un Cuelith più nuovo manda campi che il tuo plugin non ha mai visto.

- **Mai validare con uno schema rigido lo stato che ricevi.** Leggi i campi che ti servono; ignora gli altri.
- Un campo facoltativo che manca vale il suo valore predefinito; un valore sconosciuto in un elenco di opzioni vale «qualcos'altro», non un errore.
- `@cuelith/panel` e `@cuelith/sdk` si comportano già così. Se leggi i messaggi da te, fai lo stesso.

Un plugin che rifiuta i campi sconosciuti resta vuoto al primo aggiornamento. È il modo più comune in cui un plugin si rompe.

### 3. Usa solo il contratto pubblico

- Parla con Cuelith solo attraverso i metodi del protocollo e le due librerie.
- Non dipendere dalla struttura dell'interfaccia, dalla posizione dei file dentro il programma o da qualsiasi cosa non dichiarata in `@cuelith/protocol`: cambierà senza preavviso.
- I comandi si chiamano `area.verbo`. Un pannello può chiamare solo i comandi del proprio plugin.

### 4. Mai intralciare ciò che è in onda

- Il tuo codice non gira mai nel motore né nelle finestre di uscita. Non puoi disegnare direttamente su un'uscita.
- Rispondi a ogni richiesta entro **5 secondi**. Cuelith controlla ogni 10 secondi che il processo sia vivo.
- Se il processo cade viene fatto ripartire, fino a 3 volte in 60 secondi; poi resta spento finché l'utente non lo riaccende. I tuoi dati devono reggere un riavvio.
- Il lavoro lungo si fa in background, dicendo a che punto è; non tenere aperto un comando.

### 5. Chiedi il minimo

- Dichiara solo i permessi che usi. `network:<host>` invece di `network` ogni volta che conosci l'host.
- Senza un permesso Cuelith rifiuta: niente file fuori dalla tua cartella, niente altri programmi, niente rete.
- `native` vuol dire accesso completo al computer, e all'utente viene detto. Usalo solo quando non c'è altra strada (per esempio l'SDK di un apparecchio).
- Lo spazio dati di un plugin è limitato a 10 MB. I file grandi stanno nell'archivio media dell'utente.

### 6. Pannelli

- Un pannello gira in un riquadro isolato. Non ha accesso alla rete se il plugin non ha il permesso, e non può caricare nulla da internet: font, script e immagini vanno dentro il pacchetto.
- L'invio dei `<form>` lì non funziona: usa pulsanti normali.
- Prendi colori e font da `@cuelith/ui`, così il pannello è coerente col programma e ne segue gli aggiornamenti.
- I pannelli `side` sono schede nella colonna di sinistra. I pannelli `center` sono editor e si aprono in una finestra propria.
- Passa alla postazione i tasti della regia (`host.key`), così le scorciatoie dell'operatore funzionano anche col tuo pannello in primo piano.

### 7. Testi e lingue

- Nessun testo nel codice: solo chiavi di traduzione, e ogni chiave inizia con l'id del tuo plugin (`tuonome.qualcosa.titolo`). Le chiavi non ammettono il trattino.
- Fornisci almeno una lingua. Se il programma è in una lingua che il tuo plugin non ha, i tuoi testi compaiono nella lingua che hai.
- I nomi delle sezioni dei canti (Verse, Chorus, Bridge…) sono fissi e non si traducono.

### 8. Sii sincero sulle risorse

Dichiara in `resources` quanto usa il plugin a riposo e al massimo. Cuelith mostra all'utente se il computer regge e confronta la tua dichiarazione con ciò che misura.

### 9. Nomi

- Un plugin si chiama «Qualcosa per Cuelith», non «Cuelith Qualcosa». Vedi [TRADEMARK.md](TRADEMARK.md).
- Non nominare altri prodotti in testi e descrizioni. Formati aperti e protocolli vanno bene.

## Provarlo

1. `pnpm install && pnpm build` nel tuo plugin.
2. In Cuelith: **Plugin → Installati → Installa da cartella…** e scegli la cartella del plugin.
3. Prova con uno show in onda: spegni e riaccendi il plugin, termina il suo processo, cambia lingua, riavvia il programma.

## Pubblicarlo nel marketplace

1. Nel tuo repository crea una release con il pacchetto `<id>-<versione>.cpkg`.
2. Apri una pull request a [`cuelith-registry`](https://github.com/Cuelith/cuelith-registry) aggiungendo `plugins/<id>.json` e `plugins/<id>.svg`.
3. I controlli verificano che il pacchetto si scarichi, che l'impronta corrisponda, e che id, versione, compatibilità e **permessi** siano gli stessi nel pacchetto e nel registry.
4. Accettata la proposta, il plugin compare nel marketplace di ogni Cuelith.

I plugin che non vengono dall'organizzazione Cuelith compaiono come «non verificati», con un avviso prima dell'installazione. Il codice del tuo plugin è tuo, con la licenza che scegli tu.

## La licenza del tuo plugin

Cuelith è sotto GNU GPL versione 3 o successiva, e ti potresti chiedere se anche il tuo plugin debba esserlo. **No**, finché dialoga con Cuelith solo tramite l'interfaccia pubblica: il protocollo dei plugin, l'SDK, i suoi pannelli e i suoi file di dati. Lo dice in modo esplicito l'[eccezione per i plugin](https://github.com/Cuelith/cuelith-core/blob/main/PLUGIN-EXCEPTION.md). Puoi pubblicare un plugin open source con la licenza che vuoi, oppure venderlo a sorgente chiuso con le tue condizioni (EULA).

- L'SDK (`cuelith-sdk`) e il modello di plugin sono sotto Apache 2.0: puoi includerli in un plugin con qualsiasi licenza, mantenendo gli avvisi Apache.
- **Non copiare codice di `cuelith-core`** nel tuo plugin e non caricarlo nel tuo processo: quel codice è GPL e l'eccezione non ti coprirebbe più. Se ti serve qualcosa che il protocollo non offre, chiedilo (vedi sotto).
- Scrivi la tua licenza nel manifest (`license`) e in un file `LICENSE` nel pacchetto; per un plugin proprietario indica il nome del tuo EULA e dove leggerlo.
- Non usare il nome «Cuelith» nel nome del tuo plugin: scrivi «per Cuelith» (vedi [TRADEMARK.md](TRADEMARK.md)).

## Vendere un plugin

Il marketplace di Cuelith elenca i plugin a pagamento ma non vende nulla: la vendita passa dal tuo negozio presso un rivenditore registrato (oggi Lemon Squeezy, che incassa, versa l'IVA e gestisce i rimborsi). Il progetto non tocca il tuo denaro e non ci sono account.

1. Crea il prodotto nel tuo negozio con **chiavi di licenza**, **3 dispositivi per chiave** e una chiave **senza scadenza** se vendi una volta sola (in Cuelith il permesso non supera mai la scadenza della chiave).
2. Indica l'indirizzo del prodotto nel tuo negozio: diventa il pulsante «Acquista» nel catalogo e nel programma. **Il progetto non chiede commissioni** e pubblicare è gratuito.
3. Crea la tua chiave d'autore e firma ogni pacchetto: nel modello, `pnpm keys` e `pnpm sign`.
4. Proponi il plugin dal modulo su <https://cuelith.lzrhive.it/marketplace/submit/>. Dopo un controllo, viene pubblicato in automatico.

I rimborsi li gestisci tu dal negozio: un rimborso disattiva la licenza. Se il computer di chi ha comprato si rompe, libera tu il posto dal negozio (Lemon Squeezy: License keys → la chiave → attivazioni). Durante una diretta non si ferma mai nulla: una licenza persa ha effetto a diretta finita.

Puoi vendere anche altrove: non c'è nessuna esclusiva. Leggi le [condizioni complete](https://cuelith.lzrhive.it/marketplace/condizioni/). In breve, un plugin nel marketplace:

- rispetta i diritti (copyright, marchi) e non copia codice del nucleo di Cuelith, che è GPL;
- non contiene malware, codice nascosto né raccolta di dati non dichiarata, e dichiara solo i permessi che usa;
- segue le regole tecniche di questa guida e supera i controlli di compatibilità, senza interferire con ciò che è in onda;
- mostra il prezzo vero e dove si compra, e dice dove gli utenti ricevono supporto (il supporto è dell'autore);
- corregge le vulnerabilità appena le conosce.

Il marketplace può controllare ogni tanto che il pacchetto sia ancora raggiungibile e che ci sia ancora l'autore, con una email e una conferma con un clic. Se non rispondi dopo tre richiami, il plugin può diventare «dormiente»: non si può più installare né acquistare dal catalogo, chi lo ha lo continua a usare, e ricevi un ultimo messaggio con come risolvere. Se un plugin esce dal catalogo, chi l'ha già comprato non perde la licenza.

### Controllare la licenza dentro il tuo plugin (protocollo 1.15)

Cuelith è GPL: chi lo modifica può togliere i suoi controlli. Per questo un plugin a pagamento controlla la licenza **da solo**, all'avvio (mai a metà di una diretta):

```ts
import { definePlugin } from "@cuelith/sdk";

export default definePlugin({
  async activate(ctx) {
    const licenza = await ctx.license.verify();
    if (!licenza.valid) {
      ctx.log.warn(`Nessuna licenza valida (${licenza.reason}): il plugin resta fermo`);
      return;
    }
    // licenza.expires, licenza.renewing (l'app la sta rinnovando in secondo piano)
  },
});
```

`verify()` chiede a Cuelith un permesso firmato e la firma del computer su una sfida casuale, e controlla entrambi con le chiavi pubbliche del progetto. Un permesso copiato da un altro computer, una risposta registrata o un Cuelith modificato che «dice di sì» non passano. Motivi di `valid: false`: `none` (nessuna licenza valida su questo computer: mai attivata, scaduta o revocata), `invalid` (la prova non regge), `unavailable` (Cuelith non può rispondere). Cosa fare in ogni caso lo decidi tu; l'app rifiuta già di installare o avviare un plugin del marketplace senza licenza valida.

## Modificare Cuelith

Vedi [CONTRIBUTING.it.md](CONTRIBUTING.it.md). Se al tuo plugin serve qualcosa che il protocollo non offre, apri una segnalazione spiegando cosa vuoi costruire: aggiungere un metodo al protocollo è meglio che aggirarlo.
