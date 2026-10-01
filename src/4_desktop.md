# Desktop application

The desktop application presents the same options as the command line as a form, organized into
views reached from the sidebar:

> Validate · Results · Settings · Check URIs · Generate · Inspect · Help

The order is the ordinary path through the application — validate something, read the outcome,
adjust what a run does — followed by the tools for working on trust material rather than validating
against it. The first four match the browser application, which has a similar look and feel and uses
a shared code base.

## Validate

The store selector at the top chooses a complete trust environment on its own. The panels below it
supplement that store, and they open by themselves whenever they hold anything, so material left
over from an earlier run is never hidden behind a collapsed heading unless the user chooses to hide it.

The line under the selector says whose PKI the chosen store holds, whether it carries intermediate
CAs as well as anchors, and how current its material is — up to two dates, *published* and
*collected*. They are not the same question. *Published* is the date the source itself gives: a DoD
InstallRoot stream is timestamped when it is signed, and the version PITTv3 ships can be
considerably older than the day it was downloaded. *Collected* is when the material was taken from
that source, which bounds what it can possibly know: anything the publisher has done since is not
in these bytes. A store shows only the dates it can state honestly; for example, the Windows certificate
stores show none, since those are read live from the host system rather than shipped.

**Trust anchors and certification authorities.** One list per kind. Each entry may be a folder, a
certificate, a bundle holding several, or a `.cbor` store; the run decides which from the path and
then from the bytes, so there is nothing to declare.

**Revocation data.** Two lists, CRLs and OCSP responses, each with its own count and *Clear*. They
are not filtered apart — an OCSP response has no settled file extension — so either list will accept
either kind and each reports what it is actually holding rather than what its label promises. What
the split buys is being able to clear one kind without disturbing the other.

**End entity certificates.** End entity certificates are the target of validation. The validate button(s) 
are disabled until there is at least one.

Certificates can also be taken from a TLS server by identifying a host: the desktop opens the connection
and harvests the certificates it was presented and any stapled OCSP responses.

### A second opinion from Windows

**On Windows only**, a second button beside the ordinary one validates the same certificates with
the Windows chain engine instead of with `certval`. It is offered here rather than on a view of its
own because it answers a question about the material already displayed, and it reads the same
inputs the ordinary run is built from, so no control can feed one validator and not the other.

The checkbox above it, *CAPI uses this run's trust anchors*, decides where trust comes from, and
the two settings ask genuinely different questions.

**On**, the anchors this run was given — the selected store, and anything added through the
trust-anchor list — become the engine's only roots. Both validators then judge the same material,
so a difference in the answer is a difference between the validators rather than between their
inputs. That is the setting to use when the question is about `certval`.

**Off**, trust comes from this machine's certificate stores, which is the question PITTv2's CAPI
panel asked: would this computer accept the certificate. 

**With a time of interest of `0`, the two can disagree about validity periods.** `0` turns off
validity period checks in `certval`, and the Windows chain engine has no equivalent, so a CAPI run
validates as of now. The CAPI log notes this under its first line.

## Results

The Results view displays a variety of information about a validation operation: a status output 
for each end entity certificate that was considered, a run log, and four export buttons for: the 
structured report as JSON, the log as text, every path's manifest as one text file, and the 
certificates and revocation data behind every path as a zip file. The name field beside them 
is used to name the exported files along with a timestamp that corresponds to when that run began.
Names and timestamps can be used to correlate the different exported items.

On Windows, a CAPI run appears as a second tab here rather than replacing the first, since using
the same inputs to two validators is the whole point and comparing the results can be useful. 

*Clear* discards the results and the log, and is disabled when there is nothing to discard.

## Settings

The tabbed form edits a settings file. This is the same JSON the command line takes with `-s`. Fields
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

Because the paths are rediscovered against the anchors that survive, **removing a trust anchor also
removes every path to it from the saved `ca.cbor`**, even though the certificates stay. That matters
when the CA store is meant to be paired with other anchor sets: to narrow the anchors while keeping
a full CA store, use the saved `ta.cbor` with the original `ca.cbor` rather than the edited one. See
[Partial paths](2_concepts.md#partial-paths).

## Check URIs

Fetches every HTTP URI one certificate names and reports each on its own, independently of path
processing. See [Checking the URIs in a certificate](7_miscellaneous.md) for what it does and does
not tell you.

*Check Self-Signed* says whether the target certificate is self-signed: yes, no, or cannot tell when
this build has no verifier for its signature algorithm. It is a separate check that needs no
fetching; the view is simply where a single certificate is already chosen.
