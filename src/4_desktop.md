# Desktop application

The desktop application presents the same options as the command line as a form, organised into
views reached from the sidebar:

> Validate · Results · Settings · Check URIs · Generate · Cleanup · Inspect · Help

The order is the ordinary path through the application — validate something, read the outcome,
adjust what a run does — followed by the tools for working on trust material rather than validating
against it. The first four match the browser application, which shares this shell.

## Validate

The store selector at the top chooses a complete trust environment on its own. The panels below it
supplement that store, and they open by themselves whenever they hold anything, so material left
over from an earlier run is never hidden behind a collapsed heading.

**Trust anchors and certification authorities.** One list per kind. Each entry may be a folder, a
certificate, a bundle holding several, or a `.cbor` store; the run decides which from the path and
then from the bytes, so there is nothing to declare.

**Revocation data.** Two lists, CRLs and OCSP responses, each with its own count and *Clear*. They
are not filtered apart — an OCSP response has no settled file extension — so either list will accept
either kind and each reports what it is actually holding rather than what its label promises. What
the split buys is being able to clear one kind without disturbing the other.

**End entity certificates.** What the run will judge. The button at the foot counts them, and is
disabled until there is at least one.

Certificates can also be taken from a TLS server by naming a host: the desktop opens the connection
itself and keeps the certificates it was presented, including any stapled OCSP response.

## Results

The report, the run log, and four exports: the structured report as JSON, the log as text, every
path's manifest as one text file, and the certificates and revocation data behind every path as a
zip. The name field beside them names the archive and the folder inside it; every artifact of one
run is stamped with the moment that run began, so a pair of saves belongs together by name.

*Clear* discards the results and the log, and is disabled when there is nothing to discard.

## Settings

The tabbed form edits a settings file — the same JSON the command line takes with `-s`. Fields
marked *default* are not present in the file and use certval's defaults; editing one records an
override.

*Save*, *Revert to Saved* and *Reset to defaults* act on the settings themselves. The **Settings
file** box below the form names which file is being edited and offers the actions that change that:
choosing another, returning to the default, or deleting it.

The *Folders & files* tab also carries the folders a run writes to and the actions that maintain
them — cleaning up downloaded certificates, emptying the CRL index, discarding cached graphs.

## Generate

Writes a trust store from a folder of trust anchors and CA certificates. *Chase SIA and AIA* extends
that by following the URIs the certificates name; *CBOR TA store* writes a trust-anchor store
instead of a CA store. Both are consulted only while generating, so they are unavailable when
*Generate* is unchecked.

**Mozilla CSV** sits above the CA folder because it fills it: it parses the CCADB intermediate-CA
report into certificates written to that folder, which generation then builds a store from. The two
rows name the same folder for that reason — one produces the material, the next consumes it.

One thing worth knowing about the output row: it is a single field whose meaning follows the *CBOR
TA store* checkbox. Turning that on relabels the row and keeps the path, so a filename chosen for a
CA store becomes the destination for a trust-anchor store. That mirrors the command line, where
`--generate` writes to `--cbor` whichever kind of store it is making. The hint under the form always
names the file that will actually be written.

## Cleanup

Removes certificates a run could not use — unparseable, not valid at the time of interest,
self-signed, or not a CA. Where an error folder is named, and one is by default, they are **moved
there rather than deleted**.

The time of interest is the criterion, so it decides what goes. A wrong value here does not produce
a wrong answer to re-run; it changes the folder.

## Inspect

A `.cbor` store is a format only PITTv3 reads — there is no text editor or `openssl` command that
will show you what is in one. This view is that viewer.

It reports what a store holds without validating anything: its partial paths, the certificates it
holds, the name constraints in force, the trust anchors loaded, and the AIA and SIA URIs present.
The fields below the checkboxes ask about one certificate or one CA, and each runs on its own when
filled in — *partial paths for target* being the one to reach for when a certificate would not
validate and you want to know what the store could have built.

## Check URIs

Fetches every HTTP URI one certificate names and reports each on its own, independently of path
processing. See [Checking the URIs in a certificate](7_miscellaneous.md) for what it does and does
not tell you.
