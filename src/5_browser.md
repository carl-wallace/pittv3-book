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
CRLs and an OCSP response for each certificate whose issuer runs a responder.

The certificates being validated stay in the page either way. The URIs they name do not, and an OCSP
request identifies the certificate being asked about even though the certificate itself is not sent.
That distinction is the whole reason the setting exists: it is a privacy choice, not a performance
one.

## Trust material

Uploaded trust anchors and intermediate CA certificates may be DER or PEM certificates, or a `.cbor`
store file — the same format as the built-in stores, and the format *Export PKI Environment* writes.
So the trust material a run actually used can be saved and uploaded again. A `.cbor` upload merges
all of its certificates into that side.

Uploads are used **together with** the selected built-in store. Select the custom entry to rely on
uploads alone, which — with `.cbor` uploads — lets you mix any trust-anchor store with any CA store.
Uploads accumulate across selections until cleared.

### Built-in stores

The Web PKI store holds the Mozilla trust anchors plus the CCADB intermediate CAs. The DoD store
holds the NIPR roots and intermediate CAs.

Where the application is served by a PITTv3 service, the trust stores that service holds appear in
the same dropdown. A store it holds under a name the application already ships is the same material
and is not listed twice. The line under the dropdown says where the selected store came from, which
matters for a store a deployment supplied itself: its certificates may have been gathered by
following AIA URIs rather than published by the PKI they claim to come from.

## Validating

Certificates to validate accumulate as they are selected. Nothing runs until the Validate button is
pressed, which validates every loaded certificate against the current store, uploads and settings.

With *Validate all paths* off, processing stops at the first valid path; on, every discovered path
is validated and reported.

A time of interest of `0` disables validity period checks.

## Results

The Results view holds the report, the run log and the exports: the structured report as JSON, the
log as text, every path's manifest as one text file, and the certificates and revocation data behind
every path as a zip.

## Check URIs

Its own view, fetching every HTTP URI one certificate names and reporting each on its own. See
[Checking the URIs in a certificate](7_miscellaneous.md). With retrieval set to this browser only
there is nothing to fetch with, so the check needs the service.

## Hackathon

The Hackathon view validates provider `artifacts_certs_r5.zip` archives from the hackathon
repository wholesale: the archive's own trust anchors are used and each end-entity certificate is
validated against them, honoring the current settings. It is separate from the Validate view and
does not use the selected store.
