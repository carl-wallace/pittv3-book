# Browser application

The browser application is a WebAssembly build of the same validation library the other interfaces
use. **Path validation always runs in the browser.** What changes is whether anything is fetched to
feed it.

## Retrieval

The Retrieval setting has two positions, and the difference is what leaves the page.

**In this browser only.** Nothing is fetched. Validation uses the selected store and whatever has
been uploaded, and revocation status is undetermined unless revocation data was supplied.

**Through the service.** A PITTv3 service fetches on the page's behalf: issuer certificates from AIA
and SIA URIs when no path can be built, and, for the certificates on the paths it does build, their
CRLs and an OCSP response for each certificate whose issuer runs a responder. Turning off *Chase
SIA/AIA*, on the Validate view or the Settings view, keeps the service for revocation checking alone.

The certificates being validated stay in the page either way. The URIs they name do not, and an OCSP
request identifies the certificate being asked about even though the certificate itself is not sent.
That distinction is the whole reason the setting exists: it is a privacy choice, not a performance
one.

## Trust material

Uploaded trust anchors and intermediate CA certificates may be DER or PEM certificates, or a `.cbor`
store file — the same format as the built-in stores, and as the `derived/built-ta.cbor` and
`derived/built-graph.cbor` of a saved artifacts bundle. So the trust material a run actually used can
be saved and uploaded again. A `.cbor` upload merges
all of its certificates into that side.

Uploads are used **together with** the selected built-in store. Select the custom entry to rely on
uploads alone, which — with `.cbor` uploads — lets you mix any trust-anchor store with any CA store.
Uploads accumulate across selections until cleared.

### Built-in stores

The Web PKI store holds the Mozilla trust anchors plus the CCADB intermediate CAs. The DoD store
holds the NIPR roots and intermediate CAs, and a second one beside it holds the JITC
operational-test material, which is the same hierarchy as issued for testing rather than for
production — a certificate from one does not validate under the other. The ECA store holds the
External Certification Authority roots and the vendor CAs beneath them — the program under which
commercial vendors issue to people and systems outside the Department of Defense that interoperate
with it. The WCF store holds the WCF root, the intermediate beneath it and the ten signing CAs
beneath that; DISA publishes it as its own InstallRoot stream, and a path through it shows two
certificates between anchor and target rather than one, since every signing CA sits beneath that
intermediate. NIPR, ECA and WCF are separate trust sets: a certificate from one does not validate
under the others.

Six stores come from the Microsoft root program, which Windows uses and which publishes something
the others do not: the purposes it grants each root. One holds every root the program still trusts,
and five are narrowed to a single purpose — server authentication, client authentication, S/MIME,
code signing, timestamping. The server-authentication one asks the same question the Web PKI store
answers, so the two can be compared directly. None of the six carries intermediate CAs — Microsoft
publishes roots alone — so a chain validated against them needs its CAs uploaded or retrieved.

The TPM store holds the roots of the trusted platform module vendors, which is what an attestation
key certificate chains to. It answers a question none of the others do: whether a key was generated
in the part a vendor vouches for, rather than whether a person or a server is who they claim.

Two further stores are NIPR plus an interoperability root: the production material above, the
cross-certified root as a fifth anchor, and the certificates that root publishes in its own
repository. The DoD Interoperability Root CA 2 store reaches the ECA program and, through Federal
Bridge CA G4, the U.S. federal mesh; the CCEB one reaches the allied national PKIs that root
cross-certifies — Australian Defence, DND/MDN Canada — and not the federal mesh. In both, DoD Root
CA 3 and DoD Root CA 6 appear twice over, as anchors and again as certificates the interoperability
root issued, which is what lets a path climb to that root and come back down the other side. Neither
store contains the other, and a target that validates under one of them may not under the other.

Where the application is served by a PITTv3 service, the trust stores that service holds appear in
the same dropdown. A store it holds under a name the application already ships is the same material
and is not listed twice. The line under the dropdown says where the selected store came from, which
matters for a store a deployment supplied itself: its certificates may have been gathered by
following AIA URIs rather than published by the PKI they claim to come from.

