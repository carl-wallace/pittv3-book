# Miscellaneous

## Checking the URIs in a certificate

Fetches every HTTP URI a certificate names — authority information access, subject information
access, CRL distribution points and freshest CRL — and reports each one on its own.

**This is a check of the repositories, not of the certificate.** It builds no path and reaches no
verdict about trust. A certificate whose URIs all answer correctly may still fail to validate, and
one that validates may name a repository that has been offline for a year.

An issuer — supplied, or auto-discovered from the AIA extension — is what makes CRL signature
verification and OCSP possible. Without one, those rows report that they could not be checked rather
than reporting a failure.

The check is available from the Tools view in the desktop application, from its own view in the
browser, and as `--check-uris` on the command line. Path validation can also run it over every
certificate on each path it builds, appending the results to that path's log; each certificate is
checked once per run.

## Exporting what a run used

*Export PKI Environment* writes the trust material a validation actually used as a `.cbor` store
pair. Two things make that worth doing: it captures an environment assembled by uploading, and it
captures certificates a run went out and fetched that were not in any store to begin with.

The result uploads like any other store, so an environment that took a network to assemble can be
carried to a machine that has none.

## Where state is kept

The desktop application keeps its state under `~/.pittv3`:

| path | holds |
|---|---|
| `settings.json` | the settings file the form edits by default |
| `log.yaml` | the log4rs configuration, written on first use |
| `logs/pittv3.log` | the rolling log the default configuration writes |
| `cas/`, `downloads/` | CA certificates, and certificates fetched by chasing |
| `crls/` | the CRL index, which indexing prunes to the time of interest |
| `errors/` | where cleanup moves certificates rather than deleting them |
| `graphs/` | cached graphs, keyed by the material they were built from |

The browser application keeps its settings in the browser's local storage, and offers the same
settings as a downloadable file so a copy can be kept or moved to another interface.
