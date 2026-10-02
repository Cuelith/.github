# Security

## Reporting a problem

If you find a security problem in Cuelith, in the SDK or in an official plugin, **please do not open a public issue**.

Report it privately: in the repository it concerns, open the **Security** tab and choose **Report a vulnerability**. Only the maintainers can read what you write there.

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

## Supported versions

Fixes are released for the latest version. Updates arrive automatically and install when the user decides.

---

**In italiano.** Per segnalare un problema di sicurezza non aprire una segnalazione pubblica: nel repository interessato apri la scheda **Security** e scegli **Report a vulnerability**. Ciò che scrivi lo leggono solo i responsabili del progetto. Rispondiamo entro una settimana.
