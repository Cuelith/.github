# Contribuire a Cuelith

Grazie per voler dare una mano. Questa pagina spiega come proporre una modifica a un repository di Cuelith. _English version: [CONTRIBUTING.md](CONTRIBUTING.md)._

## Prima, scegli la strada giusta

| Vuoi…                                                  | Fai così                                                                                                             |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| Aggiungere una funzione per il tuo caso                | Scrivi un **plugin**. Non serve il nostro permesso e il codice resta tuo. Vedi [DEVELOPERS.it.md](DEVELOPERS.it.md). |
| Correggere un difetto, migliorare nucleo, SDK o documenti | Proponi una modifica al repository, come descritto qui sotto.                                                     |
| Segnalare un problema o proporre un'idea               | Apri una segnalazione (issue) nel repository che riguarda.                                                           |
| Segnalare un problema di sicurezza                     | **Non** aprire una segnalazione pubblica. Vedi [SECURITY.md](SECURITY.md).                                           |

Cuelith ha un nucleo piccolo per scelta. Una funzione che serve solo ad alcuni sta in un plugin: una proposta di metterla nel nucleo di solito viene rifiutata per questo, non perché l'idea sia sbagliata.

## Come entra una modifica

1. **Per le cose grandi, prima parlane.** Apri una segnalazione prima di lavorare a una funzione nuova, a una modifica del protocollo o a un cambio di aspetto o di comportamento. Due righe di discussione evitano di scrivere codice che non può essere accettato.
2. Fai un **fork** del repository e crea un ramo a partire da `dev`.
3. Fai la modifica. Piccola, e su una cosa sola.
4. Esegui i controlli (sotto). Devono passare.
5. Apri una **pull request verso `dev`**. Scrivi cosa cambia per chi usa Cuelith e come l'hai provata.
6. La prima volta un automatismo ti chiede di accettare l'[accordo di contribuzione](CLA.md) con un commento.
7. Partono i controlli automatici. Un responsabile legge, chiede correzioni se servono, e accetta.

`main` riceve solo versioni con tag. Le pull request verso `main` vengono chiuse.

## Controlli

I repository stanno affiancati in una cartella, perché in sviluppo si usano a vicenda:

```
Cuelith/
  cuelith-sdk/        protocollo, SDK, libreria dei pannelli, stile comune
  cuelith-core/       il programma: motore, desktop, postazione, uscite
  plugin-locale-it/   italiano
  plugin-locale-en/   inglese
  plugin-songs/       plugin Canti
  plugin-template/    plugin d'esempio
```

Prima in `cuelith-sdk`, poi nel repository che hai modificato:

```
pnpm install
pnpm build
pnpm check          # tipi, lint, prove
```

In `cuelith-core`, per le modifiche all'interfaccia o al motore servono anche le prove che guidano il programma vero: `pnpm e2e`.

## Regole che ogni modifica rispetta

- **Le uscite non cadono mai.** Nessun codice dei plugin gira nel motore o nelle finestre di uscita. Nulla può bloccare o ritardare ciò che è in onda.
- **Un solo contratto.** Ogni metodo, tipo ed evento si dichiara una volta sola in `@cuelith/protocol`. Una modifica al protocollo aggiorna insieme: i tipi, motore e postazione, la documentazione, il numero di versione. Dentro una versione maggiore il protocollo può solo aggiungere.
- **Compatibilità in avanti.** Chi riceve dati dal motore controlla solo la forma che gli serve e ignora i campi che non conosce. Mai validare con schemi rigidi lo stato ricevuto.
- **Nessun testo nel codice.** L'interfaccia mostra solo chiavi di traduzione. Una chiave nuova va aggiunta in **entrambe** le lingue, italiano e inglese, nella stessa pull request.
- **Niente funzioni finte.** Niente pulsanti, impostazioni o campi per qualcosa che ancora non funziona.
- **Funziona senza internet.** Niente font, script o dati caricati dalla rete mentre il programma gira.
- **Non nominare altri prodotti** in codice, commenti, prove, testi o documentazione. Formati aperti (OpenLyrics, ChordPro) e protocolli (NDI, ASIO, Dante, MIDI, OSC, DMX…) vanno bene.
- **Le prove arrivano con la modifica.** Una correzione include una prova che senza la correzione fallisce.
- Segui lo stile che trovi intorno. `pnpm format` formatta il codice.

## Licenza del tuo contributo

Il tuo contributo è pubblicato con la licenza del repository a cui contribuisci: GPL 3.0 o successiva per il nucleo (`cuelith-core`) e i plugin ufficiali, Apache 2.0 per SDK, modello di plugin, registry, documentazione e sito. Ogni repository lo dice nel suo file `LICENSE`. Contribuendo accetti l'[accordo di contribuzione](CLA.md): il diritto d'autore sul tuo lavoro resta tuo, lo offri con quella licenza e dai al titolare del progetto il diritto di cambiarla in futuro. Contribuisci solo con lavoro che hai scritto tu o che hai il diritto di proporre.

## Nome e logo

La licenza copre il codice, non il nome. Leggi [TRADEMARK.md](TRADEMARK.md) prima di pubblicare una versione modificata o di dare un nome a un plugin.

## Comportamento

Rispetto e buona fede. Vedi [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
