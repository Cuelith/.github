# Guida per chi sviluppa: costruire per Cuelith

Come scrivere un plugin che continua a funzionare mentre Cuelith cresce, e come pubblicarlo. _English version: [DEVELOPERS.md](DEVELOPERS.md)._

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

## Vendere un plugin

- **Oggi** il marketplace elenca solo plugin gratuiti. Sei libero di vendere un plugin per conto tuo, fuori dal marketplace, con la tua licenza: gli utenti lo installano da file, e Cuelith lo tratta come ogni altro plugin installato da file.
- **In programma**: plugin a pagamento nel marketplace, con un account autore, un prezzo, le tue condizioni di licenza e licenze legate ai computer di chi compra. Il progetto è pubblico (decisione 0008 in `cuelith-docs`); date e ripartizione dei ricavi non sono ancora decise. Nel programma nulla finge che esista prima che esista davvero.

## Modificare Cuelith

Vedi [CONTRIBUTING.it.md](CONTRIBUTING.it.md). Se al tuo plugin serve qualcosa che il protocollo non offre, apri una segnalazione spiegando cosa vuoi costruire: aggiungere un metodo al protocollo è meglio che aggirarlo.
