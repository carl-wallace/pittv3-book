# Desktop application

The desktop application presents the same options as the command line as a form, organised into
views reached from the sidebar:

> Validate · Results · Settings · Check URIs · Generate · Inspect · Help

The order is the ordinary path through the application — validate something, read the outcome,
adjust what a run does — followed by the tools for working on trust material rather than validating
against it. The first four match the browser application, which shares this shell.

## Validate

The store selector at the top chooses a complete trust environment on its own. The panels below it
supplement that store, and they open by themselves whenever they hold anything, so material left
over from an earlier run is never hidden behind a collapsed heading.

The line under the selector says whose PKI the chosen store holds, whether it carries intermediate
CAs as well as anchors, and how current its material is — up to two dates, *published* and
*collected*. They are not the same question. *Published* is the date the source itself gives: a DoD
InstallRoot stream is timestamped when it is signed, and the version PITTv3 ships can be
considerably older than the day it was downloaded. *Collected* is when the material was taken from
that source, which bounds what it can possibly know: anything the publisher has done since is not
in these bytes. A store shows only the dates it can state honestly, and the Windows certificate
stores show none, being read live rather than shipped.

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

Builds a trust store out of the material you give it. The *Inputs* group is two pools, trust anchors
and CA certificates, and an entry in either may be a folder, a certificate, a bundle holding
several, or an existing `.cbor` store; the run decides which from the path and then from the bytes.
An existing store is how you start from a store — there is no way to reach for one the application
ships, because what goes into a store you are about to hand out should be material you chose.

*Chase SIA and AIA* extends the pool by following the URIs the certificates name, adding what they
serve until a pass adds nothing new. What it fetches is written to the download folder, which is why
that row sits beside it. The time of interest decides which certificates reach the store: one
outside it is left out, exactly as it is when the command line reads a folder.

**The run writes nothing.** It finds every partial path over the pool and describes what it built in
the same tables the Inspect view uses, which appear beneath the button; *Save as a new store* there
is what writes. Saving asks for a folder rather than for filenames: a store is a pair, and `ta.cbor`
and `ca.cbor` are the names every other interface reads it back under.

## Inspect

A `.cbor` store is a format only PITTv3 reads — there is no text editor or `openssl` command that
will show you what is in one. This view is that viewer, and it is where Generate reports too.

It describes a store without validating anything, in three tables: every certificate the store
holds, the partial paths grouped by the leaf CA they end at, and the trust anchors. They join on
index, so a path can be followed to the certificates it names and a certificate to the paths that
carry it. *Narrow to* filters the rows. Each table exports its own summary as CSV, and the
certificates themselves as DER, so what is on screen can be taken away.

The time of interest decides which certificates are *usable*, and one outside it is listed and
marked rather than dropped — the view is describing a store, not filtering it.

Editing is staged, and saving is the only thing that moves an index. Marking a row marks it for
removal and changes nothing else; *Mark what a cleanup would remove* applies the rule the command
line's `--cleanup` applies — will not parse, not valid at the time of interest, self-signed, or not
a CA; *Clear marks* discards the marks. *Save as a new store* compacts what survived, rediscovers
the partial paths over the result rather than carrying the old ones across, and asks for a folder to
write `ta.cbor` and `ca.cbor` into. What was opened is untouched either way.

## Check URIs

Fetches every HTTP URI one certificate names and reports each on its own, independently of path
processing. See [Checking the URIs in a certificate](7_miscellaneous.md) for what it does and does
not tell you.
