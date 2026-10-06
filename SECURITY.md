# Security

## Reporting a problem

If you find a security problem in Cuelith, in the SDK or in an official plugin, **please do not open a public issue**.

Report it privately: in the repository it concerns, open the **Security** tab and choose **Report a vulnerability**. Only the maintainers can read what you write there. If you do not use GitHub, write to contact@lzrhive.it with “Security” at the start of the subject.

Please include:

- what an attacker could do, and what they need to do it;
- the version of Cuelith and the operating system;
- the steps to reproduce it, or a small example.

You will get an answer within a week. We will tell you when the fix is released and, if you wish, credit you in the release notes.

## What counts

- A plugin doing something its declared permissions do not allow.
- A station on the network doing something its role does not allow, or getting in without pairing.
- A package or an update being accepted without the expected verification.
- Anything that lets someone read or change files, shows or settings without the user's consent.

Known and documented limits are not vulnerabilities, for example: the connection of network stations is not encrypted (use a network you trust); a plugin with the `native` permission has full access to the computer, and the user is told so before installing.

## A problem with a third-party plugin

Plugins in the marketplace are written by third parties, who answer for them. If you find malware, a copyright infringement or another problem in one, write to contact@lzrhive.it. We tell the author, who can reply, and then decide with a reasoned message; for malware or a serious breach the plugin is removed from the catalogue at once. The rules are in the [conditions for publishing](https://cuelith.lzrhive.it/en/marketplace/terms/).

## Supported versions

Fixes are released for the latest version. Updates arrive automatically and install when the user decides.

---

**In italiano.** Per segnalare un problema di sicurezza non aprire una segnalazione pubblica: nel repository interessato apri la scheda **Security** e scegli **Report a vulnerability**. Ciò che scrivi lo leggono solo i responsabili del progetto. Rispondiamo entro una settimana. Se non usi GitHub, scrivi a contact@lzrhive.it con «Security» all'inizio dell'oggetto. Per un problema in un plugin di terzi (malware, violazione di copyright o altro) scrivi allo stesso indirizzo: avvisiamo l'autore e decidiamo con una comunicazione motivata.