That line also says how current the material is, in the same words the desktop uses: *published*,
the date the source itself gives, and *collected*, the day the material was taken from it. The two
can be far apart — a DoD InstallRoot stream is timestamped when signed, and can be well over a year
old by the time it is downloaded — which is why the fetch date alone would be misleading. A store
shows only the dates it can state honestly; one assembled from a service's configured directory
shows none, since a `.cbor` artifact records certificates and not when they were gathered.

## Validating

Certificates to validate accumulate as they are selected. Nothing runs until the Validate button is
pressed, which validates every loaded certificate against the current store, uploads and settings.

With *Validate all paths* off, processing stops at the first valid path; on, every discovered path
is validated and reported.

A time of interest of `0` disables validity period checks.

Setting one at all is almost always wrong unless you mean to validate relative to a past moment —
code signing, or replaying an archived run. A past time also discards revocation data published
since, which is correct and reads as a changed verdict.

## Results

The Results view holds the report, the run log and the exports: the structured report as JSON, the
log as text, every path's manifest as one text file, and the certificates and revocation data behind
every path as a zip.

## Check URIs

Its own view, fetching every HTTP URI one certificate names and reporting each on its own. See
[Checking the URIs in a certificate](7_miscellaneous.md). With retrieval set to this browser only
there is nothing to fetch with, so the check needs the service.

*Check Self-Signed* says whether the chosen certificate is self-signed: yes, no, or cannot tell when
this build has no verifier for its signature algorithm. It is answered in the page and needs no
service.

## Generate

Builds a store rather than validating against one. Its material is what is loaded in the *Inputs*
group on the tab — trust anchors and intermediate CA certificates, as DER or PEM certificates, as
bundles, or as `.cbor` stores, which is how an existing store serves as a starting point. A store
the application ships is deliberately not offered as one, as it is not on the desktop: what goes
into a store you are about to hand out should be material you chose.

*Chase SIA and AIA* follows the URIs the loaded certificates name and folds what they serve into the
pool before the paths are found, within the run's retrieval budget. That is retrieval, so it needs
the service and is unavailable without one; building from the material already loaded needs nothing,
which is why the view itself is available on a statically hosted copy.

A partial path runs from a certificate to a trust anchor, so at least one anchor has to be among the
material — the command line requires one for the same reason. The time of interest decides which
certificates reach the store.

The certificates that do reach it are the ones the command line keeps when it reads a folder: one
that will not parse, is outside the time of interest, is self-signed, or does not assert `cA` is
left out, and the run says how many it imported out of how many candidates. Those exclusions arrive
as **marks on the report** rather than as a silent drop — the command line applies the same screen
while reading the folder, where there is nothing left to show — so they can be cleared if a store of
everything is what you want.

**The run writes nothing.** It describes what it built in the tables below, and *Save as a new
store* hands the pair over as a single zip holding `ta.cbor` and `ca.cbor`, those being the names
every other interface reads them back under. One zip rather than two saves because a page cannot
start two downloads, and because the pair is one artifact.

## Inspect

Describes a store without validating anything, in the same three tables the desktop shows: every
certificate the store holds, the partial paths grouped by the leaf CA they end at, and the trust
anchors, joined on index. The store is the selected one, or the `.cbor` pair named with the two file
controls, or both — a named store's anchors and certificates merge with the selector's.

Supplying a certificate under *Partial Paths for Target* adds the paths the store could build to
that certificate, which is the question to ask when something would not validate and you want to
know what the store had to work with.

Marking, *Mark what a cleanup would remove*, *Clear marks* and *Save as a new store* behave as they
do on the desktop, the save arriving as the same zip Generate's does. The report, the marks and the
time of interest are shared with Generate: it is one report reached by two errands, not two copies.

## Hackathon

The Hackathon view validates provider `artifacts_certs_r5.zip` archives from the hackathon
repository wholesale: the archive's own trust anchors are used and each end-entity certificate is
validated against them, honoring the current settings. It is separate from the Validate view and
does not use the selected store.
